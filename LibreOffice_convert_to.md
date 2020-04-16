# 使用バージョン

- LibreOffice 6.3.4.2 on FreeBSD

# LibreOffice のスプレッドシートファイル形式変換コマンドライン

## ods / xls / xlsx → ods / xls / xlsx / CSV

```sh
$ libreoffice --convert-to <ファイル形式> <ファイル名>
```

CSV は Shift_JIS で作成される。

例：

```sh
$ libreoffice --convert-to xlsx somespreadsheet.ods
```

## CSV → ods / xls / xlsx (入力ファイルの形式・文字エンコーディングの指定)

```sh
$ libreoffice --convert-to <ファイル形式> --infilter="csv:<パラメータ>" <ファイル名>
```

例：

```sh
$ libreoffice --convert-to xlsx --infilter="44,34,76" somespreadsheet.csv
```

ここで

- 44 はカラムの間はカンマ区切り
- 34 はダブルクォートをテキスト区切り（クォーテーション）に使う
- 76 は文字エンコーディングが UTF-8

であることを表す。
`--convert-to` と変換先ファイル形式の間は空白で `--infilter` とフィルタ指定の間は「=」である必要がある。そのとおりに指定しないとエラーとなる。

# 詳細

## LibreOffice のコマンドライン起動

LibreOffice の Wikiページ [LibreOffice ソフトウェアをパラメーターを指定して起動する](https://help.libreoffice.org/Common/Starting_the_Software_With_Parameters/ja) がコマンドラインパラメータを記載している。
このうち --convert-to output_file_extension\[:output_filter_name\] \[--outdir output_dir\] files
パラメータがファイルのバッチ変換の指定である。
--convert-to を指定すると GUI の画面は表示されず変換だけを行って終了する。

また --infilter={filter} を指定すると入力時にフィルターを適用することができる。

このコマンドラインパラメータの表ではスイッチと引数の間が空白であるもの（--convert-to が該当）と = であるもの（--infilter が該当）するものがあるが、これは表の記載のとおりに使い分ける必要がある。

## フィルター

入力フィルターは --infilter で、出力フィルターは --convert-to で指定するようになっている。

フィルターとオプションの一覧については Apache OpenOffice の Wiki ページ [Filter Options](https://wiki.openoffice.org/wiki/Documentation/DevGuide/Spreadsheets/Filter_Options) が参考となる。
LibreOffice の記述ではないため差異がある場合は挙動から判断することになる。

この表中では Text - txt - csv (StarCalc) フィルターが入力・出力ともに仕様できることとなっている。
LibreOffice の挙動を見ると、入力フィルターの指定では Text - txt - csv (StarCalc) の代わりに csv もしくは CSV と書いても同様に作用するようである。
出力フィルターの指定では Text - txt - csv (StarCalc) だけが有効な記述であった。

フィルターのオプションをフィルター名の後ろにコロンで区切って記述できる。
オプションはカンマで区切られた複数のトークンからなり、入力フィルターの場合は [テキストのインポート](https://help.libreoffice.org/Common/Text_Import/ja) のUIでの指定に対応する。 
出力フィルターの場合は [テキストファイルのエクスポート](https://help.libreoffice.org/Common/Export_text_files/ja)のUIでの指定に対応する。

- 1番めのトークンは、カラムを区切る記号のASCII コードで記述する。複数の文字を使う場合は / で区切る。
カラムをカンマ（,）で区切る場合は 44 となる。
- 2番めのトークンは、テキストの区切り記号（クォーテーション）の ASCII コードを記述する。ダブルクォート（"）の場合は 34 となる。
- 3番めのトークンは、文字エンコーディングを番号で指定する。文字エンコーディングの表は同じページの Filter Options for Lotus, dBase and DIF Filters のセクションにある。省略すると日本語の文字を正しく読み込めない。また、出力フィルターの場合には指定すると文字エンコーディングは指定どおりとなったがデータが壊れてしまい期待どおりの結果が選られなかった。
日本語に該当するものは
  - Windows-932 (Japanese) : 60
  - Shift-JIS (Japanese) : 64
  - EUC-JP (Japanese) : 69
  - ISO-2022-JP (Japanese) : 72
  - Unicode (UTF-8) : 76
  - JIS 0201 (Japanese) : 81
  - JIS 0208 (Japanese) : 82
  - JIS 0212 (Japanese) : 83
  - Unicode (Java's modified UTF-8) : 90
  - Unicode UCS4 : 65534 
  - Unicode UCS2 : 65535

- 4番めのトークンは、開始行、デフォルトで1。
- 5番めのトークンは、先頭4カラムめまでの読み込み形式の指定。詳細は上記ページを参照。
- 6番めのトークンは言語識別番号。省略するとUIの言語識別番号が使用される。この番号については Microsoft のページ [Language Identifier Constants and Strings](https://docs.microsoft.com/ja-jp/windows/win32/intl/language-identifier-constants-and-strings?redirectedfrom=MSDN) を参照するように指定されているがリンク先では言語識別番号は廃止されているとされている。
おそらく日本語の指定は 1041 であるが通常は指定不要と考えられる。
- 7番めのトークンは、入力フィルターの場合はクォーテーションされているデータをテキストとして扱う指定、出力フィルターの場合はすべてのカラムをクォーテーションして出力する指定の真偽値。デフォルトは false。
- 8番めのトークンは、入力フィルターの場合は日付や指数表現を検出する指定の真偽値。出力フィルターの場合は数値を数値として出力する指定。デフォルトは false。
- 9番めのトークンは、出力フィルターの場合のみ有効でセルの内容を表示どおりに出力する指定の真偽値。デフォルトは false。




