---
layout: post
title:  "Customizing BibLaTeX's style files"
date:   2022-12-29 20:58:00 +0900
tags: 
- biblatex
- latex
---

Papers written in Japanese often have different citation formats for Japanese and English references, which has become a bottleneck in using BibTeX and BibLaTeX.

I recently created a [style file](https://github.com/sbtseiji/biblatex-jpa) to handle a mixture of Japanese and English citations in BibLaTeX. Here, I have written down what I have learned from reading BibLaTeX manuals and other sources as a memorandum.

This information may not be necessary if you use only European languages. Still, you may want to change the output of BibLaTeX a little. I hope this will be of help to you in that case.


## Basics

BibLaTeX uses `.bbx`, `.cbx`, and `.dbx`, to specify how to process the bibliography. The `.bbx` file defines the format of the bibliography, and the `.cbx` file is the citation format for the text. The `.dbx` file defines fields used in the bibliography. If you want to change the bibliography format, but leave the in-text citation format as it is in `authoryear`, you need to create a `mystyle.bbx` file, for example. And then, write the following in the preamble:

```
use package[backend=biber,citestyle=authoryear, bibstyle=mystyle]{biblatex}
```

However, if you always use this combination, you can prepare a mystyle.cbx, which contains the following: 

```
\ProvidesFile{mystyle.cbx}[2022/12/01\space my citation style]
\RequireCitationStyle{authoryear}
```

In this way, you can include the existing style file in your `.cbx` file. Then, you can use your original style by simply writing like this.

```
use package[backend=biber,style=mystyle]{biblatex}
```

The .bbx and .cbx files are necessary, but the .dbx file is optional.

## Creating your custom .bbx file

When creating a .bbx file, you don't need to write all the styles from scratch. As with .cbx files, you can include an existing style file and overwrite only the needed parts.

```
\ProvidesFile{mystyle.bbx}[2022/12/01\space my bibliography style]
\RequireBibliographyStyle{authoryear}
```

### How .bbx file works

When making changes to the bib file, you need to understand how BibLaTeX generates the bibliography. It is pretty challenging to do with the BibLaTeX manual alone. In doing so, I found the contents of basic style files such as authoryear.bbx and standard.bbx, and the contents defined in biblatex.def to be very helpful.

BibLaTeX seems to process bibliographic information basically in the following flow.

1. Assign a BibliographyDriver to each type of bib entry.

2. The BibliographyDriver processes field data in the specified order.

3. Perform necessary processing on field data with bibmacro.

4. Output the contents of each field according to the specified format.

5. In doing so, use the pre-specified delimiter.

In most cases, changes will be made at one of these levels, 2 through 5. For example, suppose you want to change the order in which authors' names, publication years, article titles, etc., are displayed in the bibliography. In that case, you need to make changes at level 2. On the other hand, if you only want to change the way the names are displayed, you need to make changes at levels 3 through 5.

### BibliographyDriver

BibLaTeX sets a BibliographyDriver for each entry type, such as `article` or `book`. This specifies which field data is processed for each entry and in what order. For example, if you want a book-type bibliography to be ordered by `author`, `title`, and `year` of publication, you would redefine the BibliographyDriver for `book`.


The process, in this case, can be described in its simplest form as follows:

```

{% raw %}\DeclareBibliographyDriver{book}{%{% endraw %}
  \usebibmacro{begentry}%
    \printnames{author}%
    \printfield{title}%
    \printdate% 
  \usebibmacro{finentry}%
}
```

