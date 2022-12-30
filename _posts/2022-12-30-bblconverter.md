---
layout: post
title:  "bblconverter: Convert BibLaTeX bibliography files to LaTeX, markdown, and docx"
date:   2022-12-30 11:55:00 +0900
tags: 
- biblatex
- yaml
- python
---

When I created the `.bbx` file to handle a mixture of Japanese and English references in BibLaTeX, I understood how BibLaTeX makes the bibliography. Based on this experience, I created a tool [`bblconverter`](https://github.com/sbtseiji/bblconverter) to convert bbl files created by BibLaTeX to plain LaTeX, bibitem, markdown, and docx.

The bbl file has a straightforward structure. It just arranges each item of the bib file into a list format. So it is not very difficult to convert it into a bibitem or markdown file. However, suppose you convert the `.bbl` file directly to LaTeX or markdown. In that case, you will have to rewrite the script when submitting your paper to a journal with different rules for constructing a references list, or if the references list format is changed in the future. And which is something you do not want to do. Therefore, I decided to use the YAML format for the bibliography in bblconverter.

Since bblconverter converts `.bbl` files into YAML lists when creating a references list, you can create LaTeX, markdown, and docx bibliographies from `.bbl` files as well as YAML formatted bibliography files.

## YAML formatted bibliography file

The bibliography file in YAML format converted from the `.bbl` file has the following form.

```
- entry: Clement2002
  entrytype: book
  skip: false
  author:
  - family: Clement
    familyi: C.
    given: E.
    giveni: E.
  publisher:
  - Wiley
  sortinit: C
  sortinithash: 4d103a86280481745c9c897c925753c0
  extradatescope: labelyear
  labeldatesource:
  labelnamesource: author
  labeltitlesource: title
  subtitle: The Cornerstone of Learning
  title: Cognitive Flexibility
  year: 2002
  dateera: ce
```

Each entry begins with a pair of "`entry:`" keys and a citation key ("Clement2002" in the example above), followed by each element of the bibliography information. The "`entrytype:`" is the type of the bibliography corresponding to "`@article`" or "`@book`" in the bib file. The subsequent "`skip:`" is a flag indicating whether or not to process this entry.

In the case of an entry that uses the "`related:`" field, such as a translation or a book in a series, it has a related entry whose cite key is identical to the "`related:`" field of the entry. For such related entries, the "`skip:`" field is set to "`true`" and is ignored when creating the bibliography.

Fields related to the author's or editor's names, such as "`author:`", contain information on each author in a list format. In these fields, "`family:`" means the family name, "`familyi:`" means the initials of the family name, "`given:`" means the given name, and "`giveni:`" means the initials of the given name.

The "`publisher:`" is also treated as a list in BibLaTeX, so it is in list format, but I think it is unlikely to list more than one publisher, so I only look at the first entry in bibconverter. Therefore, it will probably work even if it says something like "publisher: Wiley". If you specify more than one, it may cause an error.

Since this YAML is almost a verbatim transcription of the contents of the `.bbl` file, it contains many items that are not necessary for the creation of the bibliography. In this example, the "`sortinit:`", "`extradatescope:`", "`labelnamesource:`", "`labeltitlesource:`", and "`dateera:`" are not necessary. Try converting an actual `.bbl` file to YAML to see what other items are included. `bblconverter` allows you to export a YAML converted bib file.

## YAML for creating bibliography lists

`bblconverter` uses a Domain Specific Language (DSL) written in YAML to create the bibliography. Therefore, a wide range of journals can be supported by preparing a formatter YAML that matches the bibliography style of each journal.

The `bblconverter`'s formatter YAML consists of three parts: `constants:` (constants), `names:` (names list), and `driver:` (formatter for each entry type). The first part, `constants:`, specifies the constants used in the bibliography format, for example, the maximum number of authors to be listed in the bibliography. Here you specify the name and value of the constant in dictionary form as an example below. The constants defined here can be referenced in conditional clauses within the `driver:` part. This part can be omitted if there is no need to use constants.

```
constants:
  maxnames: 20
```

The second part, `names:`, specifies fields that are treated as namelist in BibLaTeX, such as the author name and editor name. This part is described in list form as follows:

```
names:
  - author
  - editor
  - editora
  - translator
  - translatora
  - origauthor
```

The third part, `driver:`, is the central part of the formatter YAML. It describes the format of each entry type, such as `article:` and `book:`. The format of each entry type is composed of "`field name: field format`," which is described in a list format in the order in which they are listed in the bibliography list.

For example, if you want to format `article` entries as "author name, year, title, journal title, volume, and pages", you can write the YAML in the following format. In this case, the name of each field must be the field name used by BibLaTeX (the field name used in the bib file in YAML format).


```
article:
  - author: format of the author names
  - year: format of the published year
  - title: format of the article's title
  - journaltitle: format of the journal name
  - volume: format of the volume number
  - pages: format of the pages
```

The format of each field is written by connecting the following commands in a list format.

```
value::FIELDNAME　Retrieves and prints field value
text::"STRING"　prints specified text
delim::DELIMITER　prints a delimiter
italic::true　starts formatting text as italic
italic::false　ends formatting text as italic
bold::true　starts formatting text as bold
bold::false　ends formatting text as bold
punct::PUNCTUATION　prints punctuation mark
```

The first `value::` is a command to retrieve and print the field's value specified by the field name. For example, suppose you want to display the name of the journal in which the article was published. In that case, you can use the following statement:

```
- journaltitle: value::journaltitle
```

The second command, `text::`, displays the specified string as is. For example, if you want the year of publication as "(2001).", the year of publication enclosed in "().", you can use the following statement:

```
- year: 
  - text::"("
  - value::year
  - text::")."
```

The third command, `delim::`, is used to display delimiters, such as `delim::COLON`. It is to avoid the problem that some characters, such as the colon (:), are recognized as special characters in YAML.

The delimiter characters defined are `COLON`, `SPACE`, `COMMA`, `PERIOD`, `DOT` (the same as `PERIOD`), `DOTS` (...), `EMDASH`, `NDASH`, and `LINEBREAK`.

The commands `italic::true` and `italic::false`, `bold::true` and `bold::false` are used to specify the text style. For example, to display book titles in italics, use the following statement:

```
- booktitle: 
  - italic::true
  - value::booktitle
  - italic::false
```

The last command, `punct::`, is for correctly processing titles with and without periods, etc. Suppose the entries in your bib file are with and without a period at the end of the title, such as "This is my first paper" and "This is my second paper.". In that case, the period at the end of the title "This is my second paper." is doubled as "This is my second paper.." if you specify your `title:` field as follows:

```
- title: 
  - value::title
  - delim::DOT
```

In that case, you can use `punct::"."`, instead of `delim::DOT` or `text::"."`. 

```
- title: 
  - value::title
  - punct::"."
```

Then the periods at the end of titles will be handled properly, and both "This is my first paper" and "This is my second paper." will be printed as "This is my first paper." and "This is my second paper."

### Conditional

The bblconverter allows conditional in the `driver:`'. Conditionals are denoted by `cond::` and are described as a list of three elements. You may omit the false case.

```
- - cond::CONDITIONAL
  - The value when the conditional is true
  - The value when the conditional is false
```

The following conditional are available:

* `cond::ifequal[value 1, value 2]`　Check if the values 1 and 2 are the same.
* `cond::ifgreater[value 1, value 2]`　Check if the value 1 is greater than the value 2.
* `cond::ifgreatereq[value 1, value 2]`　Check if the value 1 is greater than or equal to the value 2.
* `cond::ifless[value 1, value 2]`　Check if the value 1 is less than the value 2.
* `cond::iflesseq[value 1, value 2]`　Check if the value 1 is less than or equal to the value 2.
* `cond::ifdef[field::FIELDNAME, true]`　Check if the field `FIELDNAME` is defined.
* `cond::ifdef[field::FIELDNAME, false]`　Check if the field `FIELDNAME` is undefined.


Using the logical operators `&&`, `||`, and `^^`, you can also combine two conditionals. The second conditional does not have a `cond::`.

* `&&` logical conjunction　Ex) `cond::ifgreater[listcount,2]&&ifless[listcount,5]` (returns `true` when the value of the `listcount` variable is greater than two, and smaller than five)
*`||` logical disjunction　Ex) `cond::ifgreater[listcount,5]||ifless[listcount,2]` (returns `true` when the value of the `listcount` variable is greater than five, or smaller than two)
*`^^` exclusive disjunction　Ex)`cond::ifgreater[listtotal,2]^^ifdef[field::title,false]` (returns `true` when the value of the `listtotal` variable is greater than two, or the title field is undefined, but both are not true.）

### Variables

In addition to the constants specified by `constants:`, you can use the variables `listcount` and `listtotal` in the conditionals. The `listcount` is a value indicating the number of the author currently being processed counting from the top, and the `listtotal` indicates the total number of author names included in the entry. Using these variables, you can omit to print the author names when the number of authors exceeds 20, as in the following example:

```
jname: &jname 
    - cond::ifequal[listcount,maxnames]&&ifless[listcount,listtotal]
    - delim::DOTS
    - - cond::ifgreater[listcount,maxnames]&&ifless[listcount,listtotal]
      - text::""
      - - cond::ifequal[listcount,1]
        - - value::family
          - text::", "
          - value::given
        - - cond::ifless[listcount,maxnames]
          - - text::", "
            - value::family
            - text::", "
            - value::given
          - - value::family
            - text::", "
            - value::given
```