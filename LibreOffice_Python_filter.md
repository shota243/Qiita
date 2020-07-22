# 使用バージョン

- LibreOffice 6.4.4
- Python 3.7.7

# 概要

LibreOffice で文書を読み込むまたは書き出す際には入出力フィルタによりファイル形式と内部データとの変換を行う。
フィルタは [--convert-to スイッチによりコマンドラインからファイル形式を変換する](https://qiita.com/shota243/items/0ef67d786785bcf5b60f) 時とマクロないしプログラムからファイルを読み書きする際にフィルタ名とフィルタオプションで指定する。
フィルタの一覧は LibreOffice のインストールディレクトリ下のファイルもしくは UNO インタフェースにて取得できる。

# Python-UNO bridge からの入出力フィルタの利用

## 文書読み込み

文書は[com.sun.star.frame.Desktop](https://api.libreoffice.org/docs/idl/ref/servicecom_1_1sun_1_1star_1_1frame_1_1Desktop.html) インタフェースが XComponentLoader インタフェースから継承している [loadComponentFromURL](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1frame_1_1XComponentLoader.html#a6f89db7d45da267af47d1acf01cd986d)() メソッドにより読み込む。
第4引数の Arguments の中でフィルタ名やフィルタオプションを指定できる。Arguments の型は ```sequence< com::sun::star::beans::PropertyValue >```　となっているが、Python-UNO bridge の場合は com.sun.star.beans.PropertyValue インスタンスのタプルということになる。
Arguments については [```com::sun::star::document::MediaDescriptor```](https://api.libreoffice.org/docs/idl/ref/servicecom_1_1sun_1_1star_1_1document_1_1MediaDescriptor.html) 参照とされていて、
[FilterName](https://api.libreoffice.org/docs/idl/ref/servicecom_1_1sun_1_1star_1_1document_1_1MediaDescriptor.html#a777c03d61101090a1539aafc6ba1c4ca) は [TypeDetection](https://api.libreoffice.org/docs/idl/ref/servicecom_1_1sun_1_1star_1_1document_1_1TypeDetection.html) の構成にマッチしなければいけないと書いてあるが実際には TypeDetection ではなく [FilterFactory](https://api.libreoffice.org/docs/idl/ref/servicecom_1_1sun_1_1star_1_1document_1_1FilterFactory.html) サービスの Element に一致する必要がある。同じページからリンクされている[フィルタ名の一覧](https://help.libreoffice.org/latest/en-US/text/shared/guide/convertfilters.html)も正しくないようだ。

Arguments でフィルタ名を指定しない場合、ファイルの内容により可能であれば自動判定される。

```py
import uno

url = uno.systemPathToFileUrl('test.ods')
desktop = XSCRIPTCONTEXT.getDesktop()
document = desktop.loadComponentFromURL(url, '_blank', 0, ())
```

自動判定は必ずしも有効ではなく、CSV ファイルを指定すると Calc の CSVインポートのダイアログが立ち上がる。
フィルタ名とフィルタオプションを指定することでプログラムからインポート方法を指定できる。

```py
import uno
from com.sun.star.beans import PropertyValue

url = uno.systemPathToFileUrl('test.csv')
desktop = XSCRIPTCONTEXT.getDesktop()
argument = (PropertyValue(Name='FilterName', Value='Text - txt - csv (StarCalc)'),
	    PropertyValue(Name='FilterOptions', Value='44,34,76'))
document = desktop.loadComponentFromURL(url, '_blank', 0, argument)
```

## 文書書き出し

文書の書き出しは [XComponent](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1lang_1_1XComponent.html) インタフェースが継承している [XStorable](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1frame_1_1XStorable.html) インタフェースの [store](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1frame_1_1XStorable.html#af5d1fdcbfe78592afb590a4c244acf20)(), [storeAsURL](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1frame_1_1XStorable.html#a0f08ed5f6c8aa02da2ea146c966d4809)(), [storeToURL](https://api.libreoffice.org/docs/idl/ref/interfacecom_1_1sun_1_1star_1_1frame_1_1XStorable.html#af48930bc64a00251aa50915bf087f274)() メソッドで行う。
store() は GUI からの save (保存) 相当で読み込んだファイルに書き出すのでフィルタも読み込みと同じ物が用いられる。
storeAsURL() は save as 相当で、書き出し先のファイルを指定して保存。書き出し先のファイルとフィルタが新たなファイル、フィルタとして文書に記録される。使用するフィルタは入力（インポート）と出力（エクスポート）の両方ができる必要がある。
storeToURL() は export 相当で、書き出し先のファイルを指定してエキスポート。文書のファイル名とフィルタは更新されない。出力専用フィルタが使用できる。
フィルタを指定しない場合デフォルトのフィルタが使用される。出力ファイル形式は ODF となる。

```py
import uno
from com.sun.star.beans import PropertyValue

document = XSCRIPTCONTEXT.getDocument()
url = uno.systemPathToFileUrl('test.csv')
argument = (PropertyValue(Name='FilterName', Value='Text - txt - csv (StarCalc)'),
	    PropertyValue(Name='FilterOptions', Value='44,34,76,,,,,,true'))
document.storeAsURL(url, argument)
```

# 使用できるフィルタ名

## Python-UNO bridge からの取得

入出力フィルタは [FilterFactory](https://api.libreoffice.org/docs/idl/ref/servicecom_1_1sun_1_1star_1_1document_1_1FilterFactory.html) サービスにて一覧できる。
各フィルタは名前で識別される。フィルタのプロパティにすいては Apache OpenOffice Wiki の [Properties of a Filter](https://wiki.openoffice.org/wiki/Documentation/DevGuide/OfficeDev/Properties_of_a_Filter) に記述がある。
FilterFactory サービスは、LibreOffice が元々持っているフィルタに加えて拡張機能が追加したフィルタも一括して管理している。

DocumentService プロパティは対象とする文書の種類を識別する。com.sun.star.text.TextDocument なら Writer文書、com.sun.star.sheet.SpreadsheetDocument なら Calcスプレッドシート。これは各文書の getIdentifier() メソッドの返す値と一致する。
Type プロパティは入出力されるファイルの型を識別する。このプロパティに保持されるのは Type の名前であり、各 Type については TypeDetection サービスでプロパティを取得できる。
Flags プロパティは複数のフラグを合わせた物である。IMPORT フラグは入力フィルタであることを示し EXPORT フラグは出力フィルタであることを示す。DEFAULT フィルタは DocumentService のデフォルトのフィルタであることを示し、PREFERRED フィルタは Type に対して優先して使用されるフィルタであることを示す。

[TypeDetection](https://api.libreoffice.org/docs/idl/ref/servicecom_1_1sun_1_1star_1_1document_1_1TypeDetection.html) サービスはファイルの型を定義する。
各 Type は名前で識別される。
Extensions プロパティは対象とするファイルの拡張子を列記している。

以下は FilterFactory サービスから取得したフィルタの一覧で、一部のプロパティと対応する Type の一部のプロパティの表である。Flags は IMPORT, EXPORT, DEFAULT, PREFERRED に限定している。

Name|DocumentService|UIName|Flags|Type|Extensions|MediaType
----+---------------+------+-----+----+----------+---------
StarOffice XML (Base) Report Chart|com.sun.star.chart2.ChartDocument|OpenOffice.org 1.0 レポートグラフ|IMPORT EXPORT DEFAULT|StarBaseReportChart|odc|application/vnd.sun.xml.report.chart
StarOffice XML (Chart)|com.sun.star.chart2.ChartDocument|OpenOffice.org 1.0 グラフ|IMPORT|chart_StarOffice_XML_Chart|sxs|application/vnd.sun.xml.chart
chart8|com.sun.star.chart2.ChartDocument|ODF グラフ|IMPORT EXPORT DEFAULT PREFERRED|chart8|odc|application/vnd.oasis.opendocument.chart
BMP - MS Windows|com.sun.star.drawing.DrawingDocument|BMP - Windows Bitmap|IMPORT|bmp_MS_Windows|bmp|image/x-MS-bmp
ClarisWorks_Draw|com.sun.star.drawing.DrawingDocument|ClarisWorks/AppleWorks 図形描画|IMPORT PREFERRED|draw_ClarisWorks|cwk|application/clarisworks
Corel Draw Document|com.sun.star.drawing.DrawingDocument|Corel Draw|IMPORT PREFERRED|draw_CorelDraw_Document|cdr|application/vnd.corel-draw
Corel Presentation Exchange|com.sun.star.drawing.DrawingDocument|Corel Presentation Exchange|IMPORT PREFERRED|draw_Corel_Presentation_Exchange|cmx|image/x-cmx
DXF - AutoCAD Interchange|com.sun.star.drawing.DrawingDocument|DXF - AutoCAD Interchange Format|IMPORT|dxf_AutoCAD_Interchange|dxf|image/vnd.dxf
EMF - MS Windows Metafile|com.sun.star.drawing.DrawingDocument|EMF - Enhanced Metafile|IMPORT|emf_MS_Windows_Metafile|emf|image/x-emf
EPS - Encapsulated PostScript|com.sun.star.drawing.DrawingDocument|EPS - Encapsulated PostScript|IMPORT|eps_Encapsulated_PostScript|eps|image/x-eps
Freehand Document|com.sun.star.drawing.DrawingDocument|Adobe/Macromedia Freehand|IMPORT PREFERRED|draw_Freehand_Document|fh fh1 fh2 fh3 fh4 fh5 fh6 fh7 fh8 fh9 fh10 fh11|image/x-freehand
GIF - Graphics Interchange|com.sun.star.drawing.DrawingDocument|GIF - Graphics Interchange Format|IMPORT|gif_Graphics_Interchange|gif|image/gif
JPG - JPEG|com.sun.star.drawing.DrawingDocument|JPEG - Joint Photographic Experts Group|IMPORT|jpg_JPEG|jpg jpeg jfif jif jpe|image/jpeg
MET - OS/2 Metafile|com.sun.star.drawing.DrawingDocument|MET - OS/2 Metafile|IMPORT|met_OS2_Metafile|met|image/x-met
MOV - MOV|com.sun.star.drawing.DrawingDocument|MOV - QuickTime ファイルフォーマット|IMPORT|mov_MOV|mov MOV|application/movie
MWAW_Bitmap|com.sun.star.drawing.DrawingDocument|古いMacのビットマップ|IMPORT PREFERRED|MWAW_Bitmap|*|
MWAW_Drawing|com.sun.star.drawing.DrawingDocument|古いMacのドロー形式|IMPORT PREFERRED|MWAW_Drawing|*|
OpenDocument Drawing Flat XML|com.sun.star.drawing.DrawingDocument|Flat XML ODF 図形描画|IMPORT EXPORT|draw_ODG_FlatXML|fodg odg xml|application/vnd.oasis.opendocument.graphics-flat-xml
PBM - Portable Bitmap|com.sun.star.drawing.DrawingDocument|PBM - Portable Bitmap|IMPORT|pbm_Portable_Bitmap|pbm|image/x-portable-bitmap
PCT - Mac Pict|com.sun.star.drawing.DrawingDocument|PCT - Mac Pict|IMPORT|pct_Mac_Pict|pct pict|image/x-pict
PCX - Zsoft Paintbrush|com.sun.star.drawing.DrawingDocument|PCX - Zsoft Paintbrush|IMPORT|pcx_Zsoft_Paintbrush|pcx|image/x-pcx
PGM - Portable Graymap|com.sun.star.drawing.DrawingDocument|PGM - Portable Graymap|IMPORT|pgm_Portable_Graymap|pgm|image/x-portable-graymap
PNG - Portable Network Graphic|com.sun.star.drawing.DrawingDocument|PNG - Portable Network Graphic|IMPORT|png_Portable_Network_Graphic|png|image/png
PPM - Portable Pixelmap|com.sun.star.drawing.DrawingDocument|PPM - Portable Pixelmap|IMPORT|ppm_Portable_Pixelmap|ppm|image/x-portable-pixmap
PSD - Adobe Photoshop|com.sun.star.drawing.DrawingDocument|PSD - Adobe Photoshop|IMPORT|psd_Adobe_Photoshop|psd|image/vnd.adobe.photoshop
PageMaker Document|com.sun.star.drawing.DrawingDocument|Adobe PageMaker|IMPORT PREFERRED|draw_PageMaker_Document|p65 pm pm6 pmd|application/x-pagemaker
Publisher Document|com.sun.star.drawing.DrawingDocument|Microsoft Publisher 98-2010|IMPORT PREFERRED|draw_Publisher_Document|pub|application/x-mspublisher
QXP Document|com.sun.star.drawing.DrawingDocument|QuarkXPress|IMPORT PREFERRED|draw_QXP_Document|qxd qxt|
RAS - Sun Rasterfile|com.sun.star.drawing.DrawingDocument|RAS - Sun Raster Image|IMPORT|ras_Sun_Rasterfile|ras|image/x-cmu-raster
SVG - Scalable Vector Graphics Draw|com.sun.star.drawing.DrawingDocument|SVG - スケーラブル・ベクター・グラフィックス|IMPORT PREFERRED|svg_Scalable_Vector_Graphics_Draw|svg svgz|image/svg+xml
SVM - StarView Metafile|com.sun.star.drawing.DrawingDocument|SVM - StarView Metafile|IMPORT|svm_StarView_Metafile|svm|image/x-svm
StarOffice XML (Draw)|com.sun.star.drawing.DrawingDocument|OpenOffice.org 1.0 図形描画|IMPORT PREFERRED|draw_StarOffice_XML_Draw|sxd|application/vnd.sun.xml.draw
StarOffice_Drawing|com.sun.star.drawing.DrawingDocument|古い StarOffice 図形描画|IMPORT PREFERRED|StarOffice_Drawing|sda|
TGA - Truevision TARGA|com.sun.star.drawing.DrawingDocument|TGA - Truevision Targa|IMPORT|tga_Truevision_TARGA|tga|image/x-targa
TIF - Tag Image File|com.sun.star.drawing.DrawingDocument|TIFF - Tagged Image File Format|IMPORT|tif_Tag_Image_File|tif tiff|image/tiff
Visio Document|com.sun.star.drawing.DrawingDocument|Microsoft Visio 2000-2013|IMPORT PREFERRED|draw_Visio_Document|vdx vsd vsdm vsdx|application/vnd.visio
WMF - MS Windows Metafile|com.sun.star.drawing.DrawingDocument|WMF - Windows Metafile|IMPORT|wmf_MS_Windows_Metafile|wmf|image/x-wmf
WordPerfect Graphics|com.sun.star.drawing.DrawingDocument|WordPerfect Graphics|IMPORT PREFERRED|draw_WordPerfect_Graphics|wpg|image/x-wpg
XBM - X-Consortium|com.sun.star.drawing.DrawingDocument|XBM - X Bitmap|IMPORT|xbm_X_Consortium|xbm|image/x-xbitmap
XHTML Draw File|com.sun.star.drawing.DrawingDocument|XHTML|EXPORT|XHTML_File|html xhtml|application/xhtml+xml
XPM|com.sun.star.drawing.DrawingDocument|XPM - X PixMap|IMPORT|xpm_XPM|xpm|image/x-xpixmap
ZMF Document|com.sun.star.drawing.DrawingDocument|Zoner Callisto/図形描画|IMPORT PREFERRED|draw_ZMF_Document|zmf|
draw8|com.sun.star.drawing.DrawingDocument|ODF 図形描画|IMPORT EXPORT DEFAULT PREFERRED|draw8|odg|application/vnd.oasis.opendocument.graphics
draw8_template|com.sun.star.drawing.DrawingDocument|ODF 図形描画テンプレート|IMPORT EXPORT|draw8_template|otg|application/vnd.oasis.opendocument.graphics-template
draw_PCD_Photo_CD_Base|com.sun.star.drawing.DrawingDocument|PCD - Kodak Photo CD (768x512)|IMPORT|pcd_Photo_CD_Base|pcd|image/x-photo-cd
draw_PCD_Photo_CD_Base16|com.sun.star.drawing.DrawingDocument|PCD - Kodak Photo CD (192x128)|IMPORT|pcd_Photo_CD_Base16|pcd|image/x-photo-cd
draw_PCD_Photo_CD_Base4|com.sun.star.drawing.DrawingDocument|PCD - Kodak Photo CD (384x256)|IMPORT|pcd_Photo_CD_Base4|pcd|image/x-photo-cd
draw_StarOffice_XML_Draw_Template|com.sun.star.drawing.DrawingDocument|OpenOffice.org 1.0 図形描画テンプレート|IMPORT|draw_StarOffice_XML_Draw_Template|std|application/vnd.sun.xml.draw.template
draw_bmp_Export|com.sun.star.drawing.DrawingDocument|BMP - Windows Bitmap|EXPORT|bmp_MS_Windows|bmp|image/x-MS-bmp
draw_emf_Export|com.sun.star.drawing.DrawingDocument|EMF - Enhanced Metafile|EXPORT|emf_MS_Windows_Metafile|emf|image/x-emf
draw_eps_Export|com.sun.star.drawing.DrawingDocument|EPS - Encapsulated PostScript|EXPORT|eps_Encapsulated_PostScript|eps|image/x-eps
draw_flash_Export|com.sun.star.drawing.DrawingDocument|Macromedia Flash (SWF)|EXPORT|graphic_SWF|swf|
draw_gif_Export|com.sun.star.drawing.DrawingDocument|GIF - Graphics Interchange Format|EXPORT|gif_Graphics_Interchange|gif|image/gif
draw_html_Export|com.sun.star.drawing.DrawingDocument|HTML ドキュメント (Draw)|EXPORT|graphic_HTML|html htm|text/html
draw_jpg_Export|com.sun.star.drawing.DrawingDocument|JPEG - Joint Photographic Experts Group|EXPORT|jpg_JPEG|jpg jpeg jfif jif jpe|image/jpeg
draw_pdf_Export|com.sun.star.drawing.DrawingDocument|PDF - Portable Document Format|EXPORT|pdf_Portable_Document_Format|pdf|application/pdf
draw_pdf_addstream_import|com.sun.star.drawing.DrawingDocument|PDF - Portable Document Format|IMPORT|pdf_Portable_Document_Format|pdf|application/pdf
draw_pdf_import|com.sun.star.drawing.DrawingDocument|PDF - Portable Document Format (Draw)|IMPORT PREFERRED|pdf_Portable_Document_Format|pdf|application/pdf
draw_png_Export|com.sun.star.drawing.DrawingDocument|PNG - Portable Network Graphic|EXPORT|png_Portable_Network_Graphic|png|image/png
draw_svg_Export|com.sun.star.drawing.DrawingDocument|SVG - Scalable Vector Graphics|EXPORT|svg_Scalable_Vector_Graphics|svg svgz|image/svg+xml
draw_tif_Export|com.sun.star.drawing.DrawingDocument|TIFF - Tagged Image File Format|EXPORT|tif_Tag_Image_File|tif tiff|image/tiff
draw_wmf_Export|com.sun.star.drawing.DrawingDocument|WMF - Windows Metafile|EXPORT|wmf_MS_Windows_Metafile|wmf|image/x-wmf
MathML XML (Math)|com.sun.star.formula.FormulaProperties|MathML 2.0|IMPORT EXPORT|math_MathML_XML_Math|mml|application/mathml+xml
MathType 3.x|com.sun.star.formula.FormulaProperties|MathType3.x|IMPORT EXPORT|math_MathType_3x|xxx|
StarOffice XML (Math)|com.sun.star.formula.FormulaProperties|OpenOffice.org 1.0 数式|IMPORT|math_StarOffice_XML_Math|sxm|application/vnd.sun.xml.math
math8|com.sun.star.formula.FormulaProperties|ODF 数式|IMPORT EXPORT DEFAULT|math8|odf|application/vnd.oasis.opendocument.formula
math_pdf_Export|com.sun.star.formula.FormulaProperties|PDF - Portable Document Format|EXPORT|pdf_Portable_Document_Format|pdf|application/pdf
Apple Keynote|com.sun.star.presentation.PresentationDocument|Apple Keynote|IMPORT PREFERRED|impress_AppleKeynote|key|application/x-iwork-keynote-sffkey
CGM - Computer Graphics Metafile|com.sun.star.presentation.PresentationDocument|CGM - Computer Graphics Metafile|IMPORT|impress_CGM_Computer_Graphics_Metafile|cgm|image/cgm
ClarisWorks_Impress|com.sun.star.presentation.PresentationDocument|ClarisWorks/AppleWorks プレゼンテーション|IMPORT PREFERRED|impress_ClarisWorks|cwk|application/clarisworks
Impress MS PowerPoint 2007 XML|com.sun.star.presentation.PresentationDocument|PowerPoint 2007–365|IMPORT EXPORT PREFERRED|MS PowerPoint 2007 XML|pptx|application/vnd.openxmlformats-officedocument.presentationml.presentation
Impress MS PowerPoint 2007 XML AutoPlay|com.sun.star.presentation.PresentationDocument|PowerPoint 2007–365 オートプレイ|IMPORT EXPORT PREFERRED|MS PowerPoint 2007 XML AutoPlay|ppsx|application/vnd.openxmlformats-officedocument.presentationml.slideshow
Impress MS PowerPoint 2007 XML Template|com.sun.star.presentation.PresentationDocument|PowerPoint 2007–365 テンプレート|IMPORT EXPORT PREFERRED|MS PowerPoint 2007 XML Template|potx potm|application/vnd.openxmlformats-officedocument.presentationml.template
Impress MS PowerPoint 2007 XML VBA|com.sun.star.presentation.PresentationDocument|PowerPoint 2007–365 VBA|IMPORT EXPORT PREFERRED|MS PowerPoint 2007 XML VBA|pptm|application/vnd.ms-powerpoint.presentation.macroEnabled.main+xml
Impress Office Open XML|com.sun.star.presentation.PresentationDocument|Office Open XML プレゼンテーション|IMPORT EXPORT PREFERRED|Office Open XML Presentation|pptx pptm|application/vnd.openxmlformats-officedocument.presentationml.presentation
Impress Office Open XML AutoPlay|com.sun.star.presentation.PresentationDocument|Office Open XML プレゼンテーションオートプレイ|IMPORT EXPORT PREFERRED|Office Open XML Presentation AutoPlay|ppsx|application/vnd.openxmlformats-officedocument.presentationml.slideshow
Impress Office Open XML Template|com.sun.star.presentation.PresentationDocument|Office Open XML プレゼンテーションテンプレート|IMPORT EXPORT PREFERRED|Office Open XML Presentation Template|potx potm|application/vnd.openxmlformats-officedocument.presentationml.template
MS PowerPoint 97|com.sun.star.presentation.PresentationDocument|PowerPoint 97–2003|IMPORT EXPORT|impress_MS_PowerPoint_97|ppt dps|application/vnd.ms-powerpoint
MS PowerPoint 97 AutoPlay|com.sun.star.presentation.PresentationDocument|PowerPoint 97–2003 オートプレイ|IMPORT EXPORT|impress_MS_PowerPoint_97_AutoPlay|pps|application/vnd.ms-powerpoint
MS PowerPoint 97 Vorlage|com.sun.star.presentation.PresentationDocument|PowerPoint 97–2003 テンプレート|IMPORT EXPORT|impress_MS_PowerPoint_97_Vorlage|pot dpt|application/vnd.ms-powerpoint
MWAW_Presentation|com.sun.star.presentation.PresentationDocument|古いMacのプレゼンテーション|IMPORT PREFERRED|MWAW_Presentation|*|
OpenDocument Presentation Flat XML|com.sun.star.presentation.PresentationDocument|Flat XML ODF プレゼンテーション|IMPORT EXPORT|impress_ODP_FlatXML|fodp odp xml|application/vnd.oasis.opendocument.presentation-flat-xml
PowerPoint 3|com.sun.star.presentation.PresentationDocument|Microsoft PowerPoint 1-4および95|IMPORT|impress_PowerPoint3|ppt pot|
SVG - Scalable Vector Graphics|com.sun.star.presentation.PresentationDocument|SVG - Scalable Vector Graphics|IMPORT PREFERRED|svg_Scalable_Vector_Graphics|svg svgz|image/svg+xml
StarOffice XML (Impress)|com.sun.star.presentation.PresentationDocument|OpenOffice.org 1.0 プレゼンテーション|IMPORT PREFERRED|impress_StarOffice_XML_Impress|sxi|application/vnd.sun.xml.impress
StarOffice_Presentation|com.sun.star.presentation.PresentationDocument|古いStarOfficeプレゼンテーション|IMPORT PREFERRED|StarOffice_Presentation|sdd|
UOF presentation|com.sun.star.presentation.PresentationDocument|Unified Office Format プレゼンテーション|IMPORT EXPORT|Unified_Office_Format_presentation|uop uof|
XHTML Impress File|com.sun.star.presentation.PresentationDocument|XHTML|EXPORT|XHTML_File|html xhtml|application/xhtml+xml
impress8|com.sun.star.presentation.PresentationDocument|ODF プレゼンテーション|IMPORT EXPORT DEFAULT PREFERRED|impress8|odp|application/vnd.oasis.opendocument.presentation
impress8_draw|com.sun.star.presentation.PresentationDocument|ODF 図形描画 (Impress)|IMPORT EXPORT|draw8|odg|application/vnd.oasis.opendocument.graphics
impress8_template|com.sun.star.presentation.PresentationDocument|ODF プレゼンテーションテンプレート|IMPORT EXPORT|impress8_template|otp|application/vnd.oasis.opendocument.presentation-template
impress_StarOffice_XML_Draw|com.sun.star.presentation.PresentationDocument|OpenOffice.org 1.0 図形描画 (Impress)|IMPORT|draw_StarOffice_XML_Draw|sxd|application/vnd.sun.xml.draw
impress_StarOffice_XML_Impress_Template|com.sun.star.presentation.PresentationDocument|OpenOffice.org 1.0 プレゼンテーションテンプレート|IMPORT|impress_StarOffice_XML_Impress_Template|sti|application/vnd.sun.xml.impress.template
impress_bmp_Export|com.sun.star.presentation.PresentationDocument|BMP - Windows Bitmap|EXPORT|bmp_MS_Windows|bmp|image/x-MS-bmp
impress_emf_Export|com.sun.star.presentation.PresentationDocument|EMF - Enhanced Metafile|EXPORT|emf_MS_Windows_Metafile|emf|image/x-emf
impress_eps_Export|com.sun.star.presentation.PresentationDocument|EPS - Encapsulated PostScript|EXPORT|eps_Encapsulated_PostScript|eps|image/x-eps
impress_flash_Export|com.sun.star.presentation.PresentationDocument|Macromedia Flash (SWF)|EXPORT|graphic_SWF|swf|
impress_gif_Export|com.sun.star.presentation.PresentationDocument|GIF - Graphics Interchange Format|EXPORT|gif_Graphics_Interchange|gif|image/gif
impress_html_Export|com.sun.star.presentation.PresentationDocument|HTML ドキュメント (Impress)|EXPORT|graphic_HTML|html htm|text/html
impress_jpg_Export|com.sun.star.presentation.PresentationDocument|JPEG - Joint Photographic Experts Group|EXPORT|jpg_JPEG|jpg jpeg jfif jif jpe|image/jpeg
impress_pdf_Export|com.sun.star.presentation.PresentationDocument|PDF - Portable Document Format|EXPORT|pdf_Portable_Document_Format|pdf|application/pdf
impress_pdf_addstream_import|com.sun.star.presentation.PresentationDocument|PDF - Portable Document Format|IMPORT|pdf_Portable_Document_Format|pdf|application/pdf
impress_pdf_import|com.sun.star.presentation.PresentationDocument|PDF - Portable Document Format (Impress)|IMPORT PREFERRED|pdf_Portable_Document_Format|pdf|application/pdf
impress_png_Export|com.sun.star.presentation.PresentationDocument|PNG - Portable Network Graphic|EXPORT|png_Portable_Network_Graphic|png|image/png
impress_svg_Export|com.sun.star.presentation.PresentationDocument|SVG - Scalable Vector Graphics|EXPORT|svg_Scalable_Vector_Graphics|svg svgz|image/svg+xml
impress_tif_Export|com.sun.star.presentation.PresentationDocument|TIFF - Tagged Image File Format|EXPORT|tif_Tag_Image_File|tif tiff|image/tiff
impress_wmf_Export|com.sun.star.presentation.PresentationDocument|WMF - Windows Metafile|EXPORT|wmf_MS_Windows_Metafile|wmf|image/x-wmf
StarOffice XML (Base) Report|com.sun.star.report.ReportDefinition|ODF データベースレポート|IMPORT EXPORT DEFAULT|StarBaseReport|orp|application/vnd.sun.xml.report
StarOffice XML (Base)|com.sun.star.sdb.OfficeDatabaseDocument|ODF データベース|IMPORT DEFAULT|StarBase|odb|application/vnd.sun.xml.base
ADO Rowset XML|com.sun.star.sheet.SpreadsheetDocument|ADO Rowset XML|IMPORT|calc_ADO_rowset_XML|xml|
Apple Numbers|com.sun.star.sheet.SpreadsheetDocument|Apple Numbers|IMPORT PREFERRED|calc_AppleNumbers|numbers|application/x-iwork-numbers-sffnumbers
Calc MS Excel 2007 Binary|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 2007 バイナリ|IMPORT PREFERRED|MS Excel 2007 Binary|xlsb|
Calc MS Excel 2007 VBA XML|com.sun.star.sheet.SpreadsheetDocument|Excel 2007–365 (マクロ有効)|IMPORT EXPORT PREFERRED|MS Excel 2007 VBA XML|xlsm|application/vnd.ms-excel.sheet.macroEnabled.12
Calc MS Excel 2007 XML|com.sun.star.sheet.SpreadsheetDocument|Excel 2007–365|IMPORT EXPORT PREFERRED|MS Excel 2007 XML|xlsx|application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Calc MS Excel 2007 XML Template|com.sun.star.sheet.SpreadsheetDocument|Excel 2007–365 テンプレート|IMPORT EXPORT|MS Excel 2007 XML Template|xltx xltm|application/vnd.openxmlformats-officedocument.spreadsheetml.template
Calc Office Open XML|com.sun.star.sheet.SpreadsheetDocument|Office Open XML 表計算ドキュメント|IMPORT EXPORT PREFERRED|Office Open XML Spreadsheet|xlsx xlsm|application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Calc Office Open XML Template|com.sun.star.sheet.SpreadsheetDocument|Office Open XML 表計算ドキュメントテンプレート|IMPORT|Office Open XML Spreadsheet Template|xltx xltm|application/vnd.openxmlformats-officedocument.spreadsheetml.template
ClarisWorks_Calc|com.sun.star.sheet.SpreadsheetDocument|ClarisWorks/AppleWorks 表計算ドキュメント|IMPORT PREFERRED|calc_ClarisWorks|cwk|application/clarisworks
Claris_Resolve_Calc|com.sun.star.sheet.SpreadsheetDocument|ClarisResolve ドキュメント|IMPORT PREFERRED|calc_Claris_Resolve|cwk|application/clarisworks
DIF|com.sun.star.sheet.SpreadsheetDocument|Data Interchange Format|IMPORT EXPORT|calc_DIF|dif|
Gnumeric Spreadsheet|com.sun.star.sheet.SpreadsheetDocument|Gnumeric 表計算ドキュメント|IMPORT PREFERRED|Gnumeric XML|gnumeric gnm|application/x-gnumeric
HTML (StarCalc)|com.sun.star.sheet.SpreadsheetDocument|HTML ドキュメント (Calc)|IMPORT EXPORT|generic_HTML|html xhtml htm|text/html
Lotus|com.sun.star.sheet.SpreadsheetDocument|Lotus 1-2-3|IMPORT PREFERRED|calc_Lotus|wk1 wks 123|application/vnd.lotus-1-2-3
MS Excel 2003 XML|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 2003 XML|EXPORT|calc_MS_Excel_2003_XML|xml xls|
MS Excel 2003 XML Orcus|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 2003 XML|IMPORT PREFERRED|calc_MS_Excel_2003_XML|xml xls|
MS Excel 4.0|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 4.0|IMPORT PREFERRED|calc_MS_Excel_40|xls xlw xlc xlm|application/vnd.ms-excel
MS Excel 4.0 Vorlage/Template|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 4.0 テンプレート|IMPORT|calc_MS_Excel_40_VorlageTemplate|xlt|application/vnd.ms-excel
MS Excel 5.0/95|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 5.0|IMPORT PREFERRED|calc_MS_Excel_5095|xls xlc xlm xlw|application/vnd.ms-excel
MS Excel 5.0/95 Vorlage/Template|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 5.0 テンプレート|IMPORT|calc_MS_Excel_5095_VorlageTemplate|xlt|application/vnd.ms-excel
MS Excel 95|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 95|IMPORT PREFERRED|calc_MS_Excel_95|xls xlc xlm xlw|application/vnd.ms-excel
MS Excel 95 Vorlage/Template|com.sun.star.sheet.SpreadsheetDocument|Microsoft Excel 95 テンプレート|IMPORT|calc_MS_Excel_95_VorlageTemplate|xlt|application/vnd.ms-excel
MS Excel 97|com.sun.star.sheet.SpreadsheetDocument|Excel 97–2003|IMPORT EXPORT PREFERRED|calc_MS_Excel_97|xls xlc xlm xlw xlk et|application/vnd.ms-excel
MS Excel 97 Vorlage/Template|com.sun.star.sheet.SpreadsheetDocument|Excel 97–2003 テンプレート|IMPORT EXPORT|calc_MS_Excel_97_VorlageTemplate|xlt ett|application/vnd.ms-excel
MS_Works_Calc|com.sun.star.sheet.SpreadsheetDocument|Microsoft Works ドキュメント|IMPORT|calc_MS_Works_Document|wks wdb|
MWAW_Database|com.sun.star.sheet.SpreadsheetDocument|古いMacのデータベース|IMPORT PREFERRED|MWAW_Database|*|
MWAW_Spreadsheet|com.sun.star.sheet.SpreadsheetDocument|古いMacの表計算|IMPORT PREFERRED|MWAW_Spreadsheet|*|
Mac_Works_Calc|com.sun.star.sheet.SpreadsheetDocument|Microsoft Works for Mac 表計算ドキュメント (v1 - v4)|IMPORT PREFERRED|calc_Mac_Works|wps|application/vnd.ms-works
Microsoft Multiplan|com.sun.star.sheet.SpreadsheetDocument|Microsoft Multiplan|IMPORT PREFERRED|calc_MS_Multiplan|mp|
OpenDocument Spreadsheet Flat XML|com.sun.star.sheet.SpreadsheetDocument|Flat XML ODF 表計算ドキュメント|IMPORT EXPORT|calc_ODS_FlatXML|fods ods xml|application/vnd.oasis.opendocument.spreadsheet-flat-xml
Quattro Pro 6.0|com.sun.star.sheet.SpreadsheetDocument|Quattro Pro 6.0|IMPORT PREFERRED|calc_QPro|wb2|
Rich Text Format (StarCalc)|com.sun.star.sheet.SpreadsheetDocument|Rich Text Format (Calc)|IMPORT|writer_Rich_Text_Format|rtf|application/rtf
SYLK|com.sun.star.sheet.SpreadsheetDocument|SYLK|IMPORT EXPORT|calc_SYLK|slk sylk|text/spreadsheet
StarOffice XML (Calc)|com.sun.star.sheet.SpreadsheetDocument|OpenOffice.org 1.0 表計算ドキュメント|IMPORT|calc_StarOffice_XML_Calc|sxc|application/vnd.sun.xml.calc
StarOffice_Spreadsheet|com.sun.star.sheet.SpreadsheetDocument|古い StarOffice 表計算ドキュメント|IMPORT PREFERRED|StarOffice_Spreadsheet|sdc|
Text - txt - csv (StarCalc)|com.sun.star.sheet.SpreadsheetDocument|テキスト CSV|IMPORT EXPORT|generic_Text|csv tsv tab txt|text/plain
UOF spreadsheet|com.sun.star.sheet.SpreadsheetDocument|Unified Office Format 表計算ドキュメント|IMPORT EXPORT|Unified_Office_Format_spreadsheet|uos uof|
WPS_Lotus_Calc|com.sun.star.sheet.SpreadsheetDocument|Lotus ドキュメント|IMPORT|calc_WPS_Lotus_Document|wk1 wk3 wk4 123|
WPS_QPro_Calc|com.sun.star.sheet.SpreadsheetDocument|QuattroPro ドキュメント|IMPORT|calc_WPS_QPro_Document|wb1 wb2 wq1 wq2|
XHTML Calc File|com.sun.star.sheet.SpreadsheetDocument|XHTML|EXPORT|XHTML_File|html xhtml|application/xhtml+xml
calc8|com.sun.star.sheet.SpreadsheetDocument|ODF 表計算ドキュメント|IMPORT EXPORT DEFAULT|calc8|ods|application/vnd.oasis.opendocument.spreadsheet
calc8_template|com.sun.star.sheet.SpreadsheetDocument|ODF 表計算ドキュメントテンプレート|IMPORT EXPORT|calc8_template|ots|application/vnd.oasis.opendocument.spreadsheet-template
calc_HTML_WebQuery|com.sun.star.sheet.SpreadsheetDocument|Webページ クエリー (Calc)|IMPORT PREFERRED|generic_HTML|html xhtml htm|text/html
calc_StarOffice_XML_Calc_Template|com.sun.star.sheet.SpreadsheetDocument|OpenOffice.org 1.0 表計算ドキュメントテンプレート|IMPORT|calc_StarOffice_XML_Calc_Template|stc|application/vnd.sun.xml.calc.template
calc_jpg_Export|com.sun.star.sheet.SpreadsheetDocument|JPEG - Joint Photographic Experts Group|EXPORT|jpg_JPEG|jpg jpeg jfif jif jpe|image/jpeg
calc_pdf_Export|com.sun.star.sheet.SpreadsheetDocument|PDF - Portable Document Format|EXPORT|pdf_Portable_Document_Format|pdf|application/pdf
calc_pdf_addstream_import|com.sun.star.sheet.SpreadsheetDocument|PDF - Portable Document Format|IMPORT|pdf_Portable_Document_Format|pdf|application/pdf
calc_png_Export|com.sun.star.sheet.SpreadsheetDocument|PNG - Portable Network Graphic|EXPORT|png_Portable_Network_Graphic|png|image/png
calc_svg_Export|com.sun.star.sheet.SpreadsheetDocument|SVG - Scalable Vector Graphics|EXPORT|svg_Scalable_Vector_Graphics|svg svgz|image/svg+xml
dBase|com.sun.star.sheet.SpreadsheetDocument|dBASE|IMPORT EXPORT|calc_dBase|dbf|
Text (encoded) (StarWriter/GlobalDocument)|com.sun.star.text.GlobalDocument|文書 - エンコードの選択 (マスタードキュメント)|IMPORT EXPORT|generic_Text|csv tsv tab txt|text/plain
writer_globaldocument_StarOffice_XML_Writer|com.sun.star.text.GlobalDocument|OpenOffice.org 1.0 文書ドキュメント||writer_StarOffice_XML_Writer|sxw|application/vnd.sun.xml.writer
writer_globaldocument_StarOffice_XML_Writer_GlobalDocument|com.sun.star.text.GlobalDocument|OpenOffice.org 1.0 マスタードキュメント|IMPORT PREFERRED|writer_globaldocument_StarOffice_XML_Writer_GlobalDocument|sxg|application/vnd.sun.xml.writer.global
writer_globaldocument_pdf_Export|com.sun.star.text.GlobalDocument|PDF - Portable Document Format|EXPORT|pdf_Portable_Document_Format|pdf|application/pdf
writerglobal8|com.sun.star.text.GlobalDocument|ODF マスタードキュメント|IMPORT EXPORT PREFERRED|writerglobal8|odm|application/vnd.oasis.opendocument.text-master
writerglobal8_HTML|com.sun.star.text.GlobalDocument|HTML (Writer/Global)|EXPORT|generic_HTML|html xhtml htm|text/html
writerglobal8_template|com.sun.star.text.GlobalDocument|ODF マスタードキュメントテンプレート|IMPORT EXPORT|writerglobal8_template|otm|application/vnd.oasis.opendocument.text-master-template
writerglobal8_writer|com.sun.star.text.GlobalDocument|ODF 文書ドキュメント|EXPORT DEFAULT|writer8|odt|application/vnd.oasis.opendocument.text
AbiWord|com.sun.star.text.TextDocument|AbiWord ドキュメント|IMPORT|writer_AbiWord_Document|abw zabw|application/x-abiword
Apple Pages|com.sun.star.text.TextDocument|Apple Pages|IMPORT PREFERRED|writer_ApplePages|pages|application/x-iwork-pages-sffpages
BroadBand eBook|com.sun.star.text.TextDocument|BroadBand eBook|IMPORT PREFERRED|writer_BroadBand_eBook|lrf|application/x-sony-bbeb
ClarisWorks|com.sun.star.text.TextDocument|ClarisWorks/AppleWorks 文書ドキュメント|IMPORT PREFERRED|writer_ClarisWorks|cwk|application/clarisworks
DocBook File|com.sun.star.text.TextDocument|DocBook|IMPORT EXPORT|writer_DocBook_File|xml|application/docbook+xml
DosWord|com.sun.star.text.TextDocument|Microsoft Word for DOS|IMPORT|writer_DosWord|doc|
EPUB|com.sun.star.text.TextDocument|EPUB文書|EXPORT|writer_EPUB_Document|epub|application/epub+zip
FictionBook 2|com.sun.star.text.TextDocument|FictionBook 2.0|IMPORT PREFERRED|writer_FictionBook_2|fb2 zip|application/x-fictionbook+xml
HTML (StarWriter)|com.sun.star.text.TextDocument|HTML ドキュメント (Writer)|IMPORT EXPORT|generic_HTML|html xhtml htm|text/html
LotusWordPro|com.sun.star.text.TextDocument|Lotus WordPro ドキュメント|IMPORT PREFERRED|writer_LotusWordPro_Document|lwp|application/vnd.lotus-wordpro
MS WinWord 5|com.sun.star.text.TextDocument|Microsoft WinWord 1/2/5|IMPORT|writer_MS_WinWord_5|doc|application/msword
MS WinWord 6.0|com.sun.star.text.TextDocument|Microsoft Word 6.0|IMPORT|writer_MS_WinWord_60|doc|application/msword
MS Word 2003 XML|com.sun.star.text.TextDocument|Word 2003 XML|IMPORT EXPORT|writer_MS_Word_2003_XML|xml doc|
MS Word 2007 XML|com.sun.star.text.TextDocument|Word 2007–365|IMPORT EXPORT|writer_MS_Word_2007|docx|application/msword
MS Word 2007 XML Template|com.sun.star.text.TextDocument|Word 2007–365 テンプレート|IMPORT EXPORT|writer_MS_Word_2007_Template|dotx dotm|application/msword
MS Word 2007 XML VBA|com.sun.star.text.TextDocument|Word 2007–365 VBA|IMPORT EXPORT|writer_MS_Word_2007_VBA|docm|application/msword
MS Word 95|com.sun.star.text.TextDocument|Microsoft Word 95|IMPORT|writer_MS_Word_95|doc|application/msword
MS Word 95 Vorlage|com.sun.star.text.TextDocument|Microsoft Word 95 テンプレート|IMPORT|writer_MS_Word_95_Vorlage|dot|application/msword
MS Word 97|com.sun.star.text.TextDocument|Word 97–2003|IMPORT EXPORT PREFERRED|writer_MS_Word_97|doc wps|application/msword
MS Word 97 Vorlage|com.sun.star.text.TextDocument|Word 97–2003 テンプレート|IMPORT EXPORT|writer_MS_Word_97_Vorlage|dot wpt|application/msword
MS_Works|com.sun.star.text.TextDocument|Microsoft Works ドキュメント|IMPORT|writer_MS_Works_Document|wps|application/vnd.ms-works
MS_Write|com.sun.star.text.TextDocument|Microsoft Write|IMPORT|writer_MS_Write|wri|application/x-mswrite
MWAW_Text_Document|com.sun.star.text.TextDocument|古いMacの文書ドキュメント|IMPORT PREFERRED|MWAW_Text_Document|*|
MacWrite|com.sun.star.text.TextDocument|MacWrite ドキュメント|IMPORT PREFERRED|writer_MacWrite|mw mcw|application/macwriteii
Mac_Word|com.sun.star.text.TextDocument|Microsoft Word for Mac (v1 - v5)|IMPORT PREFERRED|writer_Mac_Word|doc|application/msword
Mac_Works|com.sun.star.text.TextDocument|Microsoft Works for Mac 文書ドキュメント (v1 - v4)|IMPORT PREFERRED|writer_Mac_Works|wps|application/vnd.ms-works
Mariner_Write|com.sun.star.text.TextDocument|Mariner Write Mac Classic v1.6 - v3.5|IMPORT PREFERRED|writer_Mariner_Write|mwd|
Office Open XML Text|com.sun.star.text.TextDocument|Office Open XML 文書|IMPORT EXPORT|writer_OOXML|docx docm|application/vnd.openxmlformats-officedocument.wordprocessingml.document
Office Open XML Text Template|com.sun.star.text.TextDocument|Office Open XML 文書テンプレート|IMPORT|writer_OOXML_Text_Template|dotx dotm|application/vnd.openxmlformats-officedocument.wordprocessingml.template
OpenDocument Text Flat XML|com.sun.star.text.TextDocument|Flat XML ODF 文書ドキュメント|IMPORT EXPORT|writer_ODT_FlatXML|fodt odt xml|application/vnd.oasis.opendocument.text-flat-xml
PalmDoc|com.sun.star.text.TextDocument|PalmDoc eBook|IMPORT PREFERRED|writer_PalmDoc|pdb|application/x-aportisdoc
Palm_Text_Document|com.sun.star.text.TextDocument|Palm 文書ドキュメント|IMPORT PREFERRED|Palm_Text_Document|pdb|application/vnd.palm
Plucker eBook|com.sun.star.text.TextDocument|Plucker eBook|IMPORT PREFERRED|writer_Plucker_eBook|pdb|application/prs.plucker
Rich Text Format|com.sun.star.text.TextDocument|Rich Text|IMPORT EXPORT PREFERRED|writer_Rich_Text_Format|rtf|application/rtf
StarOffice XML (Writer)|com.sun.star.text.TextDocument|OpenOffice.org 1.0 文書ドキュメント|IMPORT PREFERRED|writer_StarOffice_XML_Writer|sxw|application/vnd.sun.xml.writer
StarOffice_Writer|com.sun.star.text.TextDocument|古い StarOffice 文書ドキュメント|IMPORT PREFERRED|StarOffice_Writer|sdw|
T602Document|com.sun.star.text.TextDocument|T602 ドキュメント|IMPORT PREFERRED|writer_T602_Document|602 txt|application/x-t602
Text|com.sun.star.text.TextDocument|テキスト|IMPORT EXPORT PREFERRED|generic_Text|csv tsv tab txt|text/plain
Text (encoded)|com.sun.star.text.TextDocument|文書 - エンコードの選択|IMPORT EXPORT|generic_Text|csv tsv tab txt|text/plain
UOF text|com.sun.star.text.TextDocument|Unified Office Format テキスト|IMPORT EXPORT|Unified_Office_Format_text|uot uof|
WordPerfect|com.sun.star.text.TextDocument|WordPerfect ドキュメント|IMPORT PREFERRED|writer_WordPerfect_Document|wpd|application/vnd.wordperfect
WriteNow|com.sun.star.text.TextDocument|WriteNow ドキュメント|IMPORT PREFERRED|writer_WriteNow|wn nx^d|
XHTML Writer File|com.sun.star.text.TextDocument|XHTML|EXPORT|XHTML_File|html xhtml|application/xhtml+xml
writer8|com.sun.star.text.TextDocument|ODF 文書ドキュメント|IMPORT EXPORT DEFAULT PREFERRED|writer8|odt|application/vnd.oasis.opendocument.text
writer8_template|com.sun.star.text.TextDocument|ODF 文書ドキュメントテンプレート|IMPORT EXPORT|writer8_template|ott|application/vnd.oasis.opendocument.text-template
writer_MIZI_Hwp_97|com.sun.star.text.TextDocument|Hangul WP 97|IMPORT|writer_MIZI_Hwp_97|hwp|application/x-hwp
writer_StarOffice_XML_Writer_Template|com.sun.star.text.TextDocument|OpenOffice.org 1.0 文書ドキュメントテンプレート|IMPORT|writer_StarOffice_XML_Writer_Template|stw|application/vnd.sun.xml.writer.template
writer_jpg_Export|com.sun.star.text.TextDocument|JPEG - Joint Photographic Experts Group|EXPORT|jpg_JPEG|jpg jpeg jfif jif jpe|image/jpeg
writer_layout_dump|com.sun.star.text.TextDocument|Writer Layout XML|EXPORT|writer_layout_dump_xml|xml|
writer_pdf_Export|com.sun.star.text.TextDocument|PDF - Portable Document Format|EXPORT|pdf_Portable_Document_Format|pdf|application/pdf
writer_pdf_addstream_import|com.sun.star.text.TextDocument|PDF - Portable Document Format|IMPORT|pdf_Portable_Document_Format|pdf|application/pdf
writer_pdf_import|com.sun.star.text.TextDocument|PDF - Portable Document Format (Writer)|IMPORT PREFERRED|pdf_Portable_Document_Format|pdf|application/pdf
writer_png_Export|com.sun.star.text.TextDocument|PNG - Portable Network Graphic|EXPORT|png_Portable_Network_Graphic|png|image/png
writer_svg_Export|com.sun.star.text.TextDocument|SVG - Scalable Vector Graphics|EXPORT|svg_Scalable_Vector_Graphics|svg svgz|image/svg+xml
HTML|com.sun.star.text.WebDocument|HTML ドキュメント|IMPORT EXPORT PREFERRED|generic_HTML|html xhtml htm|text/html
Text (StarWriter/Web)|com.sun.star.text.WebDocument|文書 (Writer/Web)|IMPORT EXPORT|generic_Text|csv tsv tab txt|text/plain
Text (encoded) (StarWriter/Web)|com.sun.star.text.WebDocument|文書 - エンコードの選択 (Writer/Web)|IMPORT EXPORT|generic_Text|csv tsv tab txt|text/plain
writer_web_HTML_help|com.sun.star.text.WebDocument|Help コンテンツ|IMPORT|writer_web_HTML_help||
writer_web_StarOffice_XML_Writer|com.sun.star.text.WebDocument|OpenOffice.org 1.0 文書ドキュメント (Writer/Web)|EXPORT|writer_StarOffice_XML_Writer|sxw|application/vnd.sun.xml.writer
writer_web_StarOffice_XML_Writer_Web_Template|com.sun.star.text.WebDocument|OpenOffice.org 1.0 HTML テンプレート|IMPORT|writer_web_StarOffice_XML_Writer_Web_Template|stw|application/vnd.sun.xml.writer.web
writer_web_jpg_Export|com.sun.star.text.WebDocument|JPEG - Joint Photographic Experts Group|EXPORT|jpg_JPEG|jpg jpeg jfif jif jpe|image/jpeg
writer_web_pdf_Export|com.sun.star.text.WebDocument|PDF - Portable Document Format|EXPORT|pdf_Portable_Document_Format|pdf|application/pdf
writer_web_png_Export|com.sun.star.text.WebDocument|PNG - Portable Network Graphic|EXPORT|png_Portable_Network_Graphic|png|image/png
writerweb8_writer|com.sun.star.text.WebDocument|文書 (Writer/Web)|EXPORT|writer8|odt|application/vnd.oasis.opendocument.text
writerweb8_writer_template|com.sun.star.text.WebDocument|HTML ドキュメントテンプレート|IMPORT EXPORT|writerweb8_writer_template|oth|application/vnd.oasis.opendocument.text-web

この表を出力するのには以下のマクロを使用した。
このマクロは LibreOffice Calc のスプレッドシートを作成するので CSV に出力して MarkDown に編集した。

```py:filters.py
"""list import/expot filters from FilterFactory and TypeDetection services
"""

"""
  (C) Copyright 2020 Shojiro Fushimi, all rights reserved

  Redistribution and use in source and binary forms, with or
  without modification, are permitted provided that the following
  conditions are met:

   - Redistributions of source code must retain the above
    copyright notice, this list of conditions and the following
    disclaimer.

   - Redistributions in binary form must reproduce the above
    copyright notice, this list of conditions and the following
    disclaimer in the documentation and/or other materials
    provided with the distribution.

  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDER ``AS IS''
  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
  LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
  FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT
  SHALL THE AUTHOR OR CONTRIBUTORS BE LIABLE FOR ANY
  DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA,
  OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
  THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR
  TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT
  OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY
  OF SUCH DAMAGE.
"""

from functools import reduce

import uno
from com.sun.star.awt.FontWeight import BOLD

ALL_FIELDS = False
FIELDS = ['DocumentService', 'UIName', 'Flags']
ALL_FLAGS = False
FLAGS = ['IMPORT', 'EXPORT', 'DEFAULT', 'PREFERRED']
KEY_FIELD = 'DocumentService'

SHOW_TYPE_FIELDS = True
ALL_TYPE_FIELDS = False
TYPE_FIELDS = ['Extensions', 'MediaType']

# Flags are defined in <source directory>/include/comphelper/documentconstants.hxx
# with exception of3RDPARTYFILTER is STARONEFILTER and READONLY is OPENREADONLY there
FLAG_LABELS = [(0x00000001, 'IMPORT'),
               (0x00000002, 'EXPORT'),
               (0x00000004, 'TEMPLATE'),
               (0x00000008, 'INTERNAL'),
               (0x00000010, 'TEMPLATEPATH'),
               (0x00000020, 'OWN'),
               (0x00000040, 'ALIEN'),
               (0x00000100, 'DEFAULT'),
               (0x00000400, 'SUPPORTSSELECTION'),
               (0x00001000, 'NOTINFILEDIALOG'),
               (0x00010000, 'READONLY'),
               (0x00020000, 'MUSTINSTALL'),
               (0x00040000, 'CONSULTSERVICE'),
               (0x00080000, '3RDPARTYFILTER'),
               (0x00100000, 'PACKED'),
               (0x00200000, 'EXOTIC'),
               (0x00800000, 'COMBINED'),
               (0x01000000, 'ENCRYPTION'),
               (0x02000000, 'PASSWORDTOMODIFY'),
               (0x04000000, 'GPGENCRYPTION'),
               (0x10000000, 'PREFERRED'),
               (0x20000000, 'STARTPRESENTATION'),
               (0x40000000, 'SUPPORTSSIGNING'),
]

def main(*args, **kwds):
    ctx = XSCRIPTCONTEXT.getComponentContext()
    smgr = ctx.ServiceManager
    # get filter information from filter factory
    SERVICE_FILTERFACTORY = 'com.sun.star.document.FilterFactory'
    SERVICE_TYPEDETECTION = 'com.sun.star.document.TypeDetection'
    filter_factory = smgr.createInstanceWithContext(SERVICE_FILTERFACTORY, ctx)
    filter_dict = {}
    for filter_name in filter_factory.getElementNames():
        try:
            property_dict = {}
            for property in filter_factory.getByName(filter_name):
                property_dict[property.Name]  = property.Value
            filter_dict[filter_name] = property_dict
        except:
            continue
    fields = sorted(list(reduce(lambda x,y:x|y,
                                (set(v.keys()) for v in filter_dict.values()),
                                set())))
    if not ALL_FIELDS:
        fields = [f for f in FIELDS if f in fields]
        if SHOW_TYPE_FIELDS and 'Type' not in fields:
            fields.append('Type')
    #
    if SHOW_TYPE_FIELDS:
        type_detection = smgr.createInstanceWithContext(SERVICE_TYPEDETECTION, ctx)
        type_dict = {}
        for type_name in type_detection.getElementNames():
            try:
                property_dict = {}
                for property in type_detection.getByName(type_name):
                    property_dict[property.Name]  = property.Value
                type_dict[type_name] = property_dict
            except:
                continue
        type_fields = sorted(list(reduce(lambda x,y:x|y,
                                         (set(v.keys()) for v in type_dict.values()),
                                         set())))
        if not ALL_TYPE_FIELDS:
            type_fields = [f for f in TYPE_FIELDS if f in type_fields]
    else:
        type_dict = {}
        type_fields = []
    #
    head = ['Name'] + fields + type_fields
    filter_table = []
    for filter_name, property_dict in filter_dict.items():
        filter_values = [filter_name] + [property_dict.get(f) for f in fields]
        if 'Flags' in fields:
            idx = head.index('Flags')
            if filter_values[idx] is not None:
                flag_value = filter_values[idx]
                flag_names = []
                for n, name in FLAG_LABELS:
                    if flag_value & n == n:
                        if ALL_FLAGS or name in FLAGS:
                            flag_names.append(name)
                        flag_value &= ~n
                if flag_value != 0: # found an unknown flag
                    flag_names.append('%8.8x' % flag_value)
                filter_values[idx] = ' '.join(flag_names)
        type_values = [type_dict.get(property_dict.get('Type'), {}).get(n) for n in type_fields]
        if 'Extensions' in type_fields:
            idx = type_fields.index('Extensions')
            if type_values[idx] is not None:
                type_values[idx] = ' '.join(type_values[idx])
        filter_table.append(filter_values + type_values)
    #
    if KEY_FIELD in fields:
        idx = head.index(KEY_FIELD)
        key = lambda r:(r[idx], r)
    else:
        key = None
    filter_table.sort(key=key)
    filter_table.insert(0, head)
    # table to spreadsheet
    NEWSHEET_URL = 'private:factory/scalc'
    desktop = XSCRIPTCONTEXT.getDesktop()
    spreadsheet = desktop.loadComponentFromURL(NEWSHEET_URL, '_blank', 0, ())
    sheet = spreadsheet.getSheets().getByIndex(0)
    for i, row in enumerate(filter_table):
        for j, col in enumerate(row):
            cell = sheet.getCellByPosition(j, i)
            cell.String = str(col)
    # set column width optimal
    columns = sheet.getColumns()
    for i in range(len(head)):
        columns.getByIndex(i).OptimalWidth = True
    # make head line characters bold
    for i in range(len(head)):
        cell = sheet.getCellByPosition(i, 0)
        cell.CharWeight = BOLD
    # freeze head row (column 0, row 1)
    spreadsheet.CurrentController.freezeAtPosition(0, 1)
    # set auto filter
    cell_range = sheet.getCellRangeByPosition(0, 0, len(head) - 1, len(filter_table) - 1)
    range_name = 'filter range'
    spreadsheet.DatabaseRanges.addNewByName(range_name, cell_range.getRangeAddress())
    filter_range = spreadsheet.DatabaseRanges.getByName(range_name)
    filter_range.AutoFilter = True
```

## LibreOffice インストールディレクトリからの取得

FilterFactory, TypeDetection サービスから取得できるのと同内容は、\<LibreOfficeインストールディレクトリ\>/share/registry ディレクトリ下の .xcd ファイルに定義がある。
こちらに定義されているのは LibreOffice が元から持っているフィルタのみで、拡張機能のフィルタの定義は拡張機能のインストールディレクトリ下の .xcu ファイルにある。

下記は .xcd ファイルからフィルタの表を作成するプログラムであるが、UIName の地域化（日本語名）には対応していない。地域化データは \<LibreOfficeインストールディレクトリ\>/share/registry/res ディレクトリ下にある。

```py:filter_registry.py
#!/usr/bin/env python

"""list import/expot filters from registry
"""

"""
  (C) Copyright 2020 Shojiro Fushimi, all rights reserved

  Redistribution and use in source and binary forms, with or
  without modification, are permitted provided that the following
  conditions are met:

   - Redistributions of source code must retain the above
    copyright notice, this list of conditions and the following
    disclaimer.

   - Redistributions in binary form must reproduce the above
    copyright notice, this list of conditions and the following
    disclaimer in the documentation and/or other materials
    provided with the distribution.

  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDER ``AS IS''
  AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
  LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
  FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT
  SHALL THE AUTHOR OR CONTRIBUTORS BE LIABLE FOR ANY
  DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
  CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA,
  OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
  THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR
  TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT
  OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY
  OF SUCH DAMAGE.
"""

import sys
from argparse import ArgumentParser
from re import compile as re_compile
from functools import reduce
import csv
import xml.etree.ElementTree

DEFAULT_FIELDS = ['DocumentService', 'UIName', 'Flags']
DEFAULT_FLAGS = ['IMPORT', 'EXPORT', 'DEFAULT', 'PREFERRED']
DEFAULT_KEY_FIELD = 'DocumentService'

DEFAULT_TYPE_FIELDS = ['Extensions', 'MediaType']

NS_RE = re_compile(r'\{(.*?)\}')

def component_data_dict(file_name: str, component_name: str, node_name: str):
    tree = xml.etree.ElementTree.parse(file_name)
    root = tree.getroot()
    namespace_mo = NS_RE.match(root.tag)
    if namespace_mo is None:
        return {}
    namespace = {'oor': namespace_mo.group(1)}
    data_dict = {}
    for component_data in root.findall(f'.//oor:component-data[@oor:name="{component_name}"]', namespace):
        component_node = component_data.find(f'node[@oor:name="{node_name}"]', namespace)
        if component_node is None:
            continue
        oor_name = f'{{{namespace["oor"]}}}name'
        for data_node in component_node.findall('node'):
            name = data_node.attrib.get(oor_name)
            if name is None:
                continue
            prop_dict = {}
            for prop in data_node.findall('prop'):
                prop_name = prop.attrib.get(oor_name)
                if prop_name is None:
                    continue
                value = prop.find('value')
                if value is not None:
                    value_text = value.text
                else:
                    value_text = None
                prop_dict[prop_name] = value_text
            data_dict[name] = prop_dict
    return data_dict

def main(argv):
    aparser = ArgumentParser()
    aparser.add_argument('files', nargs='*')
    aparser.add_argument('--out', '-o', dest='out', action='store', default=None)
    aparser.add_argument('--fields', dest='fields', action='store', default=','.join(DEFAULT_FIELDS))
    aparser.add_argument('--flags', dest='flags', action='store', default=','.join(DEFAULT_FLAGS))
    aparser.add_argument('--all-fields', dest='all_fields', action='store_true', default=False)
    aparser.add_argument('--all-flags', dest='all_flags', action='store_true', default=False)
    aparser.add_argument('--key-field', dest='key_field', action='store', default=DEFAULT_KEY_FIELD)
    aparser.add_argument('--show-type-fields', dest='show_type_fields', action='store_true', default=False)
    aparser.add_argument('--type-fields', dest='type_fields', action='store', default=','.join(DEFAULT_TYPE_FIELDS))
    aparser.add_argument('--all-type-fields', dest='all_type_fields', action='store_true', default=False)
    args = aparser.parse_args(argv[1:])
    filter_dict = reduce(lambda x,y:dict(x, **y),
                         (component_data_dict(file_name, 'Filter', 'Filters') for file_name in args.files),
                         {})
    fields = sorted(list(reduce(lambda x,y:x|y,
                                (set(v.keys()) for v in filter_dict.values()),
                                set())))
    if not args.all_fields:
        fields = [f.strip() for f in args.fields.split(',') if f.strip() in fields]
        if args.show_type_fields and 'Type' not in fields:
            fields.append('Type')
    #
    if 'Flags' in fields and not args.all_flags:
        for d in filter_dict.values():
            if 'Flags' in d:
                d['Flags'] = ' '.join([f for f in d['Flags'].split() if f in args.flags])
    #
    if args.show_type_fields:
        type_dict = reduce(lambda x,y:dict(x, **y),
                           (component_data_dict(file_name, 'Types', 'Types') for file_name in args.files),
                           {})
        type_fields = sorted(list(reduce(lambda x,y:x|y,
                                         (set(v.keys()) for v in type_dict.values()),
                                         set())))
        if not args.all_type_fields:
            type_fields = [f.strip() for f in args.type_fields.split(',') if f.strip() in type_fields]
    else:
        type_dict = {}
        type_fields = []
    #
    head = ['Name'] + fields + type_fields
    try:
        keyidx = head.index(args.key_field)
        key = lambda r:(r[keyidx], r)
    except:
        key = None
    table = sorted([[k] + [v.get(f) for f in fields] + [type_dict.get(v.get('Type'), {}).get(n) for n in type_fields] for k, v in filter_dict.items()],
                   key=key)
    table.insert(0, head)
    #
    if args.out is not None:
        outfp = open(args.out, 'w', newline='')
    else:
        outfp = sys.stdout
    csv.writer(outfp).writerows(table)
    if args.out is not None:
        outfp.close()
    return 0

if __name__ == '__main__':
    rc = main(sys.argv)
    sys.exit(rc)
```
