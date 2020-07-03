# 2020/7/3 更新内容

- マクロから import するモジュールの存続期間と XSCRIPTCONTEXT の設定について記述

# 使用バージョン

- LibreOffice 6.4.4
- Python 3.7.7

# 概要

LibreOffice を操作する Python でモジュールを import する際には、通常に加えて以下のモジュール検索を行う。
- マクロ実行時に\<LibreOffice インストールディレクトリ\>/program ディレクトリが自動的にモジュール検索パス sys.path に追加されている。
- マクロ実行時には、とマクロ設置ディレクトリの pythonpath サブディレクトリと pythonpath.zip ファイルがもし存在すれば自動的に sys.path に追加されている。
- マクロもしくは \<LibreOffice インストールディレクトリ\>/program/uno.py を import したプログラムからは　com.sun.star で始まる UNO 固有モジュールからの import が行える。モジュール自体の import はできない。

# マクロの sys.path

## \<LibreOffice インストールディレクトリ\>/program ディレクトリ

Python マクロの実行時には、\<LibreOffice インストールディレクトリ\>/program ディレクトリが sys.path に含まれる。
\<LibreOffice インストールディレクトリ\> は、FreeBSD の場合 /usr/local/lib/libreoffice、Microsoft Windows の場合 %PROGRAMFILES%\LibreOffice。
uno, unohelper, pythonscript, msgbox はここから import する。

## pythonpath ディレクトリと pythonpath.zip ファイル

マクロを設置したディレクトリの pythonpath サブディレクトリもしくは pythonpath.zip ファイルが存在すれば sys.path に追加されてモジュール検索の対称となる。

ユーザーマクロの場合,

- Microsoft Windows の場合

```
%APPDATA%\LibreOffice\4\user\Scripts\python\pythonpath
%APPDATA%\LibreOffice\4\user\Scripts\python\pythonpath.zip
```

- FreeBSD もしくは UNIX 類似システムの場合

```
${HOME}/.config/libreoffice/4/user/Scripts/python/pythonpath
${HOME}/.config/libreoffice/4/user/Scripts/python/pythonpath.zip
```

となる。
これらのパスは、マクロ終了後も LibreOffice を終了するまで維持される。
保持しているのは \<LibreOffice インストールディレクトリ\>/program/pythonloader.py の Loader クラスと推測される。
同じファイル中の checkForPythonPathBesideComponent() 関数が存在のチェックと sys.path への追加に使用されている。

システム共有マクロや機能拡張の場合でもそれぞれ追加されるのでマクロ呼び出し時に不必要なディレクトリが sys.path に含まれている可能性がある。

例えば sys.path の内容をメッセージボックスに表示するのに以下のマクロを実行してみる。

```py:syspath.py
import sys
from msgbox import MsgBox

def syspath_msgbox():
    ctx = XSCRIPTCONTEXT.getComponentContext()
    sys_path_box = MsgBox(ctx)
    sys_path_box.addButton('oK')
    sys_path_box.renderFromBoxSize(300)
    sys_path = '\n'.join(sys.path)
    sys_path_box.show(sys_path, 0, 'sys.path')
```

表示されるのは以下のとおり。この時 pythonpath サブディレクトリは作ってあるが pythonpath.zip ファイルは存在しない。
2 番目に表示された /home/shota243 は LibreOffice 起動時のカレントディレクトリ。この時 PYTHONPATH 環境変数は設定していない。

```
/usr/local/lib/libreoffice/program
/home/shota243
/usr/local/lib/python37.zip
/usr/local/lib/python3.7
/usr/local/lib/python3.7/lib-dynload
/home/shota243/.local/lib/python3.7/site-packages
/usr/local/lib/python3.7/site-packages
/home/shota/.config/libreoffice/4/user/Scripts/python/pythonpath
```

