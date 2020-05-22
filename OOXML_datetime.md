# 概要

Microsoft Excel 2007以降 のファイル形式（.xlsx）は、複数の XML ファイルと関連するファイルを zip で1つにまとめたものとなっている。
このファイル形式は Office Open XML と呼ばれ Ecma International による ECMA-376 標準および ISO と IEC による ISO/IEC 29500 規格の基となっている。

日付時刻データの扱いは Microsoft Excel の仕様と ECMA-376 1st edition の内容には後方互換性のために問題があり、EMCA-376 2nd edition 以降で改められているが、実装は相変わらず 1st edition を基本とするのが主流である。

# ECMA-376

[Standard ECMA-376 Office Open XML File Formats](https://www.ecma-international.org/publications/standards/Ecma-376.htm) には以下の5つの版がある。
このページには旧版の標準もリストしてあるが、Ecma International では明示されない限り新版の標準が出ると旧版は自動的に廃止されるとしている。
- 1st edition (December 2006)
- 2nd edition (December 2008)
- 3rd edition (June 2011)
- 4th edition (December 2012)
- 5th edition (Part 3, December 2015; and Parts 1 & 4, December 2016)
このうち 4th edition が ISO/IEC 29500 と技術的に同等であると記述されている。

いずれもECMA International から PDF がダウンロード可能であり、1st edition は docx のダウンロードも可能となっている。

1st edition は以下のパートからなる。

- Part 1 - Fundamentals
- Part 2 - Open Packaging Conventions
- Part 3 - Primer
- Part 4 - Markup Language Reference
- Part 5 - Markup Compatibility and Extensibility

2nd edition は以下のパートからなる。

- Part 1 - Fundamentals And Markup Language Reference
- Part 2 - Open Packaging Conventions
- Part 3 - Markup Compatibility and Extensibility
- Part 4 - Transitional Migration Features

以下 5th edition まで 2nd edition と同じパート構成であり、5th edition では Part 2 は更新されておらず Part 2 は 4th edition が最新となっている。

# ISO/IEC 29500

ISO/IEC 29500 は ECMA-376 の 2nd edition 以降と同様の 4 パート構成となっている。また、改訂年も ECMA-376 の 2nd edition 以降と同様となっているが、Part 4 の 2008 年版には 2010 に補遺 (Addendum) と正誤表 (Corrigendum) が追加されている。
ISO 標準も新版の発行をもって旧版は廃止される。旧版は withdrawn と明示されている。

ISO/IEC 29500 は ISO から購入できる。

- Part 1
  - [ISO/IEC 29500-1:2008 Information technology — Document description and processing languages — Office Open XML File Formats — Part 1: Fundamentals and Markup Language Reference
  ](https://www.iso.org/standard/51463.html)
  - [ISO/IEC 29500-1:2011 Information technology — Document description and processing languages — Office Open XML File Formats — Part 1: Fundamentals and Markup Language Reference
  ](https://www.iso.org/standard/59575.html)
  - [ISO/IEC 29500-1:2012 Information technology — Document description and processing languages — Office Open XML File Formats — Part 1: Fundamentals and Markup Language Reference
  ](https://www.iso.org/standard/61750.html)
  - [ISO/IEC 29500-1:2016 Information technology — Document description and processing languages — Office Open XML File Formats — Part 1: Fundamentals and Markup Language Reference
  ](https://www.iso.org/standard/71691.html)

- Part2
  - [ISO/IEC 29500-2:2008 Information technology — Document description and processing languages — Office Open XML File Formats — Part 2: Open Packaging Conventions](https://www.iso.org/standard/51459.html)
  - [ISO/IEC 29500-2:2011 Information technology — Document description and processing languages — Office Open XML File Formats — Part 2: Open Packaging Conventions](https://www.iso.org/standard/59576.html)
  - [ISO/IEC 29500-2:2012 Information technology — Document description and processing languages — Office Open XML File Formats — Part 2: Open Packaging Conventions](https://www.iso.org/standard/61796.html)

- Part 3
  - [ISO/IEC 29500-3:2008
Information technology — Document description and processing languages — Office Open XML File Formats — Part 3: Markup Compatibility and Extensibility](https://www.iso.org/standard/51461.html)
  - [ISO/IEC 29500-3:2011
Information technology — Document description and processing languages — Office Open XML File Formats — Part 3: Markup Compatibility and Extensibility](https://www.iso.org/standard/59577.html)
  - [ISO/IEC 29500-3:2012
Information technology — Document description and processing languages — Office Open XML File Formats — Part 3: Markup Compatibility and Extensibility](https://www.iso.org/standard/61797.html)
  - [ISO/IEC 29500-3:2015
Information technology — Document description and processing languages — Office Open XML File Formats — Part 3: Markup Compatibility and Extensibility](https://www.iso.org/standard/65533.html)

- Part 4
  - [ISO/IEC 29500-4:2008
Information technology — Document description and processing languages — Office Open XML File Formats — Part 4: Transitional Migration Features](https://www.iso.org/standard/51462.html)
  - [ISO/IEC 29500-4:2008/Amd 1:2010
Information technology — Document description and processing languages — Office Open XML File Formats — Part 4: Transitional Migration Features — Amendment 1](https://www.iso.org/standard/54803.html)
  - [ISO/IEC 29500-4:2008/Cor 1:2010
Information technology — Document description and processing languages — Office Open XML File Formats — Part 4: Transitional Migration Features — Technical Corrigendum 1](https://www.iso.org/standard/55002.html)
  - [ISO/IEC 29500-4:2011
Information technology — Document description and processing languages — Office Open XML File Formats — Part 4: Transitional Migration Features](https://www.iso.org/standard/59578.html)
  - [ISO/IEC 29500-4:2012
Information technology — Document description and processing languages — Office Open XML File Formats — Part 4: Transitional Migration Features](https://www.iso.org/standard/61798.html)
  - [ISO/IEC 29500-4:2016
Information technology — Document description and processing languages — Office Open XML File Formats — Part 4: Transitional Migration Features](https://www.iso.org/standard/71692.html)

# Microsoft Excel と Office Open XML 標準

Microsoft の 2015/06/29 付の文書 [Office Open XML と ECMA-376 仕様](https://docs.microsoft.com/ja-jp/previous-versions/office/gg607163(v=office.14)?redirectedfrom=MSDN)
によれば ISO/IEC 29500 には Strict と Transitional の 2 つの規定があり
ECMA-376 の 1st edition と ISO/IEC 29500 の Transitional がほぼ同じで
ECMA-376 の 2nd edition と ISO/IEC 29500 の Strict が同じで
2007 Microsoft Office system は、ECMA-376 Edition 1 標準に準拠するファイルの読み取りと書き込みを行い
Office 2010 は、ECMA-376 Edition 1 に適合するファイルの読み取り、
 ISO/IEC 29500 の Transitional に適合するファイルの読み取りと書き込み、
 および ISO/IEC 29500 の Strict に適合するファイルの読み取りを行う。

記事の日付けは 2015 年だが、
「この記事の執筆の時点で、ECMA-376 標準の 2 つのエディションが存在します。また、ISO が公開する Open XML 標準のエディションもあります。」
とあり、また ISO/IEC 29500 への参照リンクも 2008 年版を指していることから上記の記述は 2008 年版をもとにした記述であると推測できる。

また、Microsoft サポートの文書 [Excel 2013 の新機能](https://support.microsoft.com/ja-jp/office/excel-2013-%e3%81%ae%e6%96%b0%e6%a9%9f%e8%83%bd-1cbc42cd-bfaf-43d7-9031-5688ef1392fd?ui=ja-jp&rs=ja-jp&ad=jp)
によれば

> 新しいファイル形式で保存する
> 
> 新しい Strict Open XML スプレッドシート (*.xlsx) ファイル形式で、ファイルを保存したり開いたりできるようになりました。 このファイル形式では、ISO8601 の日付を読み書きでき、1900 年のうるう年の問題が解決されています。 詳細については、「他のファイル形式でブックを保存する」を参照してください。

となっている。
この [他のファイル形式でブックを保存する](https://support.office.com/ja-jp/article/%E4%BB%96%E3%81%AE%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E5%BD%A2%E5%BC%8F%E3%81%A7%E3%83%96%E3%83%83%E3%82%AF%E3%82%92%E4%BF%9D%E5%AD%98%E3%81%99%E3%82%8B-6a16c862-4a36-48f9-a300-c2ca0065286e) には格納の手順が説明されているが、そこでは Strict Open XML スプレッドシート は Excel ブックのファイル形式の厳密な ISO バージョン (.xlsx)。とされている。

Excel 2013 の発表が 2013 年であるとすると ISO/IEC 29500 は 2012 年版の規格（ECMA-376 4th edition と同等）を指すべきと考えられるが、準拠に関する資料は見つけられていない。

&nbsp; | Excel 2007 | Excel 2010 | Excel 2013
-------|------------|------------|-------------
ECMA-376 Edition 1 | 読み/書き | 読み | 読み
ISO/IEC 29500 Transitional | - | 読み/書き | 読み/書き
ISO/IEC 29500 Strict  | - | 読み | 読み/書き

# LibreOffice と Office Open XML 標準

The Document Foundation の Wiki ページ [The Document Foundation, LibreOffice and OOXML](https://wiki.documentfoundation.org/LibreOffice_OOXML) によれば
The Cocument Foundation は Open Document Format (ODF) を推進するものであり、OOXML は推進もサポートもしないとのことである。
しかし、Microsoft Office 2007 および 2010 との相互運用性を重視するため Microsoft Office のファイル形式 (Microsoft Open XML, MOX と呼び、ISO 標準の OOXML とは何かしら異なるものとしている) の読み書きの機能を提供するとしている。

# ECMA-376 の日付時刻データの取扱い

## workbook の workbookPr (Workbook Properties) 要素
ECMA-376 1st edition の Part 4 の §3.2.28 に workbookPr 要素が定義されている。workbook の子要素として workbook のオプションを指定する。
この要素に date1904 という属性がある。
値が 1 または true の時、ワークブックの日付システムは 1904 開始。0 もしくは false の時、1900 年日付システムが用いられる。デフォルトは false。
ECMA-376 2nd edition 〜 5th edition の Part 1 の §18.2.28 に同様の定義がある。

この日付システムについての解説は 1st edition では Part 4 の §3.17 Formulas の中の §3.17.4 Dates and Times にある。2nd edition 〜 5th edition では Part 1 の §18.17.4 となるが、内容は異なる。

## 日付時刻の表現

### 日付時刻 (1st edition)
日付時刻は1日に1ずつ増加する**非負**の数値であるシリアル値で表される。シリアル値は日付を表す整数部分と時刻を表す小数部分からなることとなる。
シリアル値は数値演算が可能であり、日付時刻を受け取る関数はシリアル値も日付時刻を表す文字列も受け入れる。日付を受け取る関数は時刻部分を無視し、時刻を受け取る関数は日付部分を無視する。

シリアル値には2種類の基準がある。

1900年日付システムでは、最小日付は1900年1月1日であり最小シリアル値は 1。最大日付は 9999年12月31日であり最大シリアル値は 2,958,465。
1904年日付システムでは、最小日付は 1904年1月1日であり最小シリアル値は 0。最大日付は 9999年12月31日であり最大シリアル値は 2,957,003。

歴史的理由により、1900年日付システムを使用する実装は1900年を閏年として扱う。これによりシリアル値 59 が 1900年2月28日に対応し、61が1900年3月1日に対応する。
実在しない1900年2月29日がシリアル値60で存在している。
その結果、1900年1月1日から1900年2月28日の間はWEEKDAY関数は前日の曜日を表す値を帰す。

&nbsp;  | 1900年日付システム | 1904年日付システム
--------------|---------------------|------------
最小日付 | 1900-1-1 | 1904-1-1
最小シリアル値 | 1 | 0
最大日付 | 9999-12-31 | 9999-12-31
最大シリアル値 | 2,958,465 | 2,957,003
1900年が閏年 | Yes | N/A

### 時刻 (1st edition)
時刻を表すシリアル値は 0–0.99999999 の間であり、 0:00:00 (12:00:00 AM) から 23:59:59 (11:59:59 P.M.) までを表す。

1900年日付システムでは 0以上1未満のシリアル値が日付のない時刻を、1以上2,958,466未満の値が日付時刻を表すこととなる。
1904年日付システムでは 0以上1未満のシリアル値は日付のない時刻でもあり得るし1904年1月1日の日付時刻でもあり得る。

### 日付時刻 (2nd edition, 3rd edition)
日付時刻は、日付部分と時刻部分とタイムゾーンからなる ISO 8601 形式の文字列で格納される。
後方互換性のため、実装はセルもしくは計算式中の日付時刻に換算されるシリアル数値を受け入れる。
日付を表すシリアル値は、日付を表す符号付きの整数部と時刻を表す符号なしの小数部からなる。
シリアル値はタイムゾーン情報を持たず、常に UTC の日付時刻を表す。

日付を数値に変換するのに 3 つの異なる基準が用いられる。

1900年日付システムでは、最小日付時刻は -9999年1月1日0時0分0秒で最小シリアル値は -4,346,018。最大日付時刻は 9999年12月基準31日23時59分59秒で最大シリアル値は 2,958,465.9999884。
この日付システムの基準日は1989年12月31日で、そのシリアル値は 0 。
1900年後方互換日付システムでは、最小日付時刻は1900年1月1日 0時0分0秒で最小シリアル値は 1、最大日付時刻は 9999年12月31日23時59分59秒で最大シリアル値は2,958,465.9999884。基準日は1989年12月31日でシリアル値は 0。
1904年後方互換日付システムでは最小日付時刻は 1904年1月1日0時0分0秒で最小シリアル値は 0。最大日付時刻は 9999年12月31日23時59分59秒で最大シリアル値は 2,957,003.9999884。基準日は 1904年1月1日でシリアル値は 0。

歴史的理由により、1900年後方互換日付システムを使用する実装は1900年を閏年として扱う。これによりシリアル値 59 が 1900年2月28日に対応し、61が1900年3月1日に対応する。
実在しない1900年2月29日がシリアル値60で存在している。
その結果、1900年1月1日から1900年2月28日の間はWEEKDAY関数は前日の曜日を表す値を帰す。

1901年3月1日以降に関しては1900年日付システムと1900年後方互換日付システムで同じ日付時刻に対するシリアル値は一致する。1901年1月1日〜1901年2月28日ではシリアル値は1差がある。

&nbsp;        | 1900年日付システム | 1900年後方互換日付システム | 1904年後方互換日付システム
--------------|---------------------|---------------------------|--------------------------
最小日付時刻  | -9999-1-1 00:00:00 | 1900-1-1 00:00:00          | 1904-1-1 00:00:00
最小シリアル値 | -4,346,018        |                          1 | 0
最大日付時刻  | 9999-12-31 23:59:59 | 9999-12-31 23:59:59       | 9999-12-31 23:59:59
最大シリアル値 |  2,958,465.9999884| 2,958,465.9999884          | 2,957,003.9999884
基準日           | 1989-12-30 | 1989-12-31 | 1904-1-1
基準日シリアル値 | 0          |           0 | 0 
1900年が閏年 | No | Yes | N/A

### 時刻 (2nd edition, 3rd edition)
時刻を表すシリアル値は 0–0.99999999 の間であり、 0:00:00 (12:00:00 AM) から 23:59:59 (11:59:59 P.M.) までを表す。

1900年日付システムでは 0以上1未満のシリアル値は日付のない時刻でもあり得るし1989年12月30日の日付時刻でもあり得る。
1900年後方互換日付システムでは 0以上1未満の値が日付のない時刻を、1以上2,958,466未満の値が日付時刻を表すこととなる。
1904年後方互換日付システムでは 0以上1未満のシリアル値は日付のない時刻でもあり得るし1904年1月1日の日付時刻でもあり得る。

### 日付時刻 (4th edition, 5th edition)
日付時刻は、ISO 8601 形式の文字列で格納される。秒の小数部分は最大 3 桁。
最小の日付時刻は 0001年1月1日0時0分0秒、最大の日付時刻は 9999年12月31日23時59分59.999 秒。
歴史にかかわらず常にグレゴリオ暦で閏秒は扱わない。時刻はローカル時刻。
数式での日付もしくは時刻と数値との演算では、シリアル日付値に変換した上で行っ基準付もしくは時刻が必要な箇所に数値が与えられた場合はシリアル日付値として解釈される。

シリアル日付値は基準時刻からの日数を表す数値。有効な日付の範囲内で正でも負でもありえる。
1900年日付システムの基準時刻は 1899年12月30日0時0分0 秒。
1904年日付システムの基準時刻は 1904年1月1日0時0分0秒。
時刻のシリアル値は 0 以上 1 未満。

&nbsp;        | 1900年日付システム |  1904年日付システム
--------------|---------------------|----------------------------
最小日付時刻  |  0001-1-1 00:00:00   | 0001-1-1 00:00:00
最小シリアル値 | -693593            |-695055
最大日付時刻  | 9999-12-31 23:59:59.999 | 9999-12-31 23:59:59.999
最大シリアル値 |  2,958,465.9999884|  2,957,003.9999884
基準日           |  1989-12-30 | 1904-1-1
基準日シリアル値 | 0          |           0 | 0 
1900年が閏年 | No  | No

### 時刻 (4th edition, 5th edition)
時刻を表すシリアル値は 0–0.99999999 の間であり、 0:00:00 (12:00:00 AM) から 基準年12月30日の日付時刻でもありえる。
1904年日付システムでは 0以上1未満のシリアル値は日付のない時刻でもありえるし1904年1月1日の日付時刻でもありえる。

### edition の比較

1st edition は 1900年が閏年として扱うため 1900年2月までの日付の曜日がずれるという問題がある。また 1899年までの日付を表すことができない。
2nd edition では、-9999 年まで遡れる日付システムを導入したが、後方互換性はなく、互換性のために 1st edition と同等の日付システムを残している。しかし、1st edition がタイムゾーンの考慮がないのに対し 2nd edition ではタイムゾーン明記とシリアル値は UTC を表すことにしたため互換性はない。
4th edition では、タイムゾーンを UTC からローカル時刻に改め ISO 8601 文字列からもタイムゾーンを削除した点においては 1st edition との互換性を改善したが、1900年を閏年とする日付システムを廃止したため 1900年1月と2月の日付データに関しては互換性がない。

&nbsp; | 1st edition | 2nd, 3rd editions | 4th, 5th editions
-------|-------------|-------------------|---------
セルの内容 | シリアル値 | ISO 8601文字列（タイムゾーン付き） | ISO 8601文字列（タイムゾーンなし）、秒の小数部分は3桁以下
シリアル値のタイムゾーン | 指定なし | UTC | ローカル時刻
最小の日付 | 1901-01-01 | -9999-01-01 | 0001-01-01
最大の日付 | 9999-12-31 | 9999-12-31 | 9999-12-31
1900年が閏年となるシステム | あり | あり | なし

## 精度
数値の有効な範囲は絶対値が m × 2^n で m  が 2^53 未満の整数、n が -1075  以上 970 以下の整数。 
2^53 == 9007199254740992 であり、10進では15桁程度までということになる。
これは edition 共通。

日付時刻のシリアル値は 2,958,466 未満の値であるので、最大日付のあたりでは  2,958,466 / 2^53 = 3.2845570707706884e-10 であることから 1 マイクロ秒 = 1.1574074074074074e-11 までの精度を持つことはできないこととなる。1/10 ミリ秒 = 1.1574074074074074e-09 まではセーフ。
4th edition と 5th edition では、ISO 8601 文字列表記で秒の小数部分を 3 桁まですなわちミリ秒までとしているのでシリアル値で表せる範囲内となっている。
1 秒はシリアル値では 1/(60*60*24) = 1/86400 であり 10 進でも 2 進でも循環小数となるため、精度の制限までの表現となる。

## XMLでの表記
数値の表記は10進の仮数部に続きオプションで E もしくは e と 10進の指数部。
シリアル値の表記は可能な限り高い精度での数値の表記。
これは edition 共通。

# Microsoft Excel の挙動
Microsoft Excel 2016 (Windows) で、Excel ブック (.xlsx) 形式でファイルを保存すると日付時刻の値はシリアル値で格納される。
例として 1900年日付システムで 2018-09-10 12:31:45 は 43353.522048611114 となる。

```xml
<v>43353.522048611114</v>
```
Strict Open XML スプレッドシート　(.xlsx) 形式でファイルを保存すると、日付時刻の値は ISO 8601 文字列で格納されるが、秒の小数部分は入力と一致せず 3 桁も超えている。タイムゾーン情報は付加されない。

```xml
<v>2018-09-10T12:31:45.00000023748725325</v>
```

1900年を閏年としている挙動については わがせ 氏の記事 [Excelの日付データは1900年1月1日から数えた連番ではない 1900年うるう年問題](https://qiita.com/wagase/items/6444ce93a24aaaa74981) 参照。

# LibreOffice の挙動
LibreOffice calc 6.3.4_8 (FreeBSD) で Excel 2007-365 (.xlsx)形式でファイルを保存すると日付時刻の値はシリアル値で格納される。
例として 1900年日付システムで 2018-09-10 12:31:45 は 43353.52204861111 となる。Excel の場合より 1 桁短い。

```xml
	<v>43353.5220486111
	</v>
```
Office Open XML 表計算ドキュメント (.xlsx) という形式でも保存できるが、やはり日付時刻はシリアル値で格納される。

1900-01-01 はシリアル値が 2 となり Microsoft Excel とはずれがある。また 1899-12-29 はシリアル値が -1 となる。
