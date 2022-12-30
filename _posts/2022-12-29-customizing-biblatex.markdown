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


The processing of each entry type begins with `begentry` and ends with `finentry`, so the processing of each field is written between these two. The `\printnames` is a command to output the contents of a list of name types, such as author and editor names. How the field contents are displayed is defined by the `Format` of each field.

The command to print the contents of a field depends on the field type (`name`, `list`, `field`, or `date`) and is different for each field type: `\printnames{}` for `name`, `\printlist{}` for `list`, `\printdate{}` for `date`, and `\printfiled{}` for other fields. For more information on the field type you want to display, please refer to the BibLaTeX manual. Commands that print names, such as the author name field, accept options such as `\printnames[given-family]{}` or `\printnames[family-given]{}`. If you want to print the author's name in "familyname-firstname" order, you can use `\printnames[family-given]{author}`.

### bibmacro


Sometimes, you may want to process field data or treat multiple fields as a set before outputting the field contents with the `\print` command. In such cases, you can use `bibmacro`. For example, if you want to use the output "author (year of publication)." for multiple entry types, it is better to define them all together as a macro such as `author+year` than to write the same processing for each driver. The definition of a macro `author+year` which displays the field information in "author (year of publication)." format is something like this:

```
{% raw %}\newbibmacro{author+year}{%{% endraw %}
  \printnames{author}% print author(s)
  \printtext[parens]{\printdate}% print date in parentheses
}
```

Suppose you are including an existing style file and modifying it. In that case, basic macros such as `author+year` may already be defined in the file you are loading. In such a case, use `\renewbibmacro` instead of `\newbibmacro` for the macro definition. This is the same as the relationship between `newcommand` and `renewcommand`, so I don't think further explanation is necessary.

### Field Format

When an output commands such as `\printnames` is executed, the field's contents are formatted and typeset according to the format specified by `\DeclareXXFormat`. For example, the output format of the author name field `author` is defined by `\DeclarNameFormat{author}`. When `\printnames{author}` is called, the contents of the name field are output according to this `\DeclarNameFormat{author}`.

For example, suppose that the \DeclareNameFormat{author} is set as follows:

```
\namepartfamily%
\space%
\namepartgiven%
```

Then `\printnames{author}` will print the authors' names in the order "last-name first-name".

For lists of name such as `author`, formats such as `\DeclarNameFormat{family-given}` and `\DeclarNameFormat{given-family}` are also defined. And when specified as `\printnames[family-given]{author}`, the content of the name field is output in the format specified in `\DeclarNameFormat{family-given}`.


The name type and list type fields can contain multiple values. If these fields have multiple values, the processing is performed repeatedly for that number of values according to this format definition. At this time, information on how many items are currently being processed is stored in the `listcount` counter, and information on how many names are contained in the name field is stored in the `listtotal` counter. The process can be branched according to the author's order using these counter values.

For example, the bibliography format of the Japanese Psychological Association and APA requires rather complicated processing: if there are 20 or fewer co-authors, all authors are displayed; if there are more than 20, up to the 19th author is displayed, the middle is omitted with "...", and the last author's name is displayed. This process can be handled by defining `\DeclarNameFormat{family-given}` as follows:

```
{% raw %}\ifthenelse{\value{listcount}=20\AND\value{listcount}<\value{listtotal}}{%{% endraw %}
  % If the author currently being processed is the 20th author and there are still more authors after this one
  　　 \ldots% print '…'
  {% raw %}}{%{% endraw %}
    \ifthenelse{\value{listcount}>20\AND\value{listcount}<\value{listtotal}}{
      % If the author currently being processed is later than the 20th author and is not the last author
      % print nothing
    {% raw %}}{%{% endraw %}
      \namepartfamily\space\namepartgiven% print "family-name given-name"
      {% raw %}\ifthenelse{\value{listcount}=\value{listtotal}}{%{% endraw %}
        % If the author currently being processed is the last author
        % print nothing
      {% raw %}}{%{% endraw %}
        \printtext{, }% print ", "
      }%
    }%
  }%
}
```

The format definition by `\Declare***Format{}` can also be applied only to a specific entry type by using `\Declare***Format[entry type]{}`. So, for example, the following definition could be used to display author names in order of first name and last name for `article` entries, and in order of last name and first name for `book` entries.

```
\DeclareNameFormat[article]{author}{\namepartfamily\space\namepartgiven}%
\DeclareNameFormat[book]{author}{\namepartgiven\space\namepartfamily}%
```

### Delimiters

If you only want to use " " instead of "," as the delimiter between the author's last name and first name, it may be sufficient to redefine the delimiter used in each field's format without redefining the field's format. For example, the delimiter for displaying authors' names in "family-given" order is defined as `\revsdnamepunct`. And in authoryear style and apa style, its content is `\addcomma\space`. If you change this to `\space`, only a space will be inserted between the author's last name and first name.

However, simply modifying `\revsdnamepunct` will cause all references to display a space between the author's last name and first name, regardless of language. If you want to change the delimiters according to language, such as "Familyname, Givenname" for English references and "Familyname Givenname" for Japanese references, set the `AtEveryBibitem`, which is executed every time a bibliography entry is read, as follows:

```
{% raw %}\AtEveryBibitem{%{% endraw %}
  {% raw %}\ifthenelse{\equal{\thefirstlistitem{language}}{japanese}}{%{% endraw %}
    % If the language of the reference is Japanese, the delimiter is a space
    \renewcommand*{\revsdnamepunct}{\space}%
  {% raw %}}{%{% endraw %}
    % If the language of the reference is not Japanese, the delimiter is a comma and a space
    \renewcommand*{\revsdnamepunct}{\addcomma\space}%
  }%
}
```

In BibLaTeX, several delimiters are predefined under the names `***punct` and `***delim`. Please refer to the BibLaTeX manual, the `biblatex.def` file, or the `.bib` file of the style you are using as a base to find out what delimiters are defined and how. These delimiters can also be specified only in specific contexts by:

```
\DeclareDelimFormat[bib,biblist]{\revsdnamepunct}{\addcomma\space}%
```

In this case, the specification for this delimiter applies only to the context of the bibliography list and not to other contexts, e.g., citations in the text.


## Creating your .cbx file

The settings for `.cbx` files are the same as for `.bbx` files. In `.cbx` files, macros are defined for each citation command, such as `\textcite` and `\parencite`. The desired results can be obtained by changing the settings of these macros. These macros are executed for each literature entry to be cited, so for example, by saving the author hash of the last entry, multiple references by the same author, such as "Yamada (2001)" and "Yamada (2005)", can be combined and printed as "Yamada (2001, 2005)".

```
{% raw %}\renewbibmacro{textcite}{%{% endraw %}
  {% raw %}\iffieldequals{namehash}{\cbx@lasthash}{%{% endraw %}
    % If the author of this entry is the same as the last entry, do not repeat the author's name
    \setunit{\compcitedelim}%                print delimiter
    \usebibmacro{cite:plabelyear+extradate}% print year
  {% raw %}}{%{% endraw %}
    % Otherwise print author's name
    \printnames{labelname}%                  print author label
    \setunit{\printdelim{nameyeardelim}}%    print delimiter
    \usebibmacro{cite:plabelyear+extradate}% print year
    \savefield{namehash}{\cbx@lasthash}}%    save the hash of this entry
    \setunit{\multicitedelim}%               print citation delimiter
  }%
}
```

As already mentioned, `.cbx` files can be loaded and overwritten with existing style files using `RequireCitationStyle{}`, so it is best to load the style file closest to the one you want to make modifications only where necessary.