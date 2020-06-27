# 使用バージョン

- LibreOffice 6.3.6 on FreeBSD, Windows
- Python 3.7.7

# 概要

LibreOffice を Python で操作するには、

1. LibreOffice の GUI から Python マクロを起動する。
2. コマンドラインから LibreOffice を起動する際にパラメータで指定した Python マクロを起動する。
3. コマンドラインから Python プログラムを起動し、起動済みの LibreOffice に接続する。

の 3 つの方法がある。

1 番めの方法は GUI 操作が必要なのでプログラムからの一括処理が行えない。
あらゆるエラーには操作者が対応する。

2 番めの方法は、マクロの終了後に LibreOffice が終了するため、一括処理での使用が可能となる。
エラー発生時には LibreOffice はダイアログを表示して待ちとなりプログラムからは対処できなくなる。
発生しえるエラーは考慮の上、[timeout-decorator](https://pypi.org/project/timeout-decorator/)で時間制限を設ける等の対策が必要となる。
また、LibreOffice 起動時にすでに起動済みの LibreOffice があればそちらで処理が行われるため期待どおりの制御ができない可能性がある。

3 番めの方法は、あらかじめ LibreOffice に --accept スイッチを指定してソケットまたはネームドパイプで接続できるようにして起動しておく。
この方法も先に LibreOffice が起動されていればそちらで処理が行われるため期待どおりの制御ができない可能性がある。
Microsoft Windows では使用できたが FreeBSD では接続時にエラーとなり実行できない。

# Python 処理系

## Microsoft Windows

Microsoft Windows では LibreOffice に Python 処理系 %PROGRAMFILES%\LibreOffice\program\python.exe が付属しており、マクロの場合は自動的にその処理系が使用される。
コマンドラインから接続する場合も LibreOffice 付属の Python 処理系を使用する必要がある。
LibreOffice 6.3.6 に付属の Python バージョンは 3.5.9。

ライブラリは %PROGRAMFILES%\LibreOffice\program\python-core-3.5.9\lib にインストールされている。
site-packages サブフォルダもあり、sys.path も通っているが内容は README ファイルがあるだけである。
pip がないためパッケージのインストールが簡単に行えない。
[pip のサイト](https://pip.pypa.io/en/stable/) の [Installation](https://pip.pypa.io/en/stable/installing/) から
get-pip.py をダウンロードして実行することでインストールが可能であるが、個人アカウントの権限で実行すると
%PROGRAMFILES%\LibreOffice\program\python-core-3.5.9\lib\site-packages は書き込み不可のため
ユーザーのプロファイル下の %APPDATA%\Roaming\Python\Python35\site-packages にインストールされる。

```
C:\Users\shota243>cd Downloads
C:\Users\shota243\Downloads>"c:\Program Files\LibreOffice\program"\python get-pip.py
Defaulting to user installation because normal site-packages is not writeable
Collecting pip
  Downloading pip-20.1.1-py2.py3-none-any.whl (1.5 MB)
     |################################| 1.5 MB 1.3 MB/s
Collecting setuptools
  Downloading setuptools-47.1.1-py3-none-any.whl (583 kB)
     |################################| 583 kB 6.8 MB/s
Collecting wheel
  Downloading wheel-0.34.2-py2.py3-none-any.whl (26 kB)
Installing collected packages: pip, setuptools, wheel
  WARNING: The scripts pip.exe, pip3.5.exe and pip3.exe are installed in 'C:\Users\shota243\AppData\Roaming\Python\Python35\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The scripts easy_install-3.5.exe and easy_install.exe are installed in 'C:\Users\shota243\AppData\Roaming\Python\Python35\Scripts' which is not on PATH.

  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The script wheel.exe is installed in 'C:\Users\shota243\AppData\Roaming\Python\Python35\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed pip-20.1.1 setuptools-47.1.1 wheel-0.34.2
```

pip.exe も %APPDATA%\Roaming\Python\Python35\Scripts にインストールされるが、このフォルダに PATH を通すのには自分で設定しなければならないことと、pip.exe の実行には %PROGRAMFILES%\LibreOffice\program\python35.dll が必要になるため DLL 検索パスの対処も必要になることを考えると

```
%PROGRAMFILES%\LibreOffice\program\python -m pip
```

で起動する方が楽かもしれない。
pip でインストールしたパッケージも %APPDATA%\Roaming\Python\Python35\site-packages にインストールされるが、sys.path が通っているのでそのまま使用できる。

```
C:\Users\shota243>"c:\Program Files\LibreOffice\program"\python -m pip install timeout-decorator
Defaulting to user installation because normal site-packages is not writeable
Collecting timeout-decorator
  Downloading timeout-decorator-0.4.1.tar.gz (4.8 kB)
Building wheels for collected packages: timeout-decorator
  Building wheel for timeout-decorator (setup.py) ... done
  Created wheel for timeout-decorator: filename=timeout_decorator-0.4.1-py3-none-any.whl size=5022 sha256=5d3ecc37c619ed429c313b18ddf3b01b309d43dccc65d19ce974df
76af478609
  Stored in directory: c:\users\shota243\appdata\local\pip\cache\wheels\df\f4\3c\031226c90008861114781e36ef5898934aad3196808c40e6ea
Successfully built timeout-decorator
Installing collected packages: timeout-decorator
Successfully installed timeout-decorator-0.4.1
```

管理者権限で get-pip.py を実行すれば pip を %PROGRAMFILES% 下のフォルダにインストールしてシステムで共有できる。
他のパッケージも同様に%PROGRAMFILES% 下にインストールできる。
ただし、%PROGRAMFILES%\LibreOffice フォルダはLibreOffice のアップグード時に消去される可能性があり対応が必要となる可能性がある。

## FreeBSD

FreeBSD の場合は LibreOffice に Python 処理系が付属せずマクロであっても、PATH の通っている Python が使用される。
pip でインストールしたパッケージはマクロからでも使用できる。
仮想環境下で LibreOffice を起動すれば Python バージョンやパッケージを使い分けられる。

# UNO

LibreOffice をプログラムから操作するのには [UNO (Universal Network Objects)](http://www.openoffice.org/udk/) を用いる。
UNO は言語独立のコンポーネントモデルであり、Python から使用する時には [Python-UNO bridge](http://www.openoffice.org/udk/python/python-bridge.html) (PyUNO-bridge) を使用する。
マクロからであっても外部からの接続であっても同じく Python-UNO bridge を使用する。

# Python マクロ

## Python マクロの配置

LibreOffice は Python マクロの編集機能もエディタを起動する機能も持たない。
Python マクロは、別途作成の上、下記のディレクトリに配置する。

### Microsoft Windows の場合

- ユーザーマクロは

```
%APPDATA%\LibreOffice\4\user\Scripts\python
```

- システム共有マクロは

```
%PROGRAMFILES%\LibreOffice\share\Scripts\python
```

### FreeBSD もしくは UNIX 類似システムの場合

- ユーザーマクロは

```
${HOME}/.config/libreoffice/4/user/Scripts/python
```

- システム共有マクロは

```
/usr/local/lib/libreoffice/share/Scripts/python
```

ユーザーマクロ配置用ディレクトリは元からは存在しないので必要に応じて自分で作る。

## APSO - Alternative Script Organizer for Python 拡張機能

[APSO - Alternative Script Organizer for Python](https://extensions.libreoffice.org/en/extensions/show/apso-alternative-script-organizer-for-python) 拡張機能をインストールすると以下が可能となる。

- Pythonインタープリタコンソールの起動
- GUIデバッガの使用
- LibreOffice からエディタを起動してマクロを編集
- 文書への Python マクロの埋め込み

これによりマクロ配置を意識することなく編集が可能となる。
また、Pythonインタープリタから UNO へのアクセスを試すことができる。

## XSCRIPTCONTEXT グローバル変数

Pythonマクロからは XSCRIPTCONTEXTグローバル変数が参照できる。
XSCRIPTCONTEXT からは以下のメソッドによりマクロが起動された環境にアクセスできる。

- getDocument()
- getInvocationContext()
- getDesktop()
- getComponentContext()

XSCRIPTCONTEXT は [XScriptContext](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1script_1_1provider_1_1XScriptContext.html) インタフェースの実装であるが、
Python のグローバル変数としては pythonscript.ScriptContext クラスのインスタンスとなっている。

```py
>>> type(XSCRIPTCONTEXT)
<class 'pythonscript.ScriptContext'>
```

クラス定義は /usr/local/lib/libreoffice/program/pythonscript.py ファイルにある。

## コマンドラインからのマクロの起動

LibreOffice 起動時にマクロを指定するコマンドラインは以下のとおり

```
soffice 'vnd.sun.star.script:<スクリプトファイル名>$<関数名>?language=Python&location=user'
```

これによりマクロを実行して LibreOffice を終了する。
マクロが画面を表示する必要がなければ --headless スイッチをつければ画面は表示されない。
マクロ実行中にエラーが発生するとエラーダイアログを表示してユーザーの確認を要求するが --headless が指定してあるとダイアログも表示せずにハングアップする。

例：

```py:mymacro.py
def macrofunc():
    pass
```

を画面表示無しで起動するには

```
$ soffice --headless 'vnd.sun.star.script:mymacro.py$macrofunc?language=Python&location=user'
```

# コマンドラインから起動した Python からの接続

## LibreOffice の起動

外部からの接続を受けるためにはあらかじめ LibreOffice に --accept スイッチを指定して起動しておく。
ソケット接続の場合は

```
$ soffice  "--accept=socket,host=<ホスト指定>,port=<ポート番号>;urp;"
```

ネームドパイプ接続の場合は

```
$ soffice "--accept=pipe,name=<ネームドパイプ名>;urp;"
```

いずれも UNIX 系 OS のシェルではセミコロン(;) をコマンド区切りとして扱うため、クォートする必要がある。

例：

```
$ soffice --writer "--accept=socket,host=0,port=2002;urp;"
```

```
$ soffice --calc --headless "--accept=pipe,name=librepipe;urp;"
```

[Python-UNO bridge](http://www.openoffice.org/udk/python/python-bridge.html)ページの例では

```
"-accept=socket,host=localhost,port=2002;urp;"
```

と "accept" の前のマイナス記号が 1 つになっているが、これは古い記法で 2 つ書くのが現在の記法である。
また、同じ例ではプログラムで Writer 文書に "Hello, World" 文字列を挿入しているがプログラム中では文書の新規作成を行っていないのでそのまま実行すると text プロパティへのアクセスでエラーとなる。
LibreOffice の起動時に --writer スイッチで新規作成しておくとその文書が使用される。

## Python プログラムからの接続

Python プログラムでは、LibreOffice 起動時の --accept オプションスイッチのパラメータは同様の文字列を接続する側でリゾルバー（com.sun.star.bridge.UnoUrlResolver インスタンス）による接続先の解決時に指定する。
host は相手ホスト名なので異なる値を指定することになる。

LibreOffice に接続してContext を取得したら、それを使用して Desktop インスタンスを作成し、ScriptContext クラスのインスタンスを作成するとマクロにおける XSCRIPTCONTEXT グローバル変数を代替できる。
ScriptContext クラスのコンストラクタの第3引数は invocation context であるが、[XScriptContext インタフェースのドキュメント](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1script_1_1provider_1_1XScriptContext.html) によれば NULL で構わないので None を指定する。

```py
import uno
from pythonscript import ScriptContext

def connect_script_context(host='localhost', port='2002', namedpipe=None):
    UNO_RESOLVER = "com.sun.star.bridge.UnoUrlResolver"
    UNO_DESKTOP = "com.sun.star.frame.Desktop"
    localCtx = uno.getComponentContext()
    localSmgr = localCtx.ServiceManager
    resolver = localSmgr.createInstanceWithContext(UNO_RESOLVER, localCtx)
    if namedpipe is None:
        uno_string = 'uno:socket,host=%s,port=%s;urp;StarOffice.ComponentContext' % (host, port)
    else:
        uno_string = 'uno:pipe,name=%s;urp;StarOffice.ComponentContext' % namedpipe
    ctx = resolver.resolve(uno_string)
    smgr = ctx.ServiceManager
    XSCRIPTCONTEXT = ScriptContext(ctx,
                                   smgr.createInstanceWithContext(UNO_DESKTOP, ctx),
                                   None)
    return XSCRIPTCONTEXT
```

この connect_script_context() 関数の戻り値を XSCRIPTCONTEXT グローバル変数に代入することでマクロの場合をシミュレートする。

### FreeBSD の場合の設定とエラー

FreeBSD もしくは UNIX 類似システムから LibreOffice に接続する際には、/usr/local/lib/libreoffice/program 下にある Python プログラムを import すると同時に同じ場所にあるダイナミックリンクライブラリも使用するので、PYTHONPATH の他に LD_LIBRARY_PATH 環境変数も通しておく。

```
export PYTHONPATH="${PYTHONPATH}:/usr/local/lib/libreoffice/program"
export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/lib/libreoffice/program"
export PATH="${PATH}:/usr/local/lib/libreoffice/program"
```

実際に接続すると、接続して ComponentContext を得ることはできるが、Desktop を得ようとしたときにエラーとなる。

```py
>>> import uno
import uno
>>> localContext = uno.getComponentContext()
localContext = uno.getComponentContext()
>>> resolver = localContext.ServiceManager.createInstanceWithContext("com.sun.star.bridge.UnoUrlResolver", localContext)
<reateInstanceWithContext("com.sun.star.bridge.UnoUrlResolver", localContext)
>>> ctx = resolver.resolve("uno:socket,host=localhost,port=2002;urp;StarOffice.ComponentContext")
ctx = resolver.resolve("uno:socket,host=localhost,port=2002;urp;StarOffice.ComponentContext")
>>> smgr = ctx.ServiceManager
smgr = ctx.ServiceManager
>>> desktop = smgr.createInstanceWithContext("com.sun.star.frame.Desktop",ctx)
desktop = smgr.createInstanceWithContext("com.sun.star.frame.Desktop",ctx)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
__main__.RuntimeException: Binary URP bridge disposed during call

```
