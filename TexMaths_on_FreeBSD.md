# 使用バージョン

- TexMaths 0.49
- LibreOffice 7.1.5.2
- TeX Live (texlive-base) 20150521_69

# 2021年8月変更点

- TexMaths 0.49 へのアップグレードにともない Bash 固有の表記がなくなり sh の書き方への変更の記述が不要となったので削除。
- latex, dvisvgm, dvipng の代替のスクリプトにより日本語に対応。

# 概要

[TexMaths](http://roland65.free.fr/texmaths/) は、LibreOffice の拡張機能で
LaTeX の数式を PNG もしくは SVG に変換して Writer, Impress, Draw 文書に埋め込むことができる。LaTeXソースも同時に埋め込まれていて後から編集できる。

TexMaths は、Linux, OS X, Microsoft Windows で使用可能とされている。

# FreeBSD での使用

FreeBSD で TexMaths 0.49 を使用できる。
数式に含まれる日本語は表示されないが、LaTeX から SVG や PNG への変換を行うスクリプトを設定することで表示できる。 

## 依存

LaTeX 数式ソースをコンパイルするために LaTeX が必要となる。
また、結果を PNG に変換するためには dvipng, SVG に変換するためには dvisvgm を使用する。
これらは [texlive-base](https://www.freshports.org/print/texlive-base/) に含まれる。
[texlive-full](https://www.freshports.org/print/texlive-full/) をインストールすればすべて含まれることとなる。

[LibreOffice](https://www.freshports.org/editors/libreoffice) がインストール先であるため必要である。

[poppler](https://poppler.freedesktop.org/) を PDF を SVG や PNG に変換するのに使用する。
日本語を含む数式を使用しないのであれば不要。

## インストール

1. LibreOffice を起動。
2. 「ツール」メニューから「拡張機能マネージャー」を起動。
3. 「ほかの拡張機能をオンラインで取得…」リンクをクリック。ブラウザで LibreOffice の Extensions の検索画面が表示される。
4. 検索ボックスに「TexMaths」と記入して検索。
5. 検索結果から「TexMaths」リンクを選択。
6. [TexMaths ページ](https://extensions.libreoffice.org/en/extensions/show/texmaths-1) の「Release List」から最新版（現在は 0.49） の「DOWNLOAD」をクリック。
7. ダウンロードされる texmaths-0-49.oxt を LibreOffice で開くか「拡張機能マネージャー」画面から開く。
8, 「拡張機能マネージャー」画面に TexMaths 0.49 がリストされていることを確認。
9. LibreOffice の再起動を促すダイアログが出るので再起動する。

## 使用

### 編集

Writer のツールバーの左端に4 つのアイコンが追加されている。
左から

- TexMaths Equations
- TexMaths Numbered Equations
- TexMaths System Configuration
- TexMaths Recompile Equations

となる。左の 2 つが数式の編集ダイアログを起動する。
この 2 つの違いは、Numbered の方が式番号も挿入すること。
編集ダイアログで数式の LaTeX ソースを記述し「LaTeX」ボタンをクリックすることにより、文書に数式の画像が挿入される。
また、文書中にLaTeXソースを記述しておき、そのLaTeXソースを選択してTexMaths Equationsアイコンをクリックすると選択した部分が画像に変換される。
一旦画像にして挿入された数式を選択して TexMaths Equations アイコンをクリックすることにより再編集できる。
挿入する数式の画像を SVG 形式とするか PNG 形式とするかは編集ダイアログの「Preference」ボタンから設定する。PNG の場合は解像度も併せて設定する。
挿入された数式の画像をコピー＆ペーストすると元の LaTeX ソースも併せてコピーされる。

### Writer2LaTeX 拡張での LaTeX ソースへの変換

LibreOffice の [Writer2LaTeX](http://writer2latex.sourceforge.net/) 拡張で Writer 文書を LaTeX ソースに変換する際、TexMaths で入力した LaTeX 数式ソースも出力される。
Write2LaTeX の出力するソースでは日本語等 ASCII 外の文字は16進数でエンコードされているし、そのままでは元の Writer 文書のレイアウトを再現しないので、後処理が必要となる。

## 設定

### TikZ による図

[TikZ](https://texwiki.texjp.org/?TikZ) による図を入れることができる。
数式の編集ダイアログに「Preamble」ボタンがあり、パッケージの使用を宣言できる。
可換図式を描くのであればさらに TikZ ライブラリの使用も記述しておく。

```tex
\usepackage{tikz}
\usetikzlibrary{cd}
```

### upLaTeX による日本語

FreeBSD の TeX Live ばバージョンが 2015 版であり、特に dvisvgm が 1.9.2 であり古いために XeTeX を使用できない。

日本語を数式に入れるために、upLaTeX を使用する。
その際に、プリアンブルで

```tex
\documentclass[uplatex,dvipdfmx]{jsarticle}
```
を指定したいのだが、TexMaths は独自の \documentclass を設定するため Preamble に設定しても無効となる。
そこで latex コマンドの代わりに自作の Python スクリプトを設定し、その中で\documentclass を書き換える。
latex コマンドは TexMaths System Configuration の Paths タブで設定できる。

```py:juplatex
#!/usr/bin/env python

import sys
from os import rename
from os.path import isfile
from re import compile as re_compile
from subprocess import run

TEX = '/usr/local/bin/uplatex'

def main(argv) -> int:
    documentclass_re = re_compile(r'\s*\\documentclass\[.*')
    replacement = r'\\documentclass[uplatex,10pt,dvipdfmx]{jsarticle}'
    for a in argv[1:]:
        if a.lower().endswith('.tex') and isfile(a):
            tmpfile = a + '_uptmp'
            rename(a, tmpfile)
            with open(tmpfile) as infp:
                with open(a, 'w') as outfp:
                    for line in infp.readlines():
                        if documentclass_re.match(line) is not None:
                            line = documentclass_re.sub(replacement, line)
                        outfp.write(line)
    cp = run([TEX] + argv[1:], stdout=sys.stdout, stderr=sys.stderr)
    return cp.returncode

if __name__ == '__main__':
    rc = main(sys.argv)
    sys.exit(rc)
```

latex が出力した DVI を SVG に変換する dvisvgm や PNG に変換するdvipng も同じく TexMaths System Configuration の Paths タブで設定する。

DVI から SVG に変換するために、いったん dvipdfmx で PDF に変換し、それを pdfcrop で紙サイズを切り詰め、pdftocairo で SVG に変換する。
dvipdfmx で指定するフォントマップは自分の環境依存である。
pdfcrop は TeX Live の一部だが、pdftocairo は [Poppler](https://poppler.freedesktop.org/) の一部であり、
FreeBSD の場合はpoppler-utils パッケージに含まれる。
pdftocairo の代わりに mupdf パッケージの mutool も使えるかもしれないが試していない。


```py:jdvipdfmxsvg
#!/usr/bin/env python

import sys
from os import rename
from os.path import isfile
from re import compile as re_compile
from subprocess import run

FAKE_VERSION = 'dvisvgm 1.0.0'
DVIPDFMX = '/usr/local/bin/dvipdfmx'
DVIPDFMX_MAP = 'uptex-ipaex.map'
PDFCROP = '/usr/local/bin/pdfcrop'
PDFTOCAIRO = '/usr/local/bin/pdftocairo'
PDFTOCAIRO_SVG = '-svg'

def main(argv) -> int:
    if '--version' in argv:
        print(FAKE_VERSION)
        return 0
    viewbox_re = re_compile(r'(svg.* )(viewBox="[0-9 .]+")')
    def replace(mo):
        return mo.group(1) + mo.group(2).replace('"', "'")
    for a in argv:
        if len(a) > 4 and a.lower().endswith('.dvi') and isfile(a):
            dvi = a
            stem = a[:-4]
            pdf = stem + '.pdf'
            pdf_crop = stem + '-crop.pdf'
            svg = stem + '.svg'
            command_lines = [[DVIPDFMX, '-f', DVIPDFMX_MAP, dvi],
                             [PDFCROP, pdf],
                             [PDFTOCAIRO, PDFTOCAIRO_SVG, pdf_crop, svg],
                            ]
            for cl in command_lines:
                cp = run(cl, stdout=sys.stdout, stderr=sys.stdout)
                if cp.returncode != 0:
                    return cp.returncode
            tmpfile = svg + '_uptmp'
            rename(svg, tmpfile)
            with open(tmpfile) as infp:
                with open(svg, 'w') as outfp:
                    for line in infp.readlines():
                        line = viewbox_re.sub(replace, line)
                        outfp.write(line)
    return 0

if __name__ == '__main__':
    rc = main(sys.argv)
    sys.exit(rc)
```

PNG への変換も PDF 経由とするため pdfcrop までは SVG と同様である。
ただし、紙サイズの切り詰めは dvipng に対して -T tight での指定となるため、その場合にのみ pdfcrop を呼ぶようにした。TexMaths からは必ず指定されるのだが。
TexMaths がパラメータを変えてくることがあるのが透過の指定の -bg Transparent と解像度の指定の -D \<dpi\> なので、
それに応じて pdftocairo を呼ぶときにそれぞれ -transp スイッチをつけるかどうかと -r \<dpi\> で解像度を指定するかを設定するようにした。

```py:jdvipdfmxpng
#!/usr/bin/env python

import sys
from os.path import isfile
from argparse import ArgumentParser
from subprocess import run

FAKE_VERSION = 'dvipng 1.0'
DVIPDFMX = '/usr/local/bin/dvipdfmx'
DVIPDFMX_MAP = 'uptex-ipaex.map'
PDFCROP = '/usr/local/bin/pdfcrop'
PDFTOCAIRO = '/usr/local/bin/pdftocairo'
PDFTOCAIRO_PNG = '-png'
PDFTOCAIRO_RESOLUTION = '-r'
PDFTOCAIRO_TRANSPARENT = '-transp'
PDFTOCAIRO_SINGLE_PAGE = '-singlefile'

def main(argv) -> int:
    aparser = ArgumentParser()
    aparser.add_argument('files', nargs='*')
    aparser.add_argument('-q', dest='quiet', action='store_true', default=False)
    aparser.add_argument('-T', dest='image_size', action='store', default=None)
    aparser.add_argument('-bg', dest='bg', action='store', default=None)
    aparser.add_argument('--width', dest='show_width', action='store_true', default=False)
    aparser.add_argument('--height', dest='show_height', action='store_true', default=False)
    aparser.add_argument('--depth', dest='show_depth', action='store_true', default=False)
    aparser.add_argument('-D', dest='resolution', action='store', default=None)
    aparser.add_argument('-o', dest='output', action='store', default=None)
    aparser.add_argument('--version', dest='version', action='store', default=None)
    opts = aparser.parse_args(argv[1:])
    if opts.version:
        print(FAKE_VERSION)
        return 0
    for dvi in opts.files:
        if len(dvi) > 4 and dvi.lower().endswith('.dvi') and isfile(dvi):
            stem = dvi[:-4]
            pdf = stem + '.pdf'
            pdf_crop = stem + '-crop.pdf'
            png = stem + '.png'
            if opts.output is None:
                output = stem
            elif opts.output.endswith('.png'):
                output = opts.output[:-4]
            else:
                output = opts.output
            command_lines = [[DVIPDFMX, '-f', DVIPDFMX_MAP, dvi]]
            if opts.image_size == 'tight':
                command_lines.append([PDFCROP, pdf])
            resolution = [PDFTOCAIRO_RESOLUTION, opts.resolution] if opts.resolution is not None else []
            transparent = [PDFTOCAIRO_TRANSPARENT] if opts.bg == 'Transparent' else []
            pdftoppm_line = [PDFTOCAIRO, PDFTOCAIRO_PNG] + resolution + transparent + \
                            [PDFTOCAIRO_SINGLE_PAGE, pdf_crop, output]
            command_lines.append(pdftoppm_line)
            for cl in command_lines:
	        cp = run(cl, stdout=sys.stdout, stderr=sys.stdout)
                if cp.returncode != 0:
                    print(cl, 'failed', cp.returncode)
                    return cp.returncode
    return 0

if __name__ == '__main__':
    rc = main(sys.argv)
    sys.exit(rc)
```

TeX で使うので Lua の方が依存を増やさずに済むが、自分は Lua のプログラムを書いたことがないため Python を使用した。

### 地域化メッセージ

TexMaths は翻訳メッセージファイルを
${HOME}/.config/libreoffice/4/user/uno_packages/cache/uno_packages/lu1436mv810x.tmp_/TexMaths-0-v2.49.oxt/po
ディレクトリ下に持っている。
ここに ja.po ファイルを作成して日本語翻訳を入れれば表示させることができるが翻訳は現在提供されていない。