[APSO - Alternative Script Organizer for Python](https://extensions.libreoffice.org/en/extensions/show/apso-alternative-script-organizer-for-python) 拡張機能の Python インタープリタコンソールから sys.path を表示した場合は以下のとおり。

```py
APSO python console [LibreOffice]
3.7.7 (default, Jun 21 2020, 22:27:00) 
[Clang 8.0.0 (tags/RELEASE_800/final 356365)]
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> for p in sys.path:
...  print(p)
... 
/usr/local/lib/libreoffice/program
/home/shota243
/usr/local/lib/python37.zip
/usr/local/lib/python3.7
/usr/local/lib/python3.7/lib-dynload
/home/shota243/.local/lib/python3.7/site-packages
/usr/local/lib/python3.7/site-packages
/home/shota243/.config/libreoffice/4/user/uno_packages/cache/uno_packages/lu2030qffpne.tmp_/apso.oxt/python/pythonpath
>>> 
```

APSO 拡張機能のインストール先が　/home/shota243/.config/libreoffice/4/user/uno_packages/cache/uno_packages/lu2030qffpne.tmp_。
(前略)lu2030qffpne.tmp_/apso.oxt/python/apso.py が本体で (前略)lu2030qffpne.tmp_/apso.oxt/python/pythonpath ディレクトリ下の
apso_debug.py, apso_utils.py, theconsole.py がモジュール。

## マクロから import したモジュールの存続期間

マクロを終了した後でマクロからしたモジュールを変更して再度マクロを起動しても変更後のモジュールは再読み込みされず変更前の状態のままとなる。これはマクロを終了しても Python インタープリタは終了しないことによると考えられる。
モジュールを読み込み直すためには LibreOffice 自体を再起動する。

## マクロから import したモジュールの XSCRIPTCONTEXT

マクロの XSCRIPTCONTEXT 変数は [imp.new_module()](https://docs.python.org/ja/3.7/library/imp.html#imp.new_module) 関数で作成したモジュールの変数として作成され、マクロから import したモジュールからは参照できない。直接参照できないだけではなく関数の引数で渡しても、あるいは XSCRIPTCONTEXT のメソッドを呼び出して取得した Desktop 等を引数で渡しても参照時にエラーとなる。

回避策として、モジュールの側で uno.getComponentContext() でコンテクストを取得して XSCRIPTCONTEXT 変数相当のものを作成する。

```py:pythonpath/xscriptcontext.py
import uno
from pythonscript import ScriptContext

def get_xscriptcontext():
    UNO_DESKTOP = "com.sun.star.frame.Desktop"
    ctx = uno.getComponentContext()
    smgr = ctx.getServiceManager()
    XSCRIPTCONTEXT = ScriptContext(ctx,
                                   smgr.createInstanceWithContext(UNO_DESKTOP, ctx),
                                   None)
    return XSCRIPTCONTEXT
```

この XSCRIPTCONTEXT は、[LibreOffice を Python で操作する](https://qiita.com/shota243/items/286ae4083556ae98b611) に書いた起動済みの LibreOffice に接続する Python プログラム用の XSCRIPTCONTEXT 同様 inv (Invocation Context) パラメータを None として作成しているため getInvocationContext() メソッドが None を返す。
Invocation Context のサポートは将来の課題とする。

# UNO 固有モジュールからの import

マクロもしくは uno を import したプログラムでは、com.sun.star. で始まるモジュールから識別子を import できる。

これは、import の実態である \_\_import\_\_() 関数が \<LibreOffice インストールディレクトリ\>/program/uno.py 内の \_uno\_import() 関数で置き換えられていることによる。
com はモジュール検索パスのどこにも存在しない。
\_uno\_import() 関数では、標準の \_\_import\_\_() 関数でのモジュール検索に失敗すると pyuno.getClass() 関数で取得を試みる。
この際に from \<モジュール名\> import \<識別子\> の \<モジュール名\> と \<識別子\> は連結して getClass() を呼ぶので \<モジュール名\> 全体の import や from \<モジュール名\> import * での一括 import は例外となる。

例：

```py
mike% export PYTHONPATH="${PYTHONPATH}:/usr/local/lib/libreoffice/program"
mike% export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/lib/libreoffice/program"
mike% export PATH="${PATH}:/usr/local/lib/libreoffice/program"
mike% python
python
Python 3.7.7 (default, Jun 21 2020, 22:27:00) 
[Clang 8.0.0 (tags/RELEASE_800/final 356365)] on freebsd11
Type "help", "copyright", "credits" or "license" for more information.
>>> import uno
import uno
>>> from com.sun.star.uno import RuntimeException
from com.sun.star.uno import RuntimeException
>>> import com.sun.start.uno.RuntimeException
import com.sun.start.uno.RuntimeException
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/libreoffice/program/uno.py", line 356, in _uno_import
    return _builtin_import(name, *optargs, **kwargs)
ModuleNotFoundError: No module named 'com'
>>> from com.sun.star.uno import *
from com.sun.star.uno import *
from com.sun.star.uno import *
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/libreoffice/program/uno.py", line 434, in _uno_import
    raise uno_import_exc
  File "/usr/local/lib/libreoffice/program/uno.py", line 356, in _uno_import
    return _builtin_import(name, *optargs, **kwargs)
ImportError: No module named 'com' (or 'com.sun.star.uno.*' is unknown)
>>> 
```

import できるモジュール、識別子のリファレンスは[com::sun::star Module Reference](https://api.libreoffice.org/docs/idl/ref/namespacecom_1_1sun_1_1star.html) と考えられる。
