# 概要

Open Document Format (ODF) は、OASIS が標準として定めた複数の XML ファイルと関連するファイルを zip で1つにまとめたオフィスファイル形式である。
ISO と IEC による ISO/IEC 26300 規格と日本の JIS  X 4401 規格の基であり、
LibreOffice, Apache Open Office の標準ファイルフォーマットである。

# OASIS 標準

OASIS の Open Document Format 標準には 3 つの版があり、それぞれダウンロード可能である。

- [OpenDocument Format for Office Applications (OpenDocument) v1.0](https://www.oasis-open.org/standards#opendocumentv1.0) (May 2005)
- [OpenDocument Format for Office Applications (OpenDocument) v1.1](https://www.oasis-open.org/standards#opendocumentv1.1) (February 2007)
- [Open Document Format for Office Applications (OpenDocument) v1.2](https://www.oasis-open.org/standards#opendocumentv1.2) (September 2011)

第 1.2 版は以下の3 つのパートからなる。

- OpenDocument v1.2 part 1: OpenDocument Schema
- OpenDocument v1.2 part 2: Recalculated Formula (OpenFormula) Format
- OpenDocument v1.2 part 3: Packages

# ISO/IEC 26300

ISO/IEC 26300 も3 つのパートからなり、1.0 〜 1.2 版がある。
ISO/IEC 規格は ISO から購入できる。

- Version 1.0
  - [ISO/IEC 26300:2006
Information technology — Open Document Format for Office Applications (OpenDocument) v1.0](https://www.iso.org/standard/43485.html)
  - [ISO/IEC 26300:2006/Cor 1:2010
Information technology — Open Document Format for Office Applications (OpenDocument) v1.0 — Technical Corrigendum 1](https://www.iso.org/standard/55752.html)
  - [ISO/IEC 26300:2006/Cor 2:2011
Information technology — Open Document Format for Office Applications (OpenDocument) v1.0 — Technical Corrigendum 2](https://www.iso.org/standard/59192.html)
  - [ISO/IEC 26300:2006/Cor 3:2014
Information technology — Open Document Format for Office Applications (OpenDocument) v1.0 — Technical Corrigendum 3](https://www.iso.org/standard/66551.html)

- Version 1.1

  - [ISO/IEC 26300:2006/Amd 1:2012
Information technology — Open Document Format for Office Applications (OpenDocument) v1.0 — Amendment 1: Open Document Format for Office Applications (OpenDocument) v1.1](https://www.iso.org/standard/59302.html)
  - [ISO/IEC 26300:2006/Amd 1:2012/Cor 1:2014
Information technology — Open Document Format for Office Applications (OpenDocument) v1.0 — Amendment 1: Open Document Format for Office Applications (OpenDocument) v1.1 — Technical Corrigendum 1](https://www.iso.org/standard/66550.html)

- Version 1.2

  - [ISO/IEC 26300-1:2015
Information technology — Open Document Format for Office Applications (OpenDocument) v1.2 — Part 1: OpenDocument Schema](https://www.iso.org/standard/66363.html)
  - [ISO/IEC 26300-2:2015
Information technology — Open Document Format for Office Applications (OpenDocument) v1.2 — Part 2: Recalculated Formula (OpenFormula) Format](https://www.iso.org/standard/66375.html)
  - [ISO/IEC 26300-3:2015
Information technology — Open Document Format for Office Applications (OpenDocument) v1.2 — Part 3: Packages](https://www.iso.org/standard/66376.html)

# JIS X 4401

日本産業規格(JIS) では 2010 年に 1.0 版を 2014 年に 1.1 版を規格化している。
[日本産業標準調査会 (JISC) のJIS検索](https://www.jisc.go.jp/app/jis/general/GnrJISSearch.html) で検索して閲覧できる。閲覧できるのは最新版のみで、ダウンロードや印刷はできない。
規格本体は日本規格協会から購入できる。

# Open Document Format (Spreadsheet) の日付時刻の取扱い

OASIS 標準第 1.2 版の 18. Datetypes によれば、日付時刻関係では以下の [xmlschema-2] データ型が用いられる。

- date
- time
- dateTime
- duration

xmlschema-2 は [XML Schema Part 2: Datatypes Second Edition W3C Recommendation 28 October 2004](https://www.w3.org/TR/2004/REC-xmlschema-2-20041028/) である。
上記 4 データ型は 3.2 Primitive datatypes に含まれる。

dateTime の表記は  '-'? yyyy '-' mm '-' dd 'T' hh ':' mm ':' ss ('.' s+)? (zzzzzz)? となっている。
タイムゾーンはオプションだが、タイムゾーン付の表記の日付時刻は UTC に変関して取り扱われる。
タイムゾーンなしの表記の場合は不特定の場所のタイムゾーンの時刻と仮定される。

date ではタイムゾーンはオプションだが、タイムゾーン付の date は 1 日の開始時刻を追跡する。

time ではタイムゾーンはオプション。

duration の表記は ISO 8601 拡張形式 PnYnMnDTnHnMnS で表される。
nY はn年、nM は nヶ月、nD は n日、T は日付時刻セパレータ、nH はn時間、nM はn分、nSはn秒。
