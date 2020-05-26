# 使用バージョン

- LibreOffice 6.3.4_7 on FreeBSD

# 更新内容

- 説明の追加
- 出力フィルタでの文字エンコーディングの扱いの記述を UTF-8 使用可に変更。

# LibreOffice のスプレッドシートファイル形式変換コマンドライン

コマンドラインから libreoffice に --convert-to オプションスイッチで変換先のファイル形式を指定して起動することでファイル形式の変換を行える。
GUI は起動されず、libreoffice は変換のみを行って終了する。
変換後のファイルはカレントディレクトリに作成され、ファイル名は元のファイル名のサフィックス（拡張子）を変更したものとなる。
元ファイルを複数指定すればすべて変換される。

ファイルの入出力のフィルタとフィルタのオプションを設定できる。

```sh
$ libreoffice --convert-to <ファイル形式>[:<出力フィルタ名>[:<出力フィルタオプション>]] [<入力フィルタ名>[:<入力フィルタオプション>]] {<ファイル名>}
```

## ods / xls / xlsx → ods / xls / xlsx

ods (Open Document Format Spreadsheet), xls (Microsoft Excel 2003 まで), xlsx (Microsoft Excel 2007 以降）の入力においてはフィルタオプションを指定することはない。

```sh
$ libreoffice --convert-to <ファイル形式> {<ファイル名>}
```

例：

```sh
$ libreoffice --convert-to xlsx somespreadsheet.ods
```

## csv → ods / xls / xlsx (入力ファイルの形式・文字エンコーディングの指定)

CSV の読み込みに関しては CSV の形式を入力フィルタのフィルタオプションで指定する。
特に文字エンコーディングを指定しないと Shift_JIS,  UTF-8 いずれの CSV ファイルも文字化けする結果となるため、日本語の文字列を含むファイルの場合は必須となる。

```sh
$ libreoffice --convert-to <ファイル形式> --infilter="csv:<フィルタオプション>" <ファイル名>
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
詳細は後述。

`--convert-to` と変換先ファイル形式の間は空白で `--infilter` とフィルタ指定の間は「=」である必要がある。そのとおりに指定しないとエラーとなる。

## ods / xls / xlsx → csv (出力ファイルの形式・文字エンコーディングの指定)

CSV の書き出しに関しては CSV の形式を出力フィルタのフィルタオプションで指定する。
文字エンコーディングを指定しないと漢字やかなは代替文字「?」で置き換えられた ASCII ファイルとなるため必須となる。
以前にテストしたときにはデフォルトの出力が Shift_JIS 固定となり UTF-8 出力できなかったが LibreOffice の更新後に試したところデフォルトが ASCII 出力で指定により Shift_JIS, UTF-8 とも出力可能であった。

```sh
$ libreoffice --convert-to csv:"Text - txt - csv (StarCalc):<フィルタオプション>" <ファイル名>
```

例：

```sh
$ libreoffice --convert-to csv:"Text - txt - csv (StarCalc):44,34,76,,,,,,true" somespreadsheet.xlsx
```

ここで

- 44 はカラムの間はカンマ区切り
- 34 はダブルクォートをテキスト区切り（クォーテーション）に使う
- 76 は文字エンコーディングが UTF-8
- 9項目めの true は GUI の表示のとおりに出力

であることを表す。詳細は後述。

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
- 5番めのトークンは、先頭4カラムめまでの読み込み形式の指定。詳細は上記ページを参照。ただし試してみると指定にかかわらずすべて文字列となった。
- 6番めのトークンは言語識別番号。省略するとUIの言語識別番号が使用される。この番号については Microsoft のページ [Language Identifier Constants and Strings](https://docs.microsoft.com/ja-jp/windows/win32/intl/language-identifier-constants-and-strings?redirectedfrom=MSDN) を参照するように指定されているがリンク先では言語識別番号は廃止されているとされている。
おそらく日本語の指定は 1041 であるが通常は指定不要と考えられる。
- 7番めのトークンは、入力フィルターの場合はクォーテーションされているデータをテキストとして扱う指定、出力フィルターの場合はすべてのカラムをクォーテーションして出力する指定の真偽値。デフォルトは false。
- 8番めのトークンは、入力フィルターの場合は日付や指数表現を検出する指定の真偽値。出力フィルターの場合は数値を数値として出力する指定。デフォルトは false。
- 9番めのトークンは、出力フィルターの場合のみ有効でセルの内容を表示どおりに出力する指定の真偽値。デフォルトは false。