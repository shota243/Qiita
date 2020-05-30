# 使用バージョン

- TexMaths 0.48
- LibreOffice 6.3.6
- texlive-base-20150521_51

# 概要

[TexMaths](http://roland65.free.fr/texmaths/) は、LibreOffice の拡張機能で
LaTeX の数式を PNG もしくは SVG に変換して Writer, Impress, Draw 文書に埋め込むことができる。

TexMaths は、Linux, OS X, Microsoft Windows で使用可能とされている。

# FreeBSD での使用

FreeBSD で TexMaths 0.48 を Linux とほぼ同様に動作させるためには、
Bash がシステムのシェルであることを期待している部分を修正する必要がある。

## 依存

LaTeX 数式ソースをコンパイルするために LaTeX が必要となる。
また、結果を PNG に変換するためには dvipng, SVG に変換するためには dvisvgm を使用する。
これらは [texlive-base](https://www.freshports.org/print/texlive-base/) に含まれる。
[texlive-full](https://www.freshports.org/print/texlive-full/) をインストールすればすべて含まれることとなる。

[LibreOffice](https://www.freshports.org/editors/libreoffice) がインストール先であるため必要である。

[bash](https://www.freshports.org/shells/bash) があれば、使用するように修正することも可能である。
今回は bash でなくシステムのシェル /bin/sh を使用するようにした。

## インストール

### TexMaths のインストール

1. LibreOffice を起動。
2. 「ツール」メニューから「拡張機能マネージャー」を起動。
3. 「ほかの拡張機能をオンラインで取得…」リンクをクリック。ブラウザで LibreOffice の Extensions の検索画面が表示される。
4. 検索ボックスに「TexMaths」と記入して検索。
5. 検索結果から「TexMaths」リンクを選択。
6. [TexMaths ページ](https://extensions.libreoffice.org/en/extensions/show/texmaths-1) の「Release List」から最新版（現在は 0.48 の「DONWLOAD」をクリック。
7. ダウンロードされる texmaths-0-48.oxt を LibreOffice で開くか「拡張機能マネージャー」画面から開く。
8, 「拡張機能マネージャー」画面に TexMaths 0.48 がリストされていることを確認。
9. LibreOffice の再起動を促すダイアログが出るので再起動する。

この時点で TexMaths を起動すると Bash 依存のシェルスクリプトを生成保存してしまうので、まず編集する。

### TexMaths の編集

1. Writer を起動。例えば新規文書を作成することによる。
2. 「ツール」メニューから「マクロ」→「マクロの編集」を起動。
3. 「マイマクロ＆ダイアログ」の下に「TexMaths」ノードがあるので、先頭のファイルを開き「bash」を検索して「sh」で置き換える。
ファイル内で見つからなければ自動的に次のファイルを探してくれる。
TexMaths 0.48 での変更箇所は TexMathsEquations の 641 行め、TexMathsConfig の 1177 行め、TexMathsTools の 174 行め、260 行め、450 行め、457 行め、461 行めの 7 箇所。
4. 同様に「&>」を検索して「>」に変更し、リダイレクト先のファイルの後に「 2>&1」を追加する。
「&>」は Bash 固有の記法で標準出力と標準エラー出力を同じファイルにまとめてリダイレクトする指定なので、これを sh で同じことをする記法に書き換える。
TexMaths 0.48 での変更箇所は TexMathsConfig の 1158 行めと 1172 行め、1217 行めの 3 箇所。

この変更は、[bash](https://www.freshports.org/shells/bash) をインストールした上で「/bin/bash」となっている箇所のみ「/usr/local/bin/bash」に修正することでも充分なはずであるが試していない。

編集する前に TexMaths を起動した場合や後から編集内容を変更したい場合は、生成された
${HOME}/.config/libreoffice/4/user/TexMaths/TexMaths-0.48.sh シェルスクリプトも編集する。

## 使用

Writer のツールバーの左端に4 つのアイコンが追加されている。
左から

- TexMaths Equations
- TexMaths Numbered Equations
- TexMaths System Configuration
- TexMaths Recompile Equations

となる。左の 2 つが数式の編集ダイアログを起動する。
この 2 つの違いは、Numbered の方が式番号も挿入すること。

一旦 SVG や PNG にして挿入された数式を選択して TexMaths Equations アイコンをクリックすることにより再編集できる。

## 地域化メッセージ

TexMaths は翻訳メッセージファイルを
${HOME}/.config/libreoffice/4/user/uno_packages/cache/uno_packages/lu1313svtdi6.tmp_/texmaths-0-48.oxt/po
ディレクトリ下に持っている。
ここに ja.po ファイルを作成して日本語翻訳を入れれば表示させることができるが翻訳は現在提供されていない。
