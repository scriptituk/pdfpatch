# pdfpatch

A simple patching tool to modify PDF pages using Ghostscript.

This executes a patch script to overlay opaque masks and text.

It works by executing a custom PostScript `/EndPage` procedure when `showpage` is invoked, at the end of each page that needs patching.
An alternative is to use annotation pdfmarks or annotator software like [mutool](https://mupdf.com/docs/manual-mutool-run.html) but this solution is simple and accurate, and avoids annotations which are not always supported.
Note that with Ghostscript V9.21 the -dPreserveAnnots=false switch renders PDF annotations as content stream data.

## Usage

`gs -q -dNOSAFER -sDEVICE=pdfwrite -o out.pdf -sPDFname=in.pdf -- pdfpatch.ps args`  
to pass patch arguments on the commandline, or  
`gs -q -dNOSAFER -sDEVICE=pdfwrite -o out.pdf -sPDFname=in.pdf -sargname=args.txt pdfpatch.ps`  
to read patch arguments from a script file.
Patch arguments are in PostScript dictionary syntax, e.g.:  
```
/page 2 /colour 16#ffee11 /x 20 /y 100 /width 20 /height 5 /operation (rect)
/page 3 /colour 0 /x 21 /y 99 /font (Arial,Italic) /fontsize 11 /text (my text) /operation (write)
```

## Options

File options:  
`-sPDFname=in.pdf` — the PDF file to modify  
`-o out.pdf` — the output PDF file  
`-sargname=args.txt` — the optional patch script file  
`-dShowFonts` — dump all available fonts then quit, e.g. `gs -q -dNODISPLAY -dShowFonts pdfpatch.ps`  
PDF metadata options:  
`-sPDFtitle='My Title'` — set Title  
`-sPDFauthor='My Name'` — set Author  
`-sPDFsubject='My Subject'` — set Subject  
Ghostscript options can be used as well, of course, e.g.  
`-dCompatibilityLevel=1.x` — set PDF compatibility to 1.2 or higher  
`-dCompressPages=false` — output uncompressed PDF  
`-dFILTERIMAGE` — remove images  

## Arguments

These are the patch arguments and default values:
* `/from-top true` — measure from top of page
* `/offset-x 0` — fixed horizontal offest (mm)
* `/offset-y 0` — fixed vertical offest (mm)
* `/page 1` — page number, 1 to last
* `/font (Times-Roman)` — font resource name
* `/fontsize 12` — point size
* `/text ()` — text to write
* `/x 0` — rectangle left or text insert point (mm)
* `/y 0` — rectangle bottom or text baseline (mm)
* `/width 100` — rectangle width (mm)
* `/height 100` — rectangle height (mm)
* `/colour 16#000000` — paint colour (RRGGBB)
* `/operation null` — `rect` or `write`

Each patch ends when an `/operation` value is encountered, then it is cloned for the next patch operation.

## Example

This fragment changes words in Old English to modern language for an old hymn:  
![before](https://user-images.githubusercontent.com/35268161/61661018-10aec480-acc3-11e9-8171-89dfa914fc76.png)
![after](https://user-images.githubusercontent.com/35268161/104820375-35d6b180-582c-11eb-911b-7ed7ea5dedda.png)
(The cream rub-out patches would normally be white)  
The patch script looks like:
```
/font (Times-Roman)
/fontsize 12
/page 1

% rects ==================================================

/colour 16#ffffcc % cream
/height 4.5

% verse 3

/y 190.4 % 189.1 + 1.3

% Thou
/x 18
/width 10
/operation (rect)

% shalt
/x 56
/width 10
/operation (rect)

% verse 4

/y 194.6 % 193.3 + 1.3

% Thy
/x 28
/width 9
/operation (rect)

% Thou art
/x 65
/width 17
/operation (rect)

% texts ==================================================

/colour 0 % black

% verse 3

/y 189.1

/x 20
/text (You)
/operation (write)

/x 57
/text (shall)
/operation (write)

% verse 4

/y 193.3

/x 29
/text (your)
/operation (write)

/x 67
/text (you)
/operation (write)

/x 76
/text (are)
/operation (write)
```
