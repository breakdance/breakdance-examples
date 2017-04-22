# CommonMark Spec

Version 0.27 (2016-11-18)
John MacFarlane
<br>
CommonMark Spec by [John MacFarlane](http://spec.commonmark.org) is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

* [1Introduction](#introduction)
  - [1.1What is Markdown?](#what-is-markdown-)
  - [1.2Why is a spec needed?](#why-is-a-spec-needed-)
  - [1.3About this document](#about-this-document)
* [2Preliminaries](#preliminaries)
  - [2.1Characters and lines](#characters-and-lines)
  - [2.2Tabs](#tabs)
  - [2.3Insecure characters](#insecure-characters)
* [3Blocks and inlines](#blocks-and-inlines)
  - [3.1Precedence](#precedence)
  - [3.2Container blocks and leaf blocks](#container-blocks-and-leaf-blocks)
* [4Leaf blocks](#leaf-blocks)
  - [4.1Thematic breaks](#thematic-breaks)
  - [4.2ATX headings](#atx-headings)
  - [4.3Setext headings](#setext-headings)
  - [4.4Indented code blocks](#indented-code-blocks)
  - [4.5Fenced code blocks](#fenced-code-blocks)
  - [4.6HTML blocks](#html-blocks)
  - [4.7Link reference definitions](#link-reference-definitions)
  - [4.8Paragraphs](#paragraphs)
  - [4.9Blank lines](#blank-lines)
* [5Container blocks](#container-blocks)
  - [5.1Block quotes](#block-quotes)
  - [5.2List items](#list-items)
  - [5.3Lists](#lists)
* [6Inlines](#inlines)
  - [6.1Backslash escapes](#backslash-escapes)
  - [6.2Entity and numeric character references](#entity-and-numeric-character-references)
  - [6.3Code spans](#code-spans)
  - [6.4Emphasis and strong emphasis](#emphasis-and-strong-emphasis)
  - [6.5Links](#links)
  - [6.6Images](#images)
  - [6.7Autolinks](#autolinks)
  - [6.8Raw HTML](#raw-html)
  - [6.9Hard line breaks](#hard-line-breaks)
  - [6.10Soft line breaks](#soft-line-breaks)
  - [6.11Textual content](#textual-content)
* [Appendix: A parsing strategy](#appendix-a-parsing-strategy)
  - [Overview](#overview)
  - [Phase 1: block structure](#phase-1-block-structure)
  - [Phase 2: inline structure](#phase-2-inline-structure)

# 1  Introduction

## 1.1  What is Markdown?

Markdown is a plain text format for writing structured documents, based on conventions used for indicating formatting in email and usenet posts. It was developed in 2004 by John Gruber, who wrote the first Markdown-to-HTML converter in Perl, and it soon became ubiquitous. In the next decade, dozens of implementations were developed in many languages. Some extended the original Markdown syntax with conventions for footnotes, tables, and other document elements. Some allowed Markdown documents to be rendered in formats other than HTML. Websites like Reddit, StackOverflow, and GitHub had millions of people using Markdown. And Markdown started to be used beyond the web, to author books, articles, slide shows, letters, and lecture notes.

What distinguishes Markdown from many other lightweight markup syntaxes, which are often easier to write, is its readability. As Gruber writes:

> The overriding design goal for Markdown's formatting syntax is to make it as readable as possible. The idea is that a Markdown-formatted document should be publishable as-is, as plain text, without looking like it's been marked up with tags or formatting instructions. ([http://daringfireball.net/projects/markdown/](http://daringfireball.net/projects/markdown/))

The point can be illustrated by comparing a sample of [AsciiDoc](http://www.methods.co.nz/asciidoc/) with an equivalent sample of Markdown. Here is a sample of AsciiDoc from the AsciiDoc manual:

```
1. List item one.
+
List item one continued with a second paragraph followed by an
Indented block.
+
.................
$ ls *.sh
$ mv *.sh ~/tmp
.................
+
List item continued with a third paragraph.

2. List item two continued with an open block.
+
--
This paragraph is part of the preceding list item.

a. This list is nested and does not require explicit item
continuation.
+
This paragraph is part of the preceding list item.

b. List item b.

This paragraph belongs to item two of the outer list.
--
```

And here is the equivalent in Markdown:

```
1.  List item one.

    List item one continued with a second paragraph followed by an
    Indented block.

        $ ls *.sh
        $ mv *.sh ~/tmp

    List item continued with a third paragraph.

2.  List item two continued with an open block.

    This paragraph is part of the preceding list item.

    1. This list is nested and does not require explicit item continuation.

       This paragraph is part of the preceding list item.

    2. List item b.

    This paragraph belongs to item two of the outer list.
```

The AsciiDoc version is, arguably, easier to write. You don't need to worry about indentation. But the Markdown version is much easier to read. The nesting of list items is apparent to the eye in the source, not just in the processed document.

## 1.2  Why is a spec needed?

John Gruber's [canonical description of Markdown's syntax](http://daringfireball.net/projects/markdown/syntax) does not specify the syntax unambiguously. Here are some examples of questions it does not answer:

1. How much indentation is needed for a sublist? The spec says that continuation paragraphs need to be indented four spaces, but is not fully explicit about sublists. It is natural to think that they, too, must be indented four spaces, but `Markdown.pl` does not require that. This is hardly a "corner case," and divergences between implementations on this issue often lead to surprises for users in real documents. (See [this comment by John Gruber](http://article.gmane.org/gmane.text.markdown.general/1997).)
2. Is a blank line needed before a block quote or heading? Most implementations do not require the blank line. However, this can lead to unexpected results in hard-wrapped text, and also to ambiguities in parsing (note that some implementations put the heading inside the blockquote, while others do not). (John Gruber has also spoken [in favor of requiring the blank lines](http://article.gmane.org/gmane.text.markdown.general/2146).)
3. Is a blank line needed before an indented code block? (`Markdown.pl` requires it, but this is not mentioned in the documentation, and some implementations do not require it.)

```md
paragraph
    code?
```
4. What is the exact rule for determining when list items get wrapped in `<p>` tags? Can a list be partially "loose" and partially "tight"? What should we do with a list like this?

```md
1. one

2. two
3. three
```

Or this?

```md
1.  one
    - a

    - b
2.  two
```

(There are some relevant comments by John Gruber [here](http://article.gmane.org/gmane.text.markdown.general/2554).)
5. Can list markers be indented? Can ordered list markers be right-aligned?

```md
 8. item 1
 9. item 2
10. item 2a
```
6. Is this one list with a thematic break in its second item, or two lists separated by a thematic break?

```md
* a
* * * * *
* b
```
7. When list markers change from numbers to bullets, do we have two lists or one? (The Markdown syntax description suggests two, but the perl scripts and many other implementations produce one.)

```md
1. fee
2. fie
-  foe
-  fum
```
8. What are the precedence rules for the markers of inline structure? For example, is the following a valid link, or does the code span take precedence ?

```md
[a backtick (`)](/url) and [another backtick (`)](/url).
```
9. What are the precedence rules for markers of emphasis and strong emphasis? For example, how should the following be parsed?

```md
*foo *bar* baz*
```
10. What are the precedence rules between block-level and inline-level structure? For example, how should the following be parsed?

```md
- `a long code span can contain a hyphen like this
  - and it can screw things up`
```
11. Can list items include section headings? (`Markdown.pl` does not allow this, but does allow blockquotes to include headings.)

```md
- # Heading
```
12. Can list items be empty?

```md
* a
*
* b
```
13. Can link references be defined inside block quotes or list items?

```md
> Blockquote [foo].
>
> [foo]: /url
```
14. If there are multiple definitions for the same reference, which takes precedence?

```md
[foo]: /url1
[foo]: /url2

[foo][]
```

In the absence of a spec, early implementers consulted `Markdown.pl` to resolve these ambiguities. But `Markdown.pl` was quite buggy, and gave manifestly bad results in many cases, so it was not a satisfactory replacement for a spec.

Because there is no unambiguous spec, implementations have diverged considerably. As a result, users are often surprised to find that a document that renders one way on one system (say, a github wiki) renders differently on another (say, converting to docbook using pandoc). To make matters worse, because nothing in Markdown counts as a "syntax error," the divergence often isn't discovered right away.

## 1.3  About this document

This document attempts to specify Markdown syntax unambiguously. It contains many examples with side-by-side Markdown and HTML. These are intended to double as conformance tests. An accompanying script `spec_tests.py` can be used to run the tests against any Markdown program:

```
python test/spec_tests.py --spec spec.txt --program PROGRAM
```

Since this document describes how Markdown is to be parsed into an abstract syntax tree, it would have made sense to use an abstract representation of the syntax tree instead of HTML. But HTML is capable of representing the structural distinctions we need to make, and the choice of HTML for the tests makes it possible to run the tests against an implementation without writing an abstract syntax tree renderer.

This document is generated from a text file, `spec.txt`, written in Markdown with a small extension for the side-by-side tests. The script `tools/makespec.py` can be used to convert `spec.txt` into HTML or CommonMark (which can then be converted into other formats).

In the examples, the `&#x2192;` character is used to represent tabs.

# 2  Preliminaries

## 2.1  Characters and lines

Any sequence of [characters](#character) is a valid CommonMark document.

A [character](#character) is a Unicode code point. Although some code points (for example, combining accents) do not correspond to characters in an intuitive sense, all code points count as characters for purposes of this spec.

This spec does not specify an encoding; it thinks of lines as composed of [characters](#character) rather than bytes. A conforming parser may be limited to a certain encoding.

A [line](#line) is a sequence of zero or more [characters](#character) other than newline (`U+000A`) or carriage return (`U+000D`), followed by a [line ending](#line-ending) or by the end of file.

A [line ending](#line-ending) is a newline (`U+000A`), a carriage return (`U+000D`) not followed by a newline, or a carriage return and a following newline.

A line containing no characters, or a line containing only spaces (`U+0020`) or tabs (`U+0009`), is called a [blank line](#blank-line).

The following definitions of character classes will be used in this spec:

A [whitespace character](#whitespace-character) is a space (`U+0020`), tab (`U+0009`), newline (`U+000A`), line tabulation (`U+000B`), form feed (`U+000C`), or carriage return (`U+000D`).

[Whitespace](#whitespace) is a sequence of one or more [whitespace characters](#whitespace-character).

A [Unicode whitespace character](#unicode-whitespace-character) is any code point in the Unicode `Zs` class, or a tab (`U+0009`), carriage return (`U+000D`), newline (`U+000A`), or form feed (`U+000C`).

[Unicode whitespace](#unicode-whitespace) is a sequence of one or more [Unicode whitespace characters](#unicode-whitespace-character).

A [space](#space) is `U+0020`.

A [non-whitespace character](#non-whitespace-character) is any character that is not a [whitespace character](#whitespace-character).

An [ASCII punctuation character](#ascii-punctuation-character) is `!`, `"`, `#`, `$`, `%`, `&`, `'`, `(`, `)`, `*`, `+`, `,`, `-`, `.`, `/`, `:`, `;`, `<`, `=`, `>`, `?`, `@`, `[`, `\`, `]`, `^`, `_`, `````, `{`, `|`, `}`, or `~`.

A [punctuation character](#punctuation-character) is an [ASCII punctuation character](#ascii-punctuation-character) or anything in the Unicode classes `Pc`, `Pd`, `Pe`, `Pf`, `Pi`, `Po`, or `Ps`.

## 2.2  Tabs

Tabs in lines are not expanded to [spaces](#space). However, in contexts where whitespace helps to define block structure, tabs behave as if they were replaced by spaces with a tab stop of 4 characters.

Thus, for example, a tab can be used instead of four spaces in an indented code block. (Note, however, that internal tabs are passed through as literal tabs, not expanded to spaces.)

[Example 1](#example-1)

```md
→foo→baz→→bim
```

```html
<pre><code>foo→baz→→bim
</code></pre>
```

[Example 2](#example-2)

```md
→foo→baz→→bim
```

```html
<pre><code>foo→baz→→bim
</code></pre>
```

[Example 3](#example-3)

```md
a→a
ὐ→a
```

```html
<pre><code>a→a
ὐ→a
</code></pre>
```

In the following example, a continuation paragraph of a list item is indented with a tab; this has exactly the same effect as indentation with four spaces would:

[Example 4](#example-4)

```md
-foo

→bar
```

```html
<ul>
<li>
<p>foo</p>
<p>bar</p>
</li>
</ul>
```

[Example 5](#example-5)

```md
-foo

→→bar
```

```html
<ul>
<li>
<p>foo</p>
<pre><code>bar
</code></pre>
</li>
</ul>
```

Normally the `>` that begins a block quote may be followed optionally by a space, which is not considered part of the content. In the following case `>` is followed by a tab, which is treated as if it were expanded into spaces. Since one of theses spaces is considered part of the delimiter, `foo` is considered to be indented six spaces inside the block quote context, so we get an indented code block starting with two spaces.

[Example 6](#example-6)

```md
>→→foo
```

```html
<blockquote>
<pre><code>foo
</code></pre>
</blockquote>
```

[Example 7](#example-7)

```md
-→→foo
```

```html
<ul>
<li>
<pre><code>foo
</code></pre>
</li>
</ul>
```

[Example 8](#example-8)

```md
foo
→bar
```

```html
<pre><code>foo
bar
</code></pre>
```

[Example 9](#example-9)

```md
-foo
-bar
→-baz
```

```html
<ul>
<li>foo
<ul>
<li>bar
<ul>
<li>baz</li>
</ul>
</li>
</ul>
</li>
</ul>
```

[Example 10](#example-10)

```md
#→Foo
```

```html
<h1>Foo</h1>
```

[Example 11](#example-11)

```md
*→*→*→
```

```html
<hr/>
```

## 2.3  Insecure characters

For security reasons, the Unicode character `U+0000` must be replaced with the REPLACEMENT CHARACTER (`U+FFFD`).

# 3  Blocks and inlines

We can think of a document as a sequence of [blocks](#blocks)—structural elements like paragraphs, block quotations, lists, headings, rules, and code blocks. Some blocks (like block quotes and list items) contain other blocks; others (like headings and paragraphs) contain [inline](#inline) content—text, links, emphasized text, images, code, and so on.

## 3.1  Precedence

Indicators of block structure always take precedence over indicators of inline structure. So, for example, the following is a list with two items, not a list with one item containing a code span:

[Example 12](#example-12)

```md
-`one
-two`
```

```html
<ul>
<li>`one</li>
<li>two`</li>
</ul>
```

This means that parsing can proceed in two steps: first, the block structure of the document can be discerned; second, text lines inside paragraphs, headings, and other block constructs can be parsed for inline structure. The second step requires information about link reference definitions that will be available only at the end of the first step. Note that the first step requires processing lines in sequence, but the second can be parallelized, since the inline parsing of one block element does not affect the inline parsing of any other.

## 3.2  Container blocks and leaf blocks

We can divide blocks into two types: [container block](#container-block)s, which can contain other blocks, and [leaf block](#leaf-block)s, which cannot.

# 4  Leaf blocks

This section describes the different kinds of leaf block that make up a Markdown document.

## 4.1  Thematic breaks

A line consisting of 0-3 spaces of indentation, followed by a sequence of three or more matching `-`, `_`, or `*` characters, each followed optionally by any number of spaces, forms a [thematic break](#thematic-break).

[Example 13](#example-13)

```md
***
---
___
```

```html
<hr/>
<hr/>
<hr/>
```

Wrong characters:

[Example 14](#example-14)

```md
+++
```

```html
<p>+++</p>
```

[Example 15](#example-15)

```md
===
```

```html
<p>===</p>
```

Not enough characters:

[Example 16](#example-16)

```md
--
**
__
```

```html
<p>--
**
__</p>
```

One to three spaces indent are allowed:

[Example 17](#example-17)

```md
***
***
***
```

```html
<hr/>
<hr/>
<hr/>
```

Four spaces is too many:

[Example 18](#example-18)

```md
***
```

```html
<pre><code>***
</code></pre>
```

[Example 19](#example-19)

```md
Foo
***
```

```html
<p>Foo
***</p>
```

More than three characters may be used:

[Example 20](#example-20)

```md
_____________________________________
```

```html
<hr/>
```

Spaces are allowed between the characters:

[Example 21](#example-21)

```md
---
```

```html
<hr/>
```

[Example 22](#example-22)

```md
***********
```

```html
<hr/>
```

[Example 23](#example-23)

```md
----
```

```html
<hr/>
```

Spaces are allowed at the end:

[Example 24](#example-24)

```md
----
```

```html
<hr/>
```

However, no other characters may occur in the line:

[Example 25](#example-25)

```md
____a

a------

---a---
```

```html
<p>____a</p>
<p>a------</p>
<p>---a---</p>
```

It is required that all of the [non-whitespace characters](#non-whitespace-character) be the same. So, this is not a thematic break:

[Example 26](#example-26)

```md
*-*
```

```html
<p><em>-</em></p>
```

Thematic breaks do not need blank lines before or after:

[Example 27](#example-27)

```md
-foo
***
-bar
```

```html
<ul>
<li>foo</li>
</ul>
<hr/>
<ul>
<li>bar</li>
</ul>
```

Thematic breaks can interrupt a paragraph:

[Example 28](#example-28)

```md
Foo
***
bar
```

```html
<p>Foo</p>
<hr/>
<p>bar</p>
```

If a line of dashes that meets the above conditions for being a thematic break could also be interpreted as the underline of a [setext heading](#setext-heading), the interpretation as a [setext heading](#setext-heading) takes precedence. Thus, for example, this is a setext heading, not a paragraph followed by a thematic break:

[Example 29](#example-29)

```md
Foo
---
bar
```

```html
<h2>Foo</h2>
<p>bar</p>
```

When both a thematic break and a list item are possible interpretations of a line, the thematic break takes precedence:

[Example 30](#example-30)

```md
*Foo
***
*Bar
```

```html
<ul>
<li>Foo</li>
</ul>
<hr/>
<ul>
<li>Bar</li>
</ul>
```

If you want a thematic break in a list item, use a different bullet:

[Example 31](#example-31)

```md
-Foo
-***
```

```html
<ul>
<li>Foo</li>
<li>
<hr/>
</li>
</ul>
```

## 4.2  ATX headings

An [ATX heading](#atx-heading) consists of a string of characters, parsed as inline content, between an opening sequence of 1–6 unescaped `#` characters and an optional closing sequence of any number of unescaped `#` characters. The opening sequence of `#` characters must be followed by a [space](#space) or by the end of line. The optional closing sequence of `#`s must be preceded by a [space](#space) and may be followed by spaces only. The opening `#` character may be indented 0-3 spaces. The raw contents of the heading are stripped of leading and trailing spaces before being parsed as inline content. The heading level is equal to the number of `#` characters in the opening sequence.

Simple headings:

[Example 32](#example-32)

```md
#foo
##foo
###foo
####foo
#####foo
######foo
```

```html
<h1>foo</h1>
<h2>foo</h2>
<h3>foo</h3>
<h4>foo</h4>
<h5>foo</h5>
<h6>foo</h6>
```

More than six `#` characters is not a heading:

[Example 33](#example-33)

```md
#######foo
```

```html
<p>#######foo</p>
```

At least one space is required between the `#` characters and the heading's contents, unless the heading is empty. Note that many implementations currently do not require the space. However, the space was required by the [original ATX implementation](http://www.aaronsw.com/2002/atx/atx.py), and it helps prevent things like the following from being parsed as headings:

[Example 34](#example-34)

```md
#5bolt

#hashtag
```

```html
<p>#5bolt</p>
<p>#hashtag</p>
```

This is not a heading, because the first `#` is escaped:

[Example 35](#example-35)

```md
\##foo
```

```html
<p>##foo</p>
```

Contents are parsed as inlines:

[Example 36](#example-36)

```md
#foo*bar*\*baz\*
```

```html
<h1>foo<em>bar</em>*baz*</h1>
```

Leading and trailing blanks are ignored in parsing inline content:

[Example 37](#example-37)

```md
#foo
```

```html
<h1>foo</h1>
```

One to three spaces indentation are allowed:

[Example 38](#example-38)

```md
###foo
##foo
#foo
```

```html
<h3>foo</h3>
<h2>foo</h2>
<h1>foo</h1>
```

Four spaces are too much:

[Example 39](#example-39)

```md
#foo
```

```html
<pre><code>#foo
</code></pre>
```

[Example 40](#example-40)

```md
foo
#bar
```

```html
<p>foo
#bar</p>
```

A closing sequence of `#` characters is optional:

[Example 41](#example-41)

```md
##foo##
###bar###
```

```html
<h2>foo</h2>
<h3>bar</h3>
```

It need not be the same length as the opening sequence:

[Example 42](#example-42)

```md
#foo##################################
#####foo##
```

```html
<h1>foo</h1>
<h5>foo</h5>
```

Spaces are allowed after the closing sequence:

[Example 43](#example-43)

```md
###foo###
```

```html
<h3>foo</h3>
```

A sequence of `#` characters with anything but [spaces](#space) following it is not a closing sequence, but counts as part of the contents of the heading:

[Example 44](#example-44)

```md
###foo###b
```

```html
<h3>foo###b</h3>
```

The closing sequence must be preceded by a space:

[Example 45](#example-45)

```md
#foo#
```

```html
<h1>foo#</h1>
```

Backslash-escaped `#` characters do not count as part of the closing sequence:

[Example 46](#example-46)

```md
###foo\###
##foo#\##
#foo\#
```

```html
<h3>foo###</h3>
<h2>foo###</h2>
<h1>foo#</h1>
```

ATX headings need not be separated from surrounding content by blank lines, and they can interrupt paragraphs:

[Example 47](#example-47)

```md
****
##foo
****
```

```html
<hr/>
<h2>foo</h2>
<hr/>
```

[Example 48](#example-48)

```md
Foobar
#baz
Barfoo
```

```html
<p>Foobar</p>
<h1>baz</h1>
<p>Barfoo</p>
```

ATX headings can be empty:

[Example 49](#example-49)

```md
##
#
######
```

```html
<h2></h2>
<h1></h1>
<h3></h3>
```

## 4.3  Setext headings

A [setext heading](#setext-heading) consists of one or more lines of text, each containing at least one [non-whitespace character](#non-whitespace-character), with no more than 3 spaces indentation, followed by a [setext heading underline](#setext-heading-underline). The lines of text must be such that, were they not followed by the setext heading underline, they would be interpreted as a paragraph: they cannot be interpretable as a [code fence](#code-fence), [ATX heading](#atx-headings), [block quote](#block-quotes), [thematic break](#thematic-break), [list item](#list-items), or [HTML block](#html-blocks).

A [setext heading underline](#setext-heading-underline) is a sequence of `=` characters or a sequence of `-` characters, with no more than 3 spaces indentation and any number of trailing spaces. If a line containing a single `-` can be interpreted as an empty [list items](#list-items), it should be interpreted this way and not as a [setext heading underline](#setext-heading-underline).

The heading is a level 1 heading if `=` characters are used in the [setext heading underline](#setext-heading-underline), and a level 2 heading if `-` characters are used. The contents of the heading are the result of parsing the preceding lines of text as CommonMark inline content.

In general, a setext heading need not be preceded or followed by a blank line. However, it cannot interrupt a paragraph, so when a setext heading comes after a paragraph, a blank line is needed between them.

Simple examples:

[Example 50](#example-50)

```md
Foo*bar*
=========

Foo*bar*
---------
```

```html
<h1>Foo<em>bar</em></h1>
<h2>Foo<em>bar</em></h2>
```

The content of the header may span more than one line:

[Example 51](#example-51)

```md
Foo*bar
baz*
====
```

```html
<h1>Foo<em>bar
baz</em></h1>
```

The underlining can be any length:

[Example 52](#example-52)

```md
Foo
-------------------------

Foo
=
```

```html
<h2>Foo</h2>
<h1>Foo</h1>
```

The heading content can be indented up to three spaces, and need not line up with the underlining:

[Example 53](#example-53)

```md
Foo
---

Foo
-----

Foo
===
```

```html
<h2>Foo</h2>
<h2>Foo</h2>
<h1>Foo</h1>
```

Four spaces indent is too much:

[Example 54](#example-54)

```md
Foo
---

Foo
---
```

```html
<pre><code>Foo
---

Foo
</code></pre>
<hr/>
```

The setext heading underline can be indented up to three spaces, and may have trailing spaces:

[Example 55](#example-55)

```md
Foo
----
```

```html
<h2>Foo</h2>
```

Four spaces is too much:

[Example 56](#example-56)

```md
Foo
---
```

```html
<p>Foo
---</p>
```

The setext heading underline cannot contain internal spaces:

[Example 57](#example-57)

```md
Foo
==

Foo
----
```

```html
<p>Foo
==</p>
<p>Foo</p>
<hr/>
```

Trailing spaces in the content line do not cause a line break:

[Example 58](#example-58)

```md
Foo
-----
```

```html
<h2>Foo</h2>
```

Nor does a backslash at the end:

[Example 59](#example-59)

```md
Foo\
----
```

```html
<h2>Foo\</h2>
```

Since indicators of block structure take precedence over indicators of inline structure, the following are setext headings:

[Example 60](#example-60)

```md
`Foo
----
`

<atitle="alot
---
ofdashes"/>
```

```html
<h2>`Foo</h2>
<p>`</p>
<h2>&lt;atitle=&quot;alot</h2>
<p>ofdashes&quot;/&gt;</p>
```

The setext heading underline cannot be a [lazy continuation line](#lazy-continuation-line) in a list item or block quote:

[Example 61](#example-61)

```md
>Foo
---
```

```html
<blockquote>
<p>Foo</p>
</blockquote>
<hr/>
```

[Example 62](#example-62)

```md
>foo
bar
===
```

```html
<blockquote>
<p>foo
bar
===</p>
</blockquote>
```

[Example 63](#example-63)

```md
-Foo
---
```

```html
<ul>
<li>Foo</li>
</ul>
<hr/>
```

A blank line is needed between a paragraph and a following setext heading, since otherwise the paragraph becomes part of the heading's content:

[Example 64](#example-64)

```md
Foo
Bar
---
```

```html
<h2>Foo
Bar</h2>
```

But in general a blank line is not required before or after setext headings:

[Example 65](#example-65)

```md
---
Foo
---
Bar
---
Baz
```

```html
<hr/>
<h2>Foo</h2>
<h2>Bar</h2>
<p>Baz</p>
```

Setext headings cannot be empty:

[Example 66](#example-66)

```md

====
```

```html
<p>====</p>
```

Setext heading text lines must not be interpretable as block constructs other than paragraphs. So, the line of dashes in these examples gets interpreted as a thematic break:

[Example 67](#example-67)

```md
---
---
```

```html
<hr/>
<hr/>
```

[Example 68](#example-68)

```md
-foo
-----
```

```html
<ul>
<li>foo</li>
</ul>
<hr/>
```

[Example 69](#example-69)

```md
foo
---
```

```html
<pre><code>foo
</code></pre>
<hr/>
```

[Example 70](#example-70)

```md
>foo
-----
```

```html
<blockquote>
<p>foo</p>
</blockquote>
<hr/>
```

If you want a heading with `> foo` as its literal text, you can use backslash escapes:

[Example 71](#example-71)

```md
\>foo
------
```

```html
<h2>&gt;foo</h2>
```

**Compatibility note:** Most existing Markdown implementations do not allow the text of setext headings to span multiple lines. But there is no consensus about how to interpret

```md
Foo
bar
---
baz
```

One can find four different interpretations:

1. paragraph "Foo", heading "bar", paragraph "baz"
2. paragraph "Foo bar", thematic break, paragraph "baz"
3. paragraph "Foo bar — baz"
4. heading "Foo bar", paragraph "baz"

We find interpretation 4 most natural, and interpretation 4 increases the expressive power of CommonMark, by allowing multiline headings. Authors who want interpretation 1 can put a blank line after the first paragraph:

[Example 72](#example-72)

```md
Foo

bar
---
baz
```

```html
<p>Foo</p>
<h2>bar</h2>
<p>baz</p>
```

Authors who want interpretation 2 can put blank lines around the thematic break,

[Example 73](#example-73)

```md
Foo
bar

---

baz
```

```html
<p>Foo
bar</p>
<hr/>
<p>baz</p>
```

or use a thematic break that cannot count as a [setext heading underline](#setext-heading-underline), such as

[Example 74](#example-74)

```md
Foo
bar
***
baz
```

```html
<p>Foo
bar</p>
<hr/>
<p>baz</p>
```

Authors who want interpretation 3 can use backslash escapes:

[Example 75](#example-75)

```md
Foo
bar
\---
baz
```

```html
<p>Foo
bar
---
baz</p>
```

## 4.4  Indented code blocks

An [indented code block](#indented-code-block) is composed of one or more [indented chunks](#indented-chunk) separated by blank lines. An [indented chunk](#indented-chunk) is a sequence of non-blank lines, each indented four or more spaces. The contents of the code block are the literal contents of the lines, including trailing [line endings](#line-ending), minus four spaces of indentation. An indented code block has no [info string](#info-string).

An indented code block cannot interrupt a paragraph, so there must be a blank line between a paragraph and a following indented code block. (A blank line is not needed, however, between a code block and a following paragraph.)

[Example 76](#example-76)

```md
asimple
indentedcodeblock
```

```html
<pre><code>asimple
indentedcodeblock
</code></pre>
```

If there is any ambiguity between an interpretation of indentation as a code block and as indicating that material belongs to a [list item](#list-items), the list item interpretation takes precedence:

[Example 77](#example-77)

```md
-foo

bar
```

```html
<ul>
<li>
<p>foo</p>
<p>bar</p>
</li>
</ul>
```

[Example 78](#example-78)

```md
1.foo

-bar
```

```html
<ol>
<li>
<p>foo</p>
<ul>
<li>bar</li>
</ul>
</li>
</ol>
```

The contents of a code block are literal text, and do not get parsed as Markdown:

[Example 79](#example-79)

```md
<a/>
*hi*

-one
```

```html
<pre><code>&lt;a/&gt;
*hi*

-one
</code></pre>
```

Here we have three chunks separated by blank lines:

[Example 80](#example-80)

```md
chunk1

chunk2

chunk3
```

```html
<pre><code>chunk1

chunk2

chunk3
</code></pre>
```

Any initial spaces beyond four will be included in the content, even in interior blank lines:

[Example 81](#example-81)

```md
chunk1

chunk2
```

```html
<pre><code>chunk1

chunk2
</code></pre>
```

An indented code block cannot interrupt a paragraph. (This allows hanging indents and the like.)

[Example 82](#example-82)

```md
Foo
bar
```

```html
<p>Foo
bar</p>
```

However, any non-blank line with fewer than four leading spaces ends the code block immediately. So a paragraph may occur immediately after indented code:

[Example 83](#example-83)

```md
foo
bar
```

```html
<pre><code>foo
</code></pre>
<p>bar</p>
```

And indented code can occur immediately before and after other kinds of blocks:

[Example 84](#example-84)

```md
#Heading
foo
Heading
------
foo
----
```

```html
<h1>Heading</h1>
<pre><code>foo
</code></pre>
<h2>Heading</h2>
<pre><code>foo
</code></pre>
<hr/>
```

The first line can be indented more than four spaces:

[Example 85](#example-85)

```md
foo
bar
```

```html
<pre><code>foo
bar
</code></pre>
```

Blank lines preceding or following an indented code block are not included in it:

[Example 86](#example-86)

```md

foo

```

```html
<pre><code>foo
</code></pre>
```

Trailing spaces are included in the code block's content:

[Example 87](#example-87)

```md
foo
```

```html
<pre><code>foo
</code></pre>
```

## 4.5  Fenced code blocks

A [code fence](#code-fence) is a sequence of at least three consecutive backtick characters (`````) or tildes (`~`). (Tildes and backticks cannot be mixed.) A [fenced code block](#fenced-code-block) begins with a code fence, indented no more than three spaces.

The line with the opening code fence may optionally contain some text following the code fence; this is trimmed of leading and trailing spaces and called the [info string](#info-string). The [info string](#info-string) may not contain any backtick characters. (The reason for this restriction is that otherwise some inline code would be incorrectly interpreted as the beginning of a fenced code block.)

The content of the code block consists of all subsequent lines, until a closing [code fence](#code-fence) of the same type as the code block began with (backticks or tildes), and with at least as many backticks or tildes as the opening code fence. If the leading code fence is indented N spaces, then up to N spaces of indentation are removed from each line of the content (if present). (If a content line is not indented, it is preserved unchanged. If it is indented less than N spaces, all of the indentation is removed.)

The closing code fence may be indented up to three spaces, and may be followed only by spaces, which are ignored. If the end of the containing block (or document) is reached and no closing code fence has been found, the code block contains all of the lines after the opening code fence until the end of the containing block (or document). (An alternative spec would require backtracking in the event that a closing code fence is not found. But this makes parsing much less efficient, and there seems to be no real down side to the behavior described here.)

A fenced code block may interrupt a paragraph, and does not require a blank line either before or after.

The content of a code fence is treated as literal text, not parsed as inlines. The first word of the [info string](#info-string) is typically used to specify the language of the code sample, and rendered in the `class` attribute of the `code` tag. However, this spec does not mandate any particular treatment of the [info string](#info-string).

Here is a simple example with backticks:

[Example 88](#example-88)

```md
```
<
>
```
```

```html
<pre><code>&lt;
&gt;
</code></pre>
```

With tildes:

[Example 89](#example-89)

```md
~~~
<
>
~~~
```

```html
<pre><code>&lt;
&gt;
</code></pre>
```

The closing code fence must use the same character as the opening fence:

[Example 90](#example-90)

```md
```
aaa
~~~
```
```

```html
<pre><code>aaa
~~~
</code></pre>
```

[Example 91](#example-91)

```md
~~~
aaa
```
~~~
```

```html
<pre><code>aaa
```
</code></pre>
```

The closing code fence must be at least as long as the opening fence:

[Example 92](#example-92)

```md
````
aaa
```
``````
```

```html
<pre><code>aaa
```
</code></pre>
```

[Example 93](#example-93)

```md
~~~~
aaa
~~~
~~~~
```

```html
<pre><code>aaa
~~~
</code></pre>
```

Unclosed code blocks are closed by the end of the document (or the enclosing [block quote](#block-quotes) or [list item](#list-items)):

[Example 94](#example-94)

```md
```
```

```html
<pre><code></code></pre>
```

[Example 95](#example-95)

```md
`````

```
aaa
```

```html
<pre><code>
```
aaa
</code></pre>
```

[Example 96](#example-96)

```md
>```
>aaa

bbb
```

```html
<blockquote>
<pre><code>aaa
</code></pre>
</blockquote>
<p>bbb</p>
```

A code block can have all empty lines as its content:

[Example 97](#example-97)

```md
```

```
```

```html
<pre><code>

</code></pre>
```

A code block can be empty:

[Example 98](#example-98)

```md
```
```
```

```html
<pre><code></code></pre>
```

Fences can be indented. If the opening fence is indented, content lines will have equivalent opening indentation removed, if present:

[Example 99](#example-99)

```md
```
aaa
aaa
```
```

```html
<pre><code>aaa
aaa
</code></pre>
```

[Example 100](#example-100)

```md
```
aaa
aaa
aaa
```
```

```html
<pre><code>aaa
aaa
aaa
</code></pre>
```

[Example 101](#example-101)

```md
```
aaa
aaa
aaa
```
```

```html
<pre><code>aaa
aaa
aaa
</code></pre>
```

Four spaces indentation produces an indented code block:

[Example 102](#example-102)

```md
```
aaa
```
```

```html
<pre><code>```
aaa
```
</code></pre>
```

Closing fences may be indented by 0-3 spaces, and their indentation need not match that of the opening fence:

[Example 103](#example-103)

```md
```
aaa
```
```

```html
<pre><code>aaa
</code></pre>
```

[Example 104](#example-104)

```md
```
aaa
```
```

```html
<pre><code>aaa
</code></pre>
```

This is not a closing fence, because it is indented 4 spaces:

[Example 105](#example-105)

```md
```
aaa
```
```

```html
<pre><code>aaa
```
</code></pre>
```

Code fences (opening and closing) cannot contain internal spaces:

[Example 106](#example-106)

```md
``````
aaa
```

```html
<p><code></code>
aaa</p>
```

[Example 107](#example-107)

```md
~~~~~~
aaa
~~~~~
```

```html
<pre><code>aaa
~~~~~
</code></pre>
```

Fenced code blocks can interrupt paragraphs, and can be followed directly by paragraphs, without a blank line between:

[Example 108](#example-108)

```md
foo
```
bar
```
baz
```

```html
<p>foo</p>
<pre><code>bar
</code></pre>
<p>baz</p>
```

Other blocks can also occur before and after fenced code blocks without an intervening blank line:

[Example 109](#example-109)

```md
foo
---
~~~
bar
~~~
#baz
```

```html
<h2>foo</h2>
<pre><code>bar
</code></pre>
<h1>baz</h1>
```

An [info string](#info-string) can be provided after the opening code fence. Opening and closing spaces will be stripped, and the first word, prefixed with `language-`, is used as the value for the `class` attribute of the `code` element within the enclosing `pre` element.

[Example 110](#example-110)

```md
```ruby
deffoo(x)
return3
end
```
```

```html
<pre><codeclass="language-ruby">deffoo(x)
return3
end
</code></pre>
```

[Example 111](#example-111)

```md
~~~~rubystartline=3$%@#$
deffoo(x)
return3
end
~~~~~~~
```

```html
<pre><codeclass="language-ruby">deffoo(x)
return3
end
</code></pre>
```

[Example 112](#example-112)

```md
````;
````
```

```html
<pre><codeclass="language-;"></code></pre>
```

[Info strings](#info-string) for backtick code blocks cannot contain backticks:

[Example 113](#example-113)

```md
```aa```
foo
```

```html
<p><code>aa</code>
foo</p>
```

Closing code fences cannot have [info strings](#info-string):

[Example 114](#example-114)

```md
```
```aaa
```
```

```html
<pre><code>```aaa
</code></pre>
```

## 4.6  HTML blocks

An [HTML block](#html-block) is a group of lines that is treated as raw HTML (and will not be escaped in HTML output).

There are seven kinds of [HTML block](#html-block), which can be defined by their start and end conditions. The block begins with a line that meets a [start condition](#start-condition) (after up to three spaces optional indentation). It ends with the first subsequent line that meets a matching [end condition](#end-condition), or the last line of the document or other [container block](#container-block)), if no line is encountered that meets the [end condition](#end-condition). If the first line meets both the [start condition](#start-condition) and the [end condition](#end-condition), the block will contain just that line.

1. **Start condition:** line begins with the string `<script`, `<pre`, or `<style` (case-insensitive), followed by whitespace, the string `>`, or the end of the line.<br>
**End condition:** line contains an end tag `</script>`, `</pre>`, or `</style>` (case-insensitive; it need not match the start tag).
2. **Start condition:** line begins with the string `<!--`.<br>
**End condition:** line contains the string `-->`.
3. **Start condition:** line begins with the string `<?`.<br>
**End condition:** line contains the string `?>`.
4. **Start condition:** line begins with the string `<!` followed by an uppercase ASCII letter.<br>
**End condition:** line contains the character `>`.
5. **Start condition:** line begins with the string `<![CDATA[`.<br>
**End condition:** line contains the string `]]>`.
6. **Start condition:** line begins the string `<` or `</` followed by one of the strings (case-insensitive) `address`, `article`, `aside`, `base`, `basefont`, `blockquote`, `body`, `caption`, `center`, `col`, `colgroup`, `dd`, `details`, `dialog`, `dir`, `div`, `dl`, `dt`, `fieldset`, `figcaption`, `figure`, `footer`, `form`, `frame`, `frameset`, `h1`, `h2`, `h3`, `h4`, `h5`, `h6`, `head`, `header`, `hr`, `html`, `iframe`, `legend`, `li`, `link`, `main`, `menu`, `menuitem`, `meta`, `nav`, `noframes`, `ol`, `optgroup`, `option`, `p`, `param`, `section`, `source`, `summary`, `table`, `tbody`, `td`, `tfoot`, `th`, `thead`, `title`, `tr`, `track`, `ul`, followed by [whitespace](#whitespace), the end of the line, the string `>`, or the string `/>`.<br>
**End condition:** line is followed by a [blank line](#blank-line).
7. **Start condition:** line begins with a complete [open tag](#open-tag) or [closing tag](#closing-tag) (with any [tag name](#tag-name) other than `script`, `style`, or `pre`) followed only by [whitespace](#whitespace) or the end of the line.<br>
**End condition:** line is followed by a [blank line](#blank-line).

All types of [HTML blocks](#html-blocks) except type 7 may interrupt a paragraph. Blocks of type 7 may not interrupt a paragraph. (This restriction is intended to prevent unwanted interpretation of long tags inside a wrapped paragraph as starting HTML blocks.)

Some simple examples follow. Here are some basic HTML blocks of type 6:

[Example 115](#example-115)

```md
<table>
<tr>
<td>
hi
</td>
</tr>
</table>

okay.
```

```html
<table>
<tr>
<td>
hi
</td>
</tr>
</table>
<p>okay.</p>
```

[Example 116](#example-116)

```md
<div>
*hello*
<foo><a>
```

```html
<div>
*hello*
<foo><a>
```

A block can also start with a closing tag:

[Example 117](#example-117)

```md
</div>
*foo*
```

```html
</div>
*foo*
```

Here we have two HTML blocks with a Markdown paragraph between them:

[Example 118](#example-118)

```md
<DIVCLASS="foo">

*Markdown*

</DIV>
```

```html
<DIVCLASS="foo">
<p><em>Markdown</em></p>
</DIV>
```

The tag on the first line can be partial, as long as it is split where there would be whitespace:

[Example 119](#example-119)

```md
<divid="foo"
class="bar">
</div>
```

```html
<divid="foo"
class="bar">
</div>
```

[Example 120](#example-120)

```md
<divid="foo"class="bar
baz">
</div>
```

```html
<divid="foo"class="bar
baz">
</div>
```

An open tag need not be closed:

[Example 121](#example-121)

```md
<div>
*foo*

*bar*
```

```html
<div>
*foo*
<p><em>bar</em></p>
```

A partial tag need not even be completed (garbage in, garbage out):

[Example 122](#example-122)

```md
<divid="foo"
*hi*
```

```html
<divid="foo"
*hi*
```

[Example 123](#example-123)

```md
<divclass
foo
```

```html
<divclass
foo
```

The initial tag doesn't even need to be a valid tag, as long as it starts like one:

[Example 124](#example-124)

```md
<div*???-&&&-<---
*foo*
```

```html
<div*???-&&&-<---
*foo*
```

In type 6 blocks, the initial tag need not be on a line by itself:

[Example 125](#example-125)

```md
<div><ahref="bar">*foo*</a></div>
```

```html
<div><ahref="bar">*foo*</a></div>
```

[Example 126](#example-126)

```md
<table><tr><td>
foo
</td></tr></table>
```

```html
<table><tr><td>
foo
</td></tr></table>
```

Everything until the next blank line or end of document gets included in the HTML block. So, in the following example, what looks like a Markdown code block is actually part of the HTML block, which continues until a blank line or the end of the document is reached:

[Example 127](#example-127)

```md
<div></div>
```c
intx=33;
```
```

```html
<div></div>
```c
intx=33;
```
```

To start an [HTML block](#html-block) with a tag that is _not_ in the list of block-level tags in (6), you must put the tag by itself on the first line (and it must be complete):

[Example 128](#example-128)

```md
<ahref="foo">
*bar*
</a>
```

```html
<ahref="foo">
*bar*
</a>
```

In type 7 blocks, the [tag name](#tag-name) can be anything:

[Example 129](#example-129)

```md
<Warning>
*bar*
</Warning>
```

```html
<Warning>
*bar*
</Warning>
```

[Example 130](#example-130)

```md
<iclass="foo">
*bar*
</i>
```

```html
<iclass="foo">
*bar*
</i>
```

[Example 131](#example-131)

```md
</ins>
*bar*
```

```html
</ins>
*bar*
```

These rules are designed to allow us to work with tags that can function as either block-level or inline-level tags. The `<del>` tag is a nice example. We can surround content with `<del>` tags in three different ways. In this case, we get a raw HTML block, because the `<del>` tag is on a line by itself:

[Example 132](#example-132)

```md
<del>
*foo*
</del>
```

```html
<del>
*foo*
</del>
```

In this case, we get a raw HTML block that just includes the `<del>` tag (because it ends with the following blank line). So the contents get interpreted as CommonMark:

[Example 133](#example-133)

```md
<del>

*foo*

</del>
```

```html
<del>
<p><em>foo</em></p>
</del>
```

Finally, in this case, the `<del>` tags are interpreted as [raw HTML](#raw-html) _inside_ the CommonMark paragraph. (Because the tag is not on a line by itself, we get inline HTML rather than an [HTML block](#html-block).)

[Example 134](#example-134)

```md
<del>*foo*</del>
```

```html
<p><del><em>foo</em></del></p>
```

HTML tags designed to contain literal content (`script`, `style`, `pre`), comments, processing instructions, and declarations are treated somewhat differently. Instead of ending at the first blank line, these blocks end at the first line containing a corresponding end tag. As a result, these blocks can contain blank lines:

A pre tag (type 1):

[Example 135](#example-135)

```md
<prelanguage="haskell"><code>
importText.HTML.TagSoup

main::IO()
main=print$parseTagstags
</code></pre>
okay
```

```html
<prelanguage="haskell"><code>
importText.HTML.TagSoup

main::IO()
main=print$parseTagstags
</code></pre>
<p>okay</p>
```

A script tag (type 1):

[Example 136](#example-136)

```md
<scripttype="text/javascript">
//JavaScriptexample

document.getElementById("demo").innerHTML="HelloJavaScript!";
</script>
okay
```

```html
<scripttype="text/javascript">
//JavaScriptexample

document.getElementById("demo").innerHTML="HelloJavaScript!";
</script>
<p>okay</p>
```

A style tag (type 1):

[Example 137](#example-137)

```md
<style
type="text/css">
h1{color:red;}

p{color:blue;}
</style>
okay
```

```html
<style
type="text/css">
h1{color:red;}

p{color:blue;}
</style>
<p>okay</p>
```

If there is no matching end tag, the block will end at the end of the document (or the enclosing [block quote](#block-quotes) or [list item](#list-items)):

[Example 138](#example-138)

```md
<style
type="text/css">

foo
```

```html
<style
type="text/css">

foo
```

[Example 139](#example-139)

```md
><div>
>foo

bar
```

```html
<blockquote>
<div>
foo
</blockquote>
<p>bar</p>
```

[Example 140](#example-140)

```md
-<div>
-foo
```

```html
<ul>
<li>
<div>
</li>
<li>foo</li>
</ul>
```

The end tag can occur on the same line as the start tag:

[Example 141](#example-141)

```md
<style>p{color:red;}</style>
*foo*
```

```html
<style>p{color:red;}</style>
<p><em>foo</em></p>
```

[Example 142](#example-142)

```md
<!--foo-->*bar*
*baz*
```

```html
<!--foo-->*bar*
<p><em>baz</em></p>
```

Note that anything on the last line after the end tag will be included in the [HTML block](#html-block):

[Example 143](#example-143)

```md
<script>
foo
</script>1.*bar*
```

```html
<script>
foo
</script>1.*bar*
```

A comment (type 2):

[Example 144](#example-144)

```md
<!--Foo

bar
baz-->
okay
```

```html
<!--Foo

bar
baz-->
<p>okay</p>
```

A processing instruction (type 3):

[Example 145](#example-145)

```md
<?php

echo'>';

?>
okay
```

```html
<?php

echo'>';

?>
<p>okay</p>
```

A declaration (type 4):

[Example 146](#example-146)

```md
<!DOCTYPEhtml>
```

```html
<!DOCTYPEhtml>
```

CDATA (type 5):

[Example 147](#example-147)

```md
<![CDATA[
functionmatchwo(a,b)
{
if(a<b&&a<0)then{
return1;

}else{

return0;
}
}
]]>
okay
```

```html
<![CDATA[
functionmatchwo(a,b)
{
if(a<b&&a<0)then{
return1;

}else{

return0;
}
}
]]>
<p>okay</p>
```

The opening tag can be indented 1-3 spaces, but not 4:

[Example 148](#example-148)

```md
<!--foo-->

<!--foo-->gt;
```

```html
<!--foo-->
<pre><code>&lt;!--foo--&gt;
</code></pre>
```

[Example 149](#example-149)

```md
<div>

<div>
```

```html
<div>
<pre><code>&lt;div&gt;
</code></pre>
```

An HTML block of types 1–6 can interrupt a paragraph, and need not be preceded by a blank line.

[Example 150](#example-150)

```md
Foo
<div>
bar
</div>
```

```html
<p>Foo</p>
<div>
bar
</div>
```

However, a following blank line is needed, except at the end of a document, and except for blocks of types 1–5, above:

[Example 151](#example-151)

```md
<div>
bar
</div>
*foo*
```

```html
<div>
bar
</div>
*foo*
```

HTML blocks of type 7 cannot interrupt a paragraph:

[Example 152](#example-152)

```md
Foo
<ahref="bar">
baz
```

```html
<p>Foo
<ahref="bar">
baz</p>
```

This rule differs from John Gruber's original Markdown syntax specification, which says:

> The only restrictions are that block-level HTML elements — e.g. `<div>`, `<table>`, `<pre>`, `<p>`, etc. — must be separated from surrounding content by blank lines, and the start and end tags of the block should not be indented with tabs or spaces.

In some ways Gruber's rule is more restrictive than the one given here:

* It requires that an HTML block be preceded by a blank line.
* It does not allow the start tag to be indented.
* It requires a matching end tag, which it also does not allow to be indented.

Most Markdown implementations (including some of Gruber's own) do not respect all of these restrictions.

There is one respect, however, in which Gruber's rule is more liberal than the one given here, since it allows blank lines to occur inside an HTML block. There are two reasons for disallowing them here. First, it removes the need to parse balanced tags, which is expensive and can require backtracking from the end of the document if no matching end tag is found. Second, it provides a very simple and flexible way of including Markdown content inside HTML tags: simply separate the Markdown from the HTML using blank lines:

Compare:

[Example 153](#example-153)

```md
<div>

*Emphasized*text.

</div>
```

```html
<div>
<p><em>Emphasized</em>text.</p>
</div>
```

[Example 154](#example-154)

```md
<div>
*Emphasized*text.
</div>
```

```html
<div>
*Emphasized*text.
</div>
```

Some Markdown implementations have adopted a convention of interpreting content inside tags as text if the open tag has the attribute `markdown=1`. The rule given above seems a simpler and more elegant way of achieving the same expressive power, which is also much simpler to parse.

The main potential drawback is that one can no longer paste HTML blocks into Markdown documents with 100% reliability. However, _in most cases_ this will work fine, because the blank lines in HTML are usually followed by HTML block tags. For example:

[Example 155](#example-155)

```md
<table>

<tr>

<td>
Hi
</td>

</tr>

</table>
```

```html
<table>
<tr>
<td>
Hi
</td>
</tr>
</table>
```

There are problems, however, if the inner tags are indented _and_ separated by spaces, as then they will be interpreted as an indented code block:

[Example 156](#example-156)

```md
<table>

<tr>

<td>
Hi
</td>

</tr>

</table>
```

```html
<table>
<tr>
<pre><code>&lt;td&gt;
Hi
&lt;/td&gt;
</code></pre>
</tr>
</table>
```

Fortunately, blank lines are usually not necessary and can be deleted. The exception is inside `<pre>` tags, but as described above, raw HTML blocks starting with `<pre>` _can_ contain blank lines.

## 4.7  Link reference definitions

A [link reference definition](#link-reference-definition) consists of a [link label](#link-label), indented up to three spaces, followed by a colon (`:`), optional [whitespace](#whitespace) (including up to one [line ending](#line-ending)), a [link destination](#link-destination), optional [whitespace](#whitespace) (including up to one [line ending](#line-ending)), and an optional [link title](#link-title), which if it is present must be separated from the [link destination](#link-destination) by [whitespace](#whitespace). No further [non-whitespace characters](#non-whitespace-character) may occur on the line.

A [link reference definition](#link-reference-definition) does not correspond to a structural element of a document. Instead, it defines a label which can be used in [reference links](#reference-link) and reference-style [images](#images) elsewhere in the document. [Link reference definitions](#link-reference-definition) can come either before or after the links that use them.

[Example 157](#example-157)

```md
[foo]:/url"title"

[foo]
```

```html
<p><ahref="/url"title="title">foo</a></p>
```

[Example 158](#example-158)

```md
[foo]:
/url
'thetitle'

[foo]
```

```html
<p><ahref="/url"title="thetitle">foo</a></p>
```

[Example 159](#example-159)

```md
[Foo*bar\]]:my_(url)'title(withparens)'

[Foo*bar\]]
```

```html
<p><ahref="my_(url)"title="title(withparens)">Foo*bar]</a></p>
```

[Example 160](#example-160)

```md
[Foobar]:
<my%20url>
'title'

[Foobar]
```

```html
<p><ahref="my%20url"title="title">Foobar</a></p>
```

The title may extend over multiple lines:

[Example 161](#example-161)

```md
[foo]:/url'
title
line1
line2
'

[foo]
```

```html
<p><ahref="/url"title="
title
line1
line2
">foo</a></p>
```

However, it may not contain a [blank line](#blank-line):

[Example 162](#example-162)

```md
[foo]:/url'title

withblankline'

[foo]
```

```html
<p>[foo]:/url'title</p>
<p>withblankline'</p>
<p>[foo]</p>
```

The title may be omitted:

[Example 163](#example-163)

```md
[foo]:
/url

[foo]
```

```html
<p><ahref="/url">foo</a></p>
```

The link destination may not be omitted:

[Example 164](#example-164)

```md
[foo]:

[foo]
```

```html
<p>[foo]:</p>
<p>[foo]</p>
```

Both title and destination can contain backslash escapes and literal backslashes:

[Example 165](#example-165)

```md
[foo]:/url\bar\*baz"foo\"bar\baz"

[foo]
```

```html
<p><ahref="/url%5Cbar*baz"title="foo&quot;bar\baz">foo</a></p>
```

A link can come before its corresponding definition:

[Example 166](#example-166)

```md
[foo]

[foo]:url
```

```html
<p><ahref="url">foo</a></p>
```

If there are several matching definitions, the first one takes precedence:

[Example 167](#example-167)

```md
[foo]

[foo]:first
[foo]:second
```

```html
<p><ahref="first">foo</a></p>
```

As noted in the section on [Links](#links), matching of labels is case-insensitive (see [matches](#matches)).

[Example 168](#example-168)

```md
[FOO]:/url

[Foo]
```

```html
<p><ahref="/url">Foo</a></p>
```

[Example 169](#example-169)

```md
[ΑΓΩ]:/φου

[αγω]
```

```html
<p><ahref="/%CF%86%CE%BF%CF%85">αγω</a></p>
```

Here is a link reference definition with no corresponding link. It contributes nothing to the document.

[Example 170](#example-170)

```md
[foo]:/url
```

Here is another one:

[Example 171](#example-171)

```md
[
foo
]:/url
bar
```

```html
<p>bar</p>
```

This is not a link reference definition, because there are [non-whitespace characters](#non-whitespace-character) after the title:

[Example 172](#example-172)

```md
[foo]:/url"title"ok
```

```html
<p>[foo]:/url&quot;title&quot;ok</p>
```

This is a link reference definition, but it has no title:

[Example 173](#example-173)

```md
[foo]:/url
"title"ok
```

```html
<p>&quot;title&quot;ok</p>
```

This is not a link reference definition, because it is indented four spaces:

[Example 174](#example-174)

```md
[foo]:/url"title"

[foo]
```

```html
<pre><code>[foo]:/url&quot;title&quot;
</code></pre>
<p>[foo]</p>
```

This is not a link reference definition, because it occurs inside a code block:

[Example 175](#example-175)

```md
```
[foo]:/url
```

[foo]
```

```html
<pre><code>[foo]:/url
</code></pre>
<p>[foo]</p>
```

A [link reference definition](#link-reference-definition) cannot interrupt a paragraph.

[Example 176](#example-176)

```md
Foo
[bar]:/baz

[bar]
```

```html
<p>Foo
[bar]:/baz</p>
<p>[bar]</p>
```

However, it can directly follow other block elements, such as headings and thematic breaks, and it need not be followed by a blank line.

[Example 177](#example-177)

```md
#[Foo]
[foo]:/url
>bar
```

```html
<h1><ahref="/url">Foo</a></h1>
<blockquote>
<p>bar</p>
</blockquote>
```

Several [link reference definitions](#link-reference-definition) can occur one after another, without intervening blank lines.

[Example 178](#example-178)

```md
[foo]:/foo-url"foo"
[bar]:/bar-url
"bar"
[baz]:/baz-url

[foo],
[bar],
[baz]
```

```html
<p><ahref="/foo-url"title="foo">foo</a>,
<ahref="/bar-url"title="bar">bar</a>,
<ahref="/baz-url">baz</a></p>
```

[Link reference definitions](#link-reference-definition) can occur inside block containers, like lists and block quotations. They affect the entire document, not just the container in which they are defined:

[Example 179](#example-179)

```md
[foo]

>[foo]:/url
```

```html
<p><ahref="/url">foo</a></p>
<blockquote>
</blockquote>
```

## 4.8  Paragraphs

A sequence of non-blank lines that cannot be interpreted as other kinds of blocks forms a [paragraph](#paragraph). The contents of the paragraph are the result of parsing the paragraph's raw content as inlines. The paragraph's raw content is formed by concatenating the lines and removing initial and final [whitespace](#whitespace).

A simple example with two paragraphs:

[Example 180](#example-180)

```md
aaa

bbb
```

```html
<p>aaa</p>
<p>bbb</p>
```

Paragraphs can contain multiple lines, but no blank lines:

[Example 181](#example-181)

```md
aaa
bbb

ccc
ddd
```

```html
<p>aaa
bbb</p>
<p>ccc
ddd</p>
```

Multiple blank lines between paragraph have no effect:

[Example 182](#example-182)

```md
aaa

bbb
```

```html
<p>aaa</p>
<p>bbb</p>
```

Leading spaces are skipped:

[Example 183](#example-183)

```md
aaa
bbb
```

```html
<p>aaa
bbb</p>
```

Lines after the first may be indented any amount, since indented code blocks cannot interrupt paragraphs.

[Example 184](#example-184)

```md
aaa
bbb
ccc
```

```html
<p>aaa
bbb
ccc</p>
```

However, the first line may be indented at most three spaces, or an indented code block will be triggered:

[Example 185](#example-185)

```md
aaa
bbb
```

```html
<p>aaa
bbb</p>
```

[Example 186](#example-186)

```md
aaa
bbb
```

```html
<pre><code>aaa
</code></pre>
<p>bbb</p>
```

Final spaces are stripped before inline parsing, so a paragraph that ends with two or more spaces will not end with a [hard line break](#hard-line-break):

[Example 187](#example-187)

```md
aaa
bbb
```

```html
<p>aaa<br/>
bbb</p>
```

## 4.9  Blank lines

[Blank lines](#blank-lines) between block-level elements are ignored, except for the role they play in determining whether a [list](#list) is [tight](#tight) or [loose](#loose).

Blank lines at the beginning and end of the document are also ignored.

[Example 188](#example-188)

```md

aaa

#aaa

```

```html
<p>aaa</p>
<h1>aaa</h1>
```

# 5  Container blocks

A [container block](#container-block) is a block that has other blocks as its contents. There are two basic kinds of container blocks: [block quotes](#block-quotes) and [list items](#list-items). [Lists](#lists) are meta-containers for [list items](#list-items).

We define the syntax for container blocks recursively. The general form of the definition is:

> If X is a sequence of blocks, then the result of transforming X in such-and-such a way is a container of type Y with these blocks as its content.

So, we explain what counts as a block quote or list item by explaining how these can be _generated_ from their contents. This should suffice to define the syntax, although it does not give a recipe for _parsing_ these constructions. (A recipe is provided below in the section entitled [A parsing strategy](#appendix-a-parsing-strategy).)

## 5.1  Block quotes

A [block quote marker](#block-quote-marker) consists of 0-3 spaces of initial indent, plus (a) the character `>` together with a following space, or (b) a single character `>` not followed by a space.

The following rules define [block quotes](#block-quotes):

1. **Basic case.** If a string of lines _Ls_ constitute a sequence of blocks _Bs_, then the result of prepending a [block quote marker](#block-quote-marker) to the beginning of each line in _Ls_ is a [block quote](#block-quotes) containing _Bs_.
2. **Laziness.** If a string of lines _Ls_ constitute a [block quote](#block-quotes) with contents _Bs_, then the result of deleting the initial [block quote marker](#block-quote-marker) from one or more lines in which the next [non-whitespace character](#non-whitespace-character) after the [block quote marker](#block-quote-marker) is [paragraph continuation text](#paragraph-continuation-text) is a block quote with _Bs_ as its content. [Paragraph continuation text](#paragraph-continuation-text) is text that will be parsed as part of the content of a paragraph, but does not occur at the beginning of the paragraph.
3. **Consecutiveness.** A document cannot contain two [block quotes](#block-quotes) in a row unless there is a [blank line](#blank-line) between them.

Nothing else counts as a [block quote](#block-quotes).

Here is a simple example:

[Example 189](#example-189)

```md
>#Foo
>bar
>baz
```

```html
<blockquote>
<h1>Foo</h1>
<p>bar
baz</p>
</blockquote>
```

The spaces after the `>` characters can be omitted:

[Example 190](#example-190)

```md
>#Foo
>bar
>baz
```

```html
<blockquote>
<h1>Foo</h1>
<p>bar
baz</p>
</blockquote>
```

The `>` characters can be indented 1-3 spaces:

[Example 191](#example-191)

```md
>#Foo
>bar
>baz
```

```html
<blockquote>
<h1>Foo</h1>
<p>bar
baz</p>
</blockquote>
```

Four spaces gives us a code block:

[Example 192](#example-192)

```md
>#Foo
>bar
>baz
```

```html
<pre><code>&gt;#Foo
&gt;bar
&gt;baz
</code></pre>
```

The Laziness clause allows us to omit the `>` before [paragraph continuation text](#paragraph-continuation-text):

[Example 193](#example-193)

```md
>#Foo
>bar
baz
```

```html
<blockquote>
<h1>Foo</h1>
<p>bar
baz</p>
</blockquote>
```

A block quote can contain some lazy and some non-lazy continuation lines:

[Example 194](#example-194)

```md
>bar
baz
>foo
```

```html
<blockquote>
<p>bar
baz
foo</p>
</blockquote>
```

Laziness only applies to lines that would have been continuations of paragraphs had they been prepended with [block quote markers](#block-quote-marker). For example, the `>` cannot be omitted in the second line of

```md
> foo
> ---
```

without changing the meaning:

[Example 195](#example-195)

```md
>foo
---
```

```html
<blockquote>
<p>foo</p>
</blockquote>
<hr/>
```

Similarly, if we omit the `>` in the second line of

```md
> - foo
> - bar
```

then the block quote ends after the first line:

[Example 196](#example-196)

```md
>-foo
-bar
```

```html
<blockquote>
<ul>
<li>foo</li>
</ul>
</blockquote>
<ul>
<li>bar</li>
</ul>
```

For the same reason, we can't omit the `>` in front of subsequent lines of an indented or fenced code block:

[Example 197](#example-197)

```md
>foo
bar
```

```html
<blockquote>
<pre><code>foo
</code></pre>
</blockquote>
<pre><code>bar
</code></pre>
```

[Example 198](#example-198)

```md
>```
foo
```
```

```html
<blockquote>
<pre><code></code></pre>
</blockquote>
<p>foo</p>
<pre><code></code></pre>
```

Note that in the following case, we have a [lazy continuation line](#lazy-continuation-line):

[Example 199](#example-199)

```md
>foo
-bar
```

```html
<blockquote>
<p>foo
-bar</p>
</blockquote>
```

To see why, note that in

```md
> foo
>     - bar
```

the `- bar` is indented too far to start a list, and can't be an indented code block because indented code blocks cannot interrupt paragraphs, so it is [paragraph continuation text](#paragraph-continuation-text).

A block quote can be empty:

[Example 200](#example-200)

```md
>
```

```html
<blockquote>
</blockquote>
```

[Example 201](#example-201)

```md
>
>
>
```

```html
<blockquote>
</blockquote>
```

A block quote can have initial or final blank lines:

[Example 202](#example-202)

```md
>
>foo
>
```

```html
<blockquote>
<p>foo</p>
</blockquote>
```

A blank line always separates block quotes:

[Example 203](#example-203)

```md
>foo

>bar
```

```html
<blockquote>
<p>foo</p>
</blockquote>
<blockquote>
<p>bar</p>
</blockquote>
```

(Most current Markdown implementations, including John Gruber's original `Markdown.pl`, will parse this example as a single block quote with two paragraphs. But it seems better to allow the author to decide whether two block quotes or one are wanted.)

Consecutiveness means that if we put these block quotes together, we get a single block quote:

[Example 204](#example-204)

```md
>foo
>bar
```

```html
<blockquote>
<p>foo
bar</p>
</blockquote>
```

To get a block quote with two paragraphs, use:

[Example 205](#example-205)

```md
>foo
>
>bar
```

```html
<blockquote>
<p>foo</p>
<p>bar</p>
</blockquote>
```

Block quotes can interrupt paragraphs:

[Example 206](#example-206)

```md
foo
>bar
```

```html
<p>foo</p>
<blockquote>
<p>bar</p>
</blockquote>
```

In general, blank lines are not needed before or after block quotes:

[Example 207](#example-207)

```md
>aaa
***
>bbb
```

```html
<blockquote>
<p>aaa</p>
</blockquote>
<hr/>
<blockquote>
<p>bbb</p>
</blockquote>
```

However, because of laziness, a blank line is needed between a block quote and a following paragraph:

[Example 208](#example-208)

```md
>bar
baz
```

```html
<blockquote>
<p>bar
baz</p>
</blockquote>
```

[Example 209](#example-209)

```md
>bar

baz
```

```html
<blockquote>
<p>bar</p>
</blockquote>
<p>baz</p>
```

[Example 210](#example-210)

```md
>bar
>
baz
```

```html
<blockquote>
<p>bar</p>
</blockquote>
<p>baz</p>
```

It is a consequence of the Laziness rule that any number of initial `>`s may be omitted on a continuation line of a nested block quote:

[Example 211](#example-211)

```md
>>>foo
bar
```

```html
<blockquote>
<blockquote>
<blockquote>
<p>foo
bar</p>
</blockquote>
</blockquote>
</blockquote>
```

[Example 212](#example-212)

```md
>>>foo
>bar
>>baz
```

```html
<blockquote>
<blockquote>
<blockquote>
<p>foo
bar
baz</p>
</blockquote>
</blockquote>
</blockquote>
```

When including an indented code block in a block quote, remember that the [block quote marker](#block-quote-marker) includes both the `>` and a following space. So _five spaces_ are needed after the `>`:

[Example 213](#example-213)

```md
>code

>notcode
```

```html
<blockquote>
<pre><code>code
</code></pre>
</blockquote>
<blockquote>
<p>notcode</p>
</blockquote>
```

## 5.2  List items

A [list marker](#list-marker) is a [bullet list marker](#bullet-list-marker) or an [ordered list marker](#ordered-list-marker).

A [bullet list marker](#bullet-list-marker) is a `-`, `+`, or `*` character.

An [ordered list marker](#ordered-list-marker) is a sequence of 1–9 arabic digits (`0-9`), followed by either a `.` character or a `)` character. (The reason for the length limit is that with 10 digits we start seeing integer overflows in some browsers.)

The following rules define [list items](#list-items):

1. **Basic case.** If a sequence of lines _Ls_ constitute a sequence of blocks _Bs_ starting with a [non-whitespace character](#non-whitespace-character) and not separated from each other by more than one blank line, and _M_ is a list marker of width _W_ followed by 1 ≤ _N_ ≤ 4 spaces, then the result of prepending _M_ and the following spaces to the first line of _Ls_, and indenting subsequent lines of _Ls_ by _W + N_ spaces, is a list item with _Bs_ as its contents. The type of the list item (bullet or ordered) is determined by the type of its list marker. If the list item is ordered, then it is also assigned a start number, based on the ordered list marker. Exceptions: When the first list item in a [list](#list) interrupts a paragraph—that is, when it starts on a line that would otherwise count as [paragraph continuation text](#paragraph-continuation-text)—then (a) the lines _Ls_ must not begin with a blank line, and (b) if the list item is ordered, the start number must be

For example, let _Ls_ be the lines

[Example 214](#example-214)

```md
Aparagraph
withtwolines.

indentedcode

>Ablockquote.
```

```html
<p>Aparagraph
withtwolines.</p>
<pre><code>indentedcode
</code></pre>
<blockquote>
<p>Ablockquote.</p>
</blockquote>
```

And let _M_ be the marker `1.`, and _N_ = 2. Then rule #1 says that the following is an ordered list item with start number 1, and the same contents as _Ls_:

[Example 215](#example-215)

```md
1.Aparagraph
withtwolines.

indentedcode

>Ablockquote.
```

```html
<ol>
<li>
<p>Aparagraph
withtwolines.</p>
<pre><code>indentedcode
</code></pre>
<blockquote>
<p>Ablockquote.</p>
</blockquote>
</li>
</ol>
```

The most important thing to notice is that the position of the text after the list marker determines how much indentation is needed in subsequent blocks in the list item. If the list marker takes up two spaces, and there are three spaces between the list marker and the next [non-whitespace character](#non-whitespace-character), then blocks must be indented five spaces in order to fall under the list item.

Here are some examples showing how far content must be indented to be put under the list item:

[Example 216](#example-216)

```md
-one

two
```

```html
<ul>
<li>one</li>
</ul>
<p>two</p>
```

[Example 217](#example-217)

```md
-one

two
```

```html
<ul>
<li>
<p>one</p>
<p>two</p>
</li>
</ul>
```

[Example 218](#example-218)

```md
-one

two
```

```html
<ul>
<li>one</li>
</ul>
<pre><code>two
</code></pre>
```

[Example 219](#example-219)

```md
-one

two
```

```html
<ul>
<li>
<p>one</p>
<p>two</p>
</li>
</ul>
```

It is tempting to think of this in terms of columns: the continuation blocks must be indented at least to the column of the first [non-whitespace character](#non-whitespace-character) after the list marker. However, that is not quite right. The spaces after the list marker determine how much relative indentation is needed. Which column this indentation reaches will depend on how the list item is embedded in other constructions, as shown by this example:

[Example 220](#example-220)

```md
>>1.one
>>
>>two
```

```html
<blockquote>
<blockquote>
<ol>
<li>
<p>one</p>
<p>two</p>
</li>
</ol>
</blockquote>
</blockquote>
```

Here `two` occurs in the same column as the list marker `1.`, but is actually contained in the list item, because there is sufficient indentation after the last containing blockquote marker.

The converse is also possible. In the following example, the word `two` occurs far to the right of the initial text of the list item, `one`, but it is not considered part of the list item, because it is not indented far enough past the blockquote marker:

[Example 221](#example-221)

```md
>>-one
>>
>>two
```

```html
<blockquote>
<blockquote>
<ul>
<li>one</li>
</ul>
<p>two</p>
</blockquote>
</blockquote>
```

Note that at least one space is needed between the list marker and any following content, so these are not list items:

[Example 222](#example-222)

```md
-one

2.two
```

```html
<p>-one</p>
<p>2.two</p>
```

A list item may contain blocks that are separated by more than one blank line.

[Example 223](#example-223)

```md
-foo

bar
```

```html
<ul>
<li>
<p>foo</p>
<p>bar</p>
</li>
</ul>
```

A list item may contain any kind of block:

[Example 224](#example-224)

```md
1.foo

```
bar
```

baz

>bam
```

```html
<ol>
<li>
<p>foo</p>
<pre><code>bar
</code></pre>
<p>baz</p>
<blockquote>
<p>bam</p>
</blockquote>
</li>
</ol>
```

A list item that contains an indented code block will preserve empty lines within the code block verbatim.

[Example 225](#example-225)

```md
-Foo

bar

baz
```

```html
<ul>
<li>
<p>Foo</p>
<pre><code>bar

baz
</code></pre>
</li>
</ul>
```

Note that ordered list start numbers must be nine digits or less:

[Example 226](#example-226)

```md
123456789.ok
```

```html
<olstart="quot;123456789">
<li>ok</li>
</ol>
```

[Example 227](#example-227)

```md
1234567890.notok
```

```html
<p>1234567890.notok</p>
```

A start number may begin with 0s:

[Example 228](#example-228)

```md
0.ok
```

```html
<olstart="0">
<li>ok</li>
</ol>
```

[Example 229](#example-229)

```md
003.ok
```

```html
<olstart="3">
<li>ok</li>
</ol>
```

A start number may not be negative:

[Example 230](#example-230)

```md
-1.notok
```

```html
<p>-1.notok</p>
```

1. **Item starting with indented code.** If a sequence of lines _Ls_ constitute a sequence of blocks _Bs_ starting with an indented code block and not separated from each other by more than one blank line, and _M_ is a list marker of width _W_ followed by one space, then the result of prepending _M_ and the following space to the first line of _Ls_, and indenting subsequent lines of _Ls_ by _W + 1_ spaces, is a list item with _Bs_ as its contents. If a line is empty, then it need not be indented. The type of the list item (bullet or ordered) is determined by the type of its list marker. If the list item is ordered, then it is also assigned a start number, based on the ordered list marker.

An indented code block will have to be indented four spaces beyond the edge of the region where text will be included in the list item. In the following case that is 6 spaces:

[Example 231](#example-231)

```md
-foo

bar
```

```html
<ul>
<li>
<p>foo</p>
<pre><code>bar
</code></pre>
</li>
</ul>
```

And in this case it is 11 spaces:

[Example 232](#example-232)

```md
10.foo

bar
```

```html
<olstart="10">
<li>
<p>foo</p>
<pre><code>bar
</code></pre>
</li>
</ol>
```

If the _first_ block in the list item is an indented code block, then by rule #2, the contents must be indented _one_ space after the list marker:

[Example 233](#example-233)

```md
indentedcode

paragraph

morecode
```

```html
<pre><code>indentedcode
</code></pre>
<p>paragraph</p>
<pre><code>morecode
</code></pre>
```

[Example 234](#example-234)

```md
1.indentedcode

paragraph

morecode
```

```html
<ol>
<li>
<pre><code>indentedcode
</code></pre>
<p>paragraph</p>
<pre><code>morecode
</code></pre>
</li>
</ol>
```

Note that an additional space indent is interpreted as space inside the code block:

[Example 235](#example-235)

```md
1.indentedcode

paragraph

morecode
```

```html
<ol>
<li>
<pre><code>indentedcode
</code></pre>
<p>paragraph</p>
<pre><code>morecode
</code></pre>
</li>
</ol>
```

Note that rules #1 and #2 only apply to two cases: (a) cases in which the lines to be included in a list item begin with a [non-whitespace character](#non-whitespace-character), and (b) cases in which they begin with an indented code block. In a case like the following, where the first block begins with a three-space indent, the rules do not allow us to form a list item by indenting the whole thing and prepending a list marker:

[Example 236](#example-236)

```md
foo

bar
```

```html
<p>foo</p>
<p>bar</p>
```

[Example 237](#example-237)

```md
-foo

bar
```

```html
<ul>
<li>foo</li>
</ul>
<p>bar</p>
```

This is not a significant restriction, because when a block begins with 1-3 spaces indent, the indentation can always be removed without a change in interpretation, allowing rule #1 to be applied. So, in the above case:

[Example 238](#example-238)

```md
-foo

bar
```

```html
<ul>
<li>
<p>foo</p>
<p>bar</p>
</li>
</ul>
```

1. **Item starting with a blank line.** If a sequence of lines _Ls_ starting with a single [blank line](#blank-line) constitute a (possibly empty) sequence of blocks _Bs_, not separated from each other by more than one blank line, and _M_ is a list marker of width _W_, then the result of prepending _M_ to the first line of _Ls_, and indenting subsequent lines of _Ls_ by _W + 1_ spaces, is a list item with _Bs_ as its contents. If a line is empty, then it need not be indented. The type of the list item (bullet or ordered) is determined by the type of its list marker. If the list item is ordered, then it is also assigned a start number, based on the ordered list marker.

Here are some list items that start with a blank line but are not empty:

[Example 239](#example-239)

```md
-
foo
-
```
bar
```
-
baz
```

```html
<ul>
<li>foo</li>
<li>
<pre><code>bar
</code></pre>
</li>
<li>
<pre><code>baz
</code></pre>
</li>
</ul>
```

When the list item starts with a blank line, the number of spaces following the list marker doesn't change the required indentation:

[Example 240](#example-240)

```md
-
foo
```

```html
<ul>
<li>foo</li>
</ul>
```

A list item can begin with at most one blank line. In the following example, `foo` is not part of the list item:

[Example 241](#example-241)

```md
-

foo
```

```html
<ul>
<li></li>
</ul>
<p>foo</p>
```

Here is an empty bullet list item:

[Example 242](#example-242)

```md
-foo
-
-bar
```

```html
<ul>
<li>foo</li>
<li></li>
<li>bar</li>
</ul>
```

It does not matter whether there are spaces following the [list marker](#list-marker):

[Example 243](#example-243)

```md
-foo
-
-bar
```

```html
<ul>
<li>foo</li>
<li></li>
<li>bar</li>
</ul>
```

Here is an empty ordered list item:

[Example 244](#example-244)

```md
1.foo
2.
3.bar
```

```html
<ol>
<li>foo</li>
<li></li>
<li>bar</li>
</ol>
```

A list may start or end with an empty list item:

[Example 245](#example-245)

```md
*
```

```html
<ul>
<li></li>
</ul>
```

However, an empty list item cannot interrupt a paragraph:

[Example 246](#example-246)

```md
foo
*

foo
1.
```

```html
<p>foo
*</p>
<p>foo
1.</p>
```

1. **Indentation.** If a sequence of lines _Ls_ constitutes a list item according to rule #1, #2, or #3, then the result of indenting each line of _Ls_ by 1-3 spaces (the same for each line) also constitutes a list item with the same contents and attributes. If a line is empty, then it need not be indented.

Indented one space:

[Example 247](#example-247)

```md
1.Aparagraph
withtwolines.

indentedcode

>Ablockquote.
```

```html
<ol>
<li>
<p>Aparagraph
withtwolines.</p>
<pre><code>indentedcode
</code></pre>
<blockquote>
<p>Ablockquote.</p>
</blockquote>
</li>
</ol>
```

Indented two spaces:

[Example 248](#example-248)

```md
1.Aparagraph
withtwolines.

indentedcode

>Ablockquote.
```

```html
<ol>
<li>
<p>Aparagraph
withtwolines.</p>
<pre><code>indentedcode
</code></pre>
<blockquote>
<p>Ablockquote.</p>
</blockquote>
</li>
</ol>
```

Indented three spaces:

[Example 249](#example-249)

```md
1.Aparagraph
withtwolines.

indentedcode

>Ablockquote.
```

```html
<ol>
<li>
<p>Aparagraph
withtwolines.</p>
<pre><code>indentedcode
</code></pre>
<blockquote>
<p>Ablockquote.</p>
</blockquote>
</li>
</ol>
```

Four spaces indent gives a code block:

[Example 250](#example-250)

```md
1.Aparagraph
withtwolines.

indentedcode

>Ablockquote.
```

```html
<pre><code>1.Aparagraph
withtwolines.

indentedcode

&gt;Ablockquote.
</code></pre>
```

1. **Laziness.** If a string of lines _Ls_ constitute a [list item](#list-items) with contents _Bs_, then the result of deleting some or all of the indentation from one or more lines in which the next [non-whitespace character](#non-whitespace-character) after the indentation is [paragraph continuation text](#paragraph-continuation-text) is a list item with the same contents and attributes. The unindented lines are called [lazy continuation line](#lazy-continuation-line)s.

Here is an example with [lazy continuation lines](#lazy-continuation-line):

[Example 251](#example-251)

```md
1.Aparagraph
withtwolines.

indentedcode

>Ablockquote.
```

```html
<ol>
<li>
<p>Aparagraph
withtwolines.</p>
<pre><code>indentedcode
</code></pre>
<blockquote>
<p>Ablockquote.</p>
</blockquote>
</li>
</ol>
```

Indentation can be partially deleted:

[Example 252](#example-252)

```md
1.Aparagraph
withtwolines.
```

```html
<ol>
<li>Aparagraph
withtwolines.</li>
</ol>
```

These examples show how laziness can work in nested structures:

[Example 253](#example-253)

```md
>1.>Blockquote
continuedhere.
```

```html
<blockquote>
<ol>
<li>
<blockquote>
<p>Blockquote
continuedhere.</p>
</blockquote>
</li>
</ol>
</blockquote>
```

[Example 254](#example-254)

```md
>1.>Blockquote
>continuedhere.
```

```html
<blockquote>
<ol>
<li>
<blockquote>
<p>Blockquote
continuedhere.</p>
</blockquote>
</li>
</ol>
</blockquote>
```

1. **That's all.** Nothing that is not counted as a list item by rules #1–5 counts as a [list item](#list-items).

The rules for sublists follow from the general rules above. A sublist must be indented the same number of spaces a paragraph would need to be in order to be included in the list item.

So, in this case we need two spaces indent:

[Example 255](#example-255)

```md
-foo
-bar
-baz
-boo
```

```html
<ul>
<li>foo
<ul>
<li>bar
<ul>
<li>baz
<ul>
<li>boo</li>
</ul>
</li>
</ul>
</li>
</ul>
</li>
</ul>
```

One is not enough:

[Example 256](#example-256)

```md
-foo
-bar
-baz
-boo
```

```html
<ul>
<li>foo</li>
<li>bar</li>
<li>baz</li>
<li>boo</li>
</ul>
```

Here we need four, because the list marker is wider:

[Example 257](#example-257)

```md
10)foo
-bar
```

```html
<olstart="10">
<li>foo
<ul>
<li>bar</li>
</ul>
</li>
</ol>
```

Three is not enough:

[Example 258](#example-258)

```md
10)foo
-bar
```

```html
<olstart="10">
<li>foo</li>
</ol>
<ul>
<li>bar</li>
</ul>
```

A list may be the first block in a list item:

[Example 259](#example-259)

```md
--foo
```

```html
<ul>
<li>
<ul>
<li>foo</li>
</ul>
</li>
</ul>
```

[Example 260](#example-260)

```md
1.-2.foo
```

```html
<ol>
<li>
<ul>
<li>
<olstart="2">
<li>foo</li>
</ol>
</li>
</ul>
</li>
</ol>
```

A list item can contain a heading:

[Example 261](#example-261)

```md
-#Foo
-Bar
---
baz
```

```html
<ul>
<li>
<h1>Foo</h1>
</li>
<li>
<h2>Bar</h2>
baz</li>
</ul>
```

### 5.2.1  Motivation

John Gruber's Markdown spec says the following about list items:

1. "List markers typically start at the left margin, but may be indented by up to three spaces. List markers must be followed by one or more spaces or a tab."
2. "To make lists look nice, you can wrap items with hanging indents.... But if you don't want to, you don't have to."
3. "List items may consist of multiple paragraphs. Each subsequent paragraph in a list item must be indented by either 4 spaces or one tab."
4. "It looks nice if you indent every line of the subsequent paragraphs, but here again, Markdown will allow you to be lazy."
5. "To put a blockquote within a list item, the blockquote's `>` delimiters need to be indented."
6. "To put a code block within a list item, the code block needs to be indented twice — 8 spaces or two tabs."

These rules specify that a paragraph under a list item must be indented four spaces (presumably, from the left margin, rather than the start of the list marker, but this is not said), and that code under a list item must be indented eight spaces instead of the usual four. They also say that a block quote must be indented, but not by how much; however, the example given has four spaces indentation. Although nothing is said about other kinds of block-level content, it is certainly reasonable to infer that _all_ block elements under a list item, including other lists, must be indented four spaces. This principle has been called the _four-space rule_.

The four-space rule is clear and principled, and if the reference implementation `Markdown.pl` had followed it, it probably would have become the standard. However, `Markdown.pl` allowed paragraphs and sublists to start with only two spaces indentation, at least on the outer level. Worse, its behavior was inconsistent: a sublist of an outer-level list needed two spaces indentation, but a sublist of this sublist needed three spaces. It is not surprising, then, that different implementations of Markdown have developed very different rules for determining what comes under a list item. (Pandoc and python-Markdown, for example, stuck with Gruber's syntax description and the four-space rule, while discount, redcarpet, marked, PHP Markdown, and others followed `Markdown.pl`'s behavior more closely.)

Unfortunately, given the divergences between implementations, there is no way to give a spec for list items that will be guaranteed not to break any existing documents. However, the spec given here should correctly handle lists formatted with either the four-space rule or the more forgiving `Markdown.pl` behavior, provided they are laid out in a way that is natural for a human to read.

The strategy here is to let the width and indentation of the list marker determine the indentation necessary for blocks to fall under the list item, rather than having a fixed and arbitrary number. The writer can think of the body of the list item as a unit which gets indented to the right enough to fit the list marker (and any indentation on the list marker). (The laziness rule, #5, then allows continuation lines to be unindented if needed.)

This rule is superior, we claim, to any rule requiring a fixed level of indentation from the margin. The four-space rule is clear but unnatural. It is quite unintuitive that

```md
- foo

  bar

  - baz
```

should be parsed as two lists with an intervening paragraph,

```html
<ul>
<li>foo</li>
</ul>
<p>bar</p>
<ul>
<li>baz</li>
</ul>
```

as the four-space rule demands, rather than a single list,

```html
<ul>
<li>
<p>foo</p>
<p>bar</p>
<ul>
<li>baz</li>
</ul>
</li>
</ul>
```

The choice of four spaces is arbitrary. It can be learned, but it is not likely to be guessed, and it trips up beginners regularly.

Would it help to adopt a two-space rule? The problem is that such a rule, together with the rule allowing 1–3 spaces indentation of the initial list marker, allows text that is indented _less than_ the original list marker to be included in the list item. For example, `Markdown.pl` parses

```md
   - one

  two
```

as a single list item, with `two` a continuation paragraph:

```html
<ul>
<li>
<p>one</p>
<p>two</p>
</li>
</ul>
```

and similarly

```md
>   - one
>
>  two
```

as

```html
<blockquote>
<ul>
<li>
<p>one</p>
<p>two</p>
</li>
</ul>
</blockquote>
```

This is extremely unintuitive.

Rather than requiring a fixed indent from the margin, we could require a fixed indent (say, two spaces, or even one space) from the list marker (which may itself be indented). This proposal would remove the last anomaly discussed. Unlike the spec presented above, it would count the following as a list item with a subparagraph, even though the paragraph `bar` is not indented as far as the first paragraph `foo`:

```md
 10. foo

   bar
```

Arguably this text does read like a list item with `bar` as a subparagraph, which may count in favor of the proposal. However, on this proposal indented code would have to be indented six spaces after the list marker. And this would break a lot of existing Markdown, which has the pattern:

```md
1.  foo

        indented code
```

where the code is indented eight spaces. The spec above, by contrast, will parse this text as expected, since the code block's indentation is measured from the beginning of `foo`.

The one case that needs special treatment is a list item that _starts_ with indented code. How much indentation is required in that case, since we don't have a "first paragraph" to measure from? Rule #2 simply stipulates that in such cases, we require one space indentation from the list marker (and then the normal four spaces for the indented code). This will match the four-space rule in cases where the list marker plus its initial indentation takes four spaces (a common case), but diverge in other cases.

## 5.3  Lists

A [list](#list) is a sequence of one or more list items [of the same type](#of-the-same-type). The list items may be separated by any number of blank lines.

Two list items are [of the same type](#of-the-same-type) if they begin with a [list marker](#list-marker) of the same type. Two list markers are of the same type if (a) they are bullet list markers using the same character (`-`, `+`, or `*`) or (b) they are ordered list numbers with the same delimiter (either `.` or `)`).

A list is an [ordered list](#ordered-list) if its constituent list items begin with [ordered list markers](#ordered-list-marker), and a [bullet list](#bullet-list) if its constituent list items begin with [bullet list markers](#bullet-list-marker).

The [start number](#start-number) of an [ordered list](#ordered-list) is determined by the list number of its initial list item. The numbers of subsequent list items are disregarded.

A list is [loose](#loose) if any of its constituent list items are separated by blank lines, or if any of its constituent list items directly contain two block-level elements with a blank line between them. Otherwise a list is [tight](#tight). (The difference in HTML output is that paragraphs in a loose list are wrapped in `<p>` tags, while paragraphs in a tight list are not.)

Changing the bullet or ordered list delimiter starts a new list:

[Example 262](#example-262)

```md
-foo
-bar
+baz
```

```html
<ul>
<li>foo</li>
<li>bar</li>
</ul>
<ul>
<li>baz</li>
</ul>
```

[Example 263](#example-263)

```md
1.foo
2.bar
3)baz
```

```html
<ol>
<li>foo</li>
<li>bar</li>
</ol>
<olstart="3">
<li>baz</li>
</ol>
```

In CommonMark, a list can interrupt a paragraph. That is, no blank line is needed to separate a paragraph from a following list:

[Example 264](#example-264)

```md
Foo
-bar
-baz
```

```html
<p>Foo</p>
<ul>
<li>bar</li>
<li>baz</li>
</ul>
```

`Markdown.pl` does not allow this, through fear of triggering a list via a numeral in a hard-wrapped line:

```md
The number of windows in my house is
14.  The number of doors is 6.
```

Oddly, though, `Markdown.pl` _does_ allow a blockquote to interrupt a paragraph, even though the same considerations might apply.

In CommonMark, we do allow lists to interrupt paragraphs, for two reasons. First, it is natural and not uncommon for people to start lists without blank lines:

```md
I need to buy
- new shoes
- a coat
- a plane ticket
```

Second, we are attracted to a

> [principle of uniformity](#principle-of-uniformity): if a chunk of text has a certain meaning, it will continue to have the same meaning when put into a container block (such as a list item or blockquote).

(Indeed, the spec for [list items](#list-items) and [block quotes](#block-quotes) presupposes this principle.) This principle implies that if

```md
  * I need to buy
    - new shoes
    - a coat
    - a plane ticket
```

is a list item containing a paragraph followed by a nested sublist, as all Markdown implementations agree it is (though the paragraph may be rendered without `<p>` tags, since the list is "tight"), then

```md
I need to buy
- new shoes
- a coat
- a plane ticket
```

by itself should be a paragraph followed by a nested sublist.

Since it is well established Markdown practice to allow lists to interrupt paragraphs inside list items, the [principle of uniformity](#principle-of-uniformity) requires us to allow this outside list items as well. ([reStructuredText](http://docutils.sourceforge.net/rst.html) takes a different approach, requiring blank lines before lists even inside other list items.)

In order to solve of unwanted lists in paragraphs with hard-wrapped numerals, we allow only lists starting with `1` to interrupt paragraphs. Thus,

[Example 265](#example-265)

```md
Thenumberofwindowsinmyhouseis
14.Thenumberofdoorsis6.
```

```html
<p>Thenumberofwindowsinmyhouseis
14.Thenumberofdoorsis6.</p>
```

We may still get an unintended result in cases like

[Example 266](#example-266)

```md
Thenumberofwindowsinmyhouseis
1.Thenumberofdoorsis6.
```

```html
<p>Thenumberofwindowsinmyhouseis</p>
<ol>
<li>Thenumberofdoorsis6.</li>
</ol>
```

but this rule should prevent most spurious list captures.

There can be any number of blank lines between items:

[Example 267](#example-267)

```md
-foo

-bar

-baz
```

```html
<ul>
<li>
<p>foo</p>
</li>
<li>
<p>bar</p>
</li>
<li>
<p>baz</p>
</li>
</ul>
```

[Example 268](#example-268)

```md
-foo
-bar
-baz

bim
```

```html
<ul>
<li>foo
<ul>
<li>bar
<ul>
<li>
<p>baz</p>
<p>bim</p>
</li>
</ul>
</li>
</ul>
</li>
</ul>
```

To separate consecutive lists of the same type, or to separate a list from an indented code block that would otherwise be parsed as a subparagraph of the final list item, you can insert a blank HTML comment:

[Example 269](#example-269)

```md
-foo
-bar

<!---->

-baz
-bim
```

```html
<ul>
<li>foo</li>
<li>bar</li>
</ul>
<!---->
<ul>
<li>baz</li>
<li>bim</li>
</ul>
```

[Example 270](#example-270)

```md
-foo

notcode

-foo

<!---->

code
```

```html
<ul>
<li>
<p>foo</p>
<p>notcode</p>
</li>
<li>
<p>foo</p>
</li>
</ul>
<!---->
<pre><code>code
</code></pre>
```

List items need not be indented to the same level. The following list items will be treated as items at the same list level, since none is indented enough to belong to the previous list item:

[Example 271](#example-271)

```md
-a
-b
-c
-d
-e
-f
-g
-h
-i
```

```html
<ul>
<li>a</li>
<li>b</li>
<li>c</li>
<li>d</li>
<li>e</li>
<li>f</li>
<li>g</li>
<li>h</li>
<li>i</li>
</ul>
```

[Example 272](#example-272)

```md
1.a

2.b

3.c
```

```html
<ol>
<li>
<p>a</p>
</li>
<li>
<p>b</p>
</li>
<li>
<p>c</p>
</li>
</ol>
```

This is a loose list, because there is a blank line between two of the list items:

[Example 273](#example-273)

```md
-a
-b

-c
```

```html
<ul>
<li>
<p>a</p>
</li>
<li>
<p>b</p>
</li>
<li>
<p>c</p>
</li>
</ul>
```

So is this, with a empty second item:

[Example 274](#example-274)

```md
*a
*

*c
```

```html
<ul>
<li>
<p>a</p>
</li>
<li></li>
<li>
<p>c</p>
</li>
</ul>
```

These are loose lists, even though there is no space between the items, because one of the items directly contains two block-level elements with a blank line between them:

[Example 275](#example-275)

```md
-a
-b

c
-d
```

```html
<ul>
<li>
<p>a</p>
</li>
<li>
<p>b</p>
<p>c</p>
</li>
<li>
<p>d</p>
</li>
</ul>
```

[Example 276](#example-276)

```md
-a
-b

[ref]:/url
-d
```

```html
<ul>
<li>
<p>a</p>
</li>
<li>
<p>b</p>
</li>
<li>
<p>d</p>
</li>
</ul>
```

This is a tight list, because the blank lines are in a code block:

[Example 277](#example-277)

```md
-a
-```
b

```
-c
```

```html
<ul>
<li>a</li>
<li>
<pre><code>b

</code></pre>
</li>
<li>c</li>
</ul>
```

This is a tight list, because the blank line is between two paragraphs of a sublist. So the sublist is loose while the outer list is tight:

[Example 278](#example-278)

```md
-a
-b

c
-d
```

```html
<ul>
<li>a
<ul>
<li>
<p>b</p>
<p>c</p>
</li>
</ul>
</li>
<li>d</li>
</ul>
```

This is a tight list, because the blank line is inside the block quote:

[Example 279](#example-279)

```md
*a
>b
>
*c
```

```html
<ul>
<li>a
<blockquote>
<p>b</p>
</blockquote>
</li>
<li>c</li>
</ul>
```

This list is tight, because the consecutive block elements are not separated by blank lines:

[Example 280](#example-280)

```md
-a
>b
```
c
```
-d
```

```html
<ul>
<li>a
<blockquote>
<p>b</p>
</blockquote>
<pre><code>c
</code></pre>
</li>
<li>d</li>
</ul>
```

A single-paragraph list is tight:

[Example 281](#example-281)

```md
-a
```

```html
<ul>
<li>a</li>
</ul>
```

[Example 282](#example-282)

```md
-a
-b
```

```html
<ul>
<li>a
<ul>
<li>b</li>
</ul>
</li>
</ul>
```

This list is loose, because of the blank line between the two block elements in the list item:

[Example 283](#example-283)

```md
1.```
foo
```

bar
```

```html
<ol>
<li>
<pre><code>foo
</code></pre>
<p>bar</p>
</li>
</ol>
```

Here the outer list is loose, the inner list tight:

[Example 284](#example-284)

```md
*foo
*bar

baz
```

```html
<ul>
<li>
<p>foo</p>
<ul>
<li>bar</li>
</ul>
<p>baz</p>
</li>
</ul>
```

[Example 285](#example-285)

```md
-a
-b
-c

-d
-e
-f
```

```html
<ul>
<li>
<p>a</p>
<ul>
<li>b</li>
<li>c</li>
</ul>
</li>
<li>
<p>d</p>
<ul>
<li>e</li>
<li>f</li>
</ul>
</li>
</ul>
```

# 6  Inlines

Inlines are parsed sequentially from the beginning of the character stream to the end (left to right, in left-to-right languages). Thus, for example, in

[Example 286](#example-286)

```md
`hi`lo`
```

```html
<p><code>hi</code>lo`</p>
```

`hi` is parsed as code, leaving the backtick at the end as a literal backtick.

## 6.1  Backslash escapes

Any ASCII punctuation character may be backslash-escaped:

[Example 287](#example-287)

```md
\!\"\#\$\%\&\'\(\)\*\+\,\-\.\/\:\;\<\=\>\?\@\[\\\]\^\_\`\{\|\}\~
```

```html
<p>!&quot;#$%&amp;'()*+,-./:;&lt;=&gt;?@[\]^_`{|}~</p>
```

Backslashes before other characters are treated as literal backslashes:

[Example 288](#example-288)

```md
\→\A\a\\3\φ\<<
```

```html
<p>\→\A\a\\3\φ\<<</p>
```

Escaped characters are treated as regular characters and do not have their usual Markdown meanings:

[Example 289](#example-289)

```md
\*notemphasized*
\<br/>notatag
\[notalink](/foo)
\`notcode`
1\.notalist
\*notalist
\#notaheading
\[foo]:/url"notareference"
```

```html
<p>*notemphasized*
&lt;br/&gt;notatag
[notalink](/foo)
`notcode`
1.notalist
*notalist
#notaheading
[foo]:/url&quot;notareference&quot;</p>
```

If a backslash is itself escaped, the following character is not:

[Example 290](#example-290)

```md
\\*emphasis*
```

```html
<p>\<em>emphasis</em></p>
```

A backslash at the end of the line is a [hard line break](#hard-line-break):

[Example 291](#example-291)

```md
foo\
bar
```

```html
<p>foo<br/>
bar</p>
```

Backslash escapes do not work in code blocks, code spans, autolinks, or raw HTML:

[Example 292](#example-292)

```md
``\[\```
```

```html
<p><code>\[\`</code></p>
```

[Example 293](#example-293)

```md
\[\]
```

```html
<pre><code>\[\]
</code></pre>
```

[Example 294](#example-294)

```md
~~~
\[\]
~~~
```

```html
<pre><code>\[\]
</code></pre>
```

[Example 295](#example-295)

```md
<http://example.com?find=\*>
```

```html
<p><ahref="http://example.com?find=%5C*">http://example.com?find=\*</a></p>
```

[Example 296](#example-296)

```md
<ahref="/bar\/)">
```

```html
<ahref="/bar\/)">
```

But they work in all other contexts, including URLs and link titles, link references, and [info strings](#info-string) in [fenced code blocks](#fenced-code-block):

[Example 297](#example-297)

```md
[foo](/bar\*"ti\*tle")
```

```html
<p><ahref="/bar*"title="ti*tle">foo</a></p>
```

[Example 298](#example-298)

```md
[foo]

[foo]:/bar\*"ti\*tle"
```

```html
<p><ahref="/bar*"title="ti*tle">foo</a></p>
```

[Example 299](#example-299)

```md
```foo\+bar
foo
```
```

```html
<pre><codeclass="language-foo+bar">foo
</code></pre>
```

## 6.2  Entity and numeric character references

All valid HTML entity references and numeric character references, except those occuring in code blocks and code spans, are recognized as such and treated as equivalent to the corresponding Unicode characters. Conforming CommonMark parsers need not store information about whether a particular character was represented in the source using a Unicode character or an entity reference.

[Entity references](#entity-references) consist of `&` + any of the valid HTML5 entity names + `;`. The document [https://html.spec.whatwg.org/multipage/entities.json](https://html.spec.whatwg.org/multipage/entities.json) is used as an authoritative source for the valid entity references and their corresponding code points.

[Example 300](#example-300)

```md
&nbsp;&amp;&copy;&AElig;&Dcaron;
&frac34;&HilbertSpace;&DifferentialD;
&ClockwiseContourIntegral;&ngE;
```

```html
<p> &amp;©ÆĎ
¾ℋⅆ
∲≧̸</p>
```

[Decimal numeric character](#decimal-numeric-character) consist of `&#` + a string of 1–8 arabic digits + `;`. A numeric character reference is parsed as the corresponding Unicode character. Invalid Unicode code points will be replaced by the REPLACEMENT CHARACTER (`U+FFFD`). For security reasons, the code point `U+0000` will also be replaced by `U+FFFD`.

[Example 301](#example-301)

```md
&#35;&#1234;&#992;&#98765432;&#0;
```

```html
<p>#ӒϠ��</p>
```

[Hexadecimal numeric character](#hexadecimal-numeric-character) consist of `&#` + either `X` or `x` + a string of 1-8 hexadecimal digits + `;`. They too are parsed as the corresponding Unicode character (this time specified with a hexadecimal numeral instead of decimal).

[Example 302](#example-302)

```md
&#X22;&#XD06;&#xcab;
```

```html
<p>&quot;ആಫ</p>
```

Here are some nonentities:

[Example 303](#example-303)

```md
&nbsp&x;&#;&#x;
&ThisIsNotDefined;&hi?;
```

```html
<p>&amp;nbsp&amp;x;&amp;#;&amp;#x;
&amp;ThisIsNotDefined;&amp;hi?;</p>
```

Although HTML5 does accept some entity references without a trailing semicolon (such as `&copy`), these are not recognized here, because it makes the grammar too ambiguous:

[Example 304](#example-304)

```md
&copy
```

```html
<p>&amp;copy</p>
```

Strings that are not on the list of HTML5 named entities are not recognized as entity references either:

[Example 305](#example-305)

```md
&MadeUpEntity;
```

```html
<p>&amp;MadeUpEntity;</p>
```

Entity and numeric character references are recognized in any context besides code spans or code blocks, including URLs, [link titles](#link-title), and [fenced code block](#fenced-code-block) [info strings](#info-string):

[Example 306](#example-306)

```md
<ahref="&ouml;&ouml;.html">
```

```html
<ahref="&ouml;&ouml;.html">
```

[Example 307](#example-307)

```md
[foo](/f&ouml;&ouml;"f&ouml;&ouml;")
```

```html
<p><ahref="/f%C3%B6%C3%B6"title="föö">foo</a></p>
```

[Example 308](#example-308)

```md
[foo]

[foo]:/f&ouml;&ouml;"f&ouml;&ouml;"
```

```html
<p><ahref="/f%C3%B6%C3%B6"title="föö">foo</a></p>
```

[Example 309](#example-309)

```md
```f&ouml;&ouml;
foo
```
```

```html
<pre><codeclass="language-föö">foo
</code></pre>
```

Entity and numeric character references are treated as literal text in code spans and code blocks:

[Example 310](#example-310)

```md
`f&ouml;&ouml;`
```

```html
<p><code>f&amp;ouml;&amp;ouml;</code></p>
```

[Example 311](#example-311)

```md
f&ouml;f&ouml;
```

```html
<pre><code>f&amp;ouml;f&amp;ouml;
</code></pre>
```

## 6.3  Code spans

A [backtick string](#backtick-string) is a string of one or more backtick characters (`````) that is neither preceded nor followed by a backtick.

A [code span](#code-span) begins with a backtick string and ends with a backtick string of equal length. The contents of the code span are the characters between the two backtick strings, with leading and trailing spaces and [line endings](#line-ending) removed, and [whitespace](#whitespace) collapsed to single spaces.

This is a simple code span:

[Example 312](#example-312)

```md
`foo`
```

```html
<p><code>foo</code></p>
```

Here two backticks are used, because the code contains a backtick. This example also illustrates stripping of leading and trailing spaces:

[Example 313](#example-313)

```md
``foo`bar``
```

```html
<p><code>foo`bar</code></p>
```

This example shows the motivation for stripping leading and trailing spaces:

[Example 314](#example-314)

```md
````
```

```html
<p><code>``</code></p>
```

[Line endings](#line-ending) are treated like spaces:

[Example 315](#example-315)

```md
``
foo
``
```

```html
<p><code>foo</code></p>
```

Interior spaces and [line endings](#line-ending) are collapsed into single spaces, just as they would be by a browser:

[Example 316](#example-316)

```md
`foobar
baz`
```

```html
<p><code>foobarbaz</code></p>
```

Not all [Unicode whitespace](#unicode-whitespace) (for instance, non-breaking space) is collapsed, however:

[Example 317](#example-317)

```md
`a  b`
```

```html
<p><code>a  b</code></p>
```

Q: Why not just leave the spaces, since browsers will collapse them anyway? A: Because we might be targeting a non-HTML format, and we shouldn't rely on HTML-specific rendering assumptions.

(Existing implementations differ in their treatment of internal spaces and [line endings](#line-ending). Some, including `Markdown.pl` and `showdown`, convert an internal [line ending](#line-ending) into a `<br />` tag. But this makes things difficult for those who like to hard-wrap their paragraphs, since a line break in the midst of a code span will cause an unintended line break in the output. Others just leave internal spaces as they are, which is fine if only HTML is being targeted.)

[Example 318](#example-318)

```md
`foo``bar`
```

```html
<p><code>foo``bar</code></p>
```

Note that backslash escapes do not work in code spans. All backslashes are treated literally:

[Example 319](#example-319)

```md
`foo\`bar`
```

```html
<p><code>foo\</code>bar`</p>
```

Backslash escapes are never needed, because one can always choose a string of _n_ backtick characters as delimiters, where the code does not contain any strings of exactly _n_ backtick characters.

Code span backticks have higher precedence than any other inline constructs except HTML tags and autolinks. Thus, for example, this is not parsed as emphasized text, since the second `*` is part of a code span:

[Example 320](#example-320)

```md
*foo`*`
```

```html
<p>*foo<code>*</code></p>
```

And this is not parsed as a link:

[Example 321](#example-321)

```md
[nota`link](/foo`)
```

```html
<p>[nota<code>link](/foo</code>)</p>
```

Code spans, HTML tags, and autolinks have the same precedence. Thus, this is code:

[Example 322](#example-322)

```md
`<ahref="`">`
```

```html
<p><code>&lt;ahref=&quot;</code>&quot;&gt;`</p>
```

But this is an HTML tag:

[Example 323](#example-323)

```md
<ahref="`">`
```

```html
<p><ahref="`">`</p>
```

And this is code:

[Example 324](#example-324)

```md
`<http://foo.bar.`baz>`
```

```html
<p><code>&lt;http://foo.bar.</code>baz&gt;`</p>
```

But this is an autolink:

[Example 325](#example-325)

```md
<http://foo.bar.`baz>`
```

```html
<p><ahref="http://foo.bar.%60baz">http://foo.bar.`baz</a>`</p>
```

When a backtick string is not closed by a matching backtick string, we just have literal backticks:

[Example 326](#example-326)

```md
```foo``
```

```html
<p>```foo``</p>
```

[Example 327](#example-327)

```md
`foo
```

```html
<p>`foo</p>
```

## 6.4  Emphasis and strong emphasis

John Gruber's original [Markdown syntax description](http://daringfireball.net/projects/markdown/syntax#em) says:

> Markdown treats asterisks (`*`) and underscores (`_`) as indicators of emphasis. Text wrapped with one `*` or `_` will be wrapped with an HTML `<em>` tag; double `*`'s or `_`'s will be wrapped with an HTML `<strong>` tag.

This is enough for most users, but these rules leave much undecided, especially when it comes to nested emphasis. The original `Markdown.pl` test suite makes it clear that triple `***` and `___` delimiters can be used for strong emphasis, and most implementations have also allowed the following patterns:

```md
***strong emph***
***strong** in emph*
***emph* in strong**
**in strong *emph***
*in emph **strong***
```

The following patterns are less widely supported, but the intent is clear and they are useful (especially in contexts like bibliography entries):

```md
*emph *with emph* in it*
**strong **with strong** in it**
```

Many implementations have also restricted intraword emphasis to the `*` forms, to avoid unwanted emphasis in words containing internal underscores. (It is best practice to put these in code spans, but users often do not.)

```md
internal emphasis: foo*bar*baz
no emphasis: foo_bar_baz
```

The rules given below capture all of these patterns, while allowing for efficient parsing strategies that do not backtrack.

First, some definitions. A [delimiter run](#delimiter-run) is either a sequence of one or more `*` characters that is not preceded or followed by a `*` character, or a sequence of one or more `_` characters that is not preceded or followed by a `_` character.

A [left-flanking delimiter run](#left-flanking-delimiter-run) is a [delimiter run](#delimiter-run) that is (a) not followed by [Unicode whitespace](#unicode-whitespace), and (b) either not followed by a [punctuation character](#punctuation-character), or preceded by [Unicode whitespace](#unicode-whitespace) or a [punctuation character](#punctuation-character). For purposes of this definition, the beginning and the end of the line count as Unicode whitespace.

A [right-flanking delimiter run](#right-flanking-delimiter-run) is a [delimiter run](#delimiter-run) that is (a) not preceded by [Unicode whitespace](#unicode-whitespace), and (b) either not preceded by a [punctuation character](#punctuation-character), or followed by [Unicode whitespace](#unicode-whitespace) or a [punctuation character](#punctuation-character). For purposes of this definition, the beginning and the end of the line count as Unicode whitespace.

Here are some examples of delimiter runs.

* left-flanking but not right-flanking:

```
***abc
  _abc
**"abc"
 _"abc"
```
* right-flanking but not left-flanking:

```
 abc***
 abc_
"abc"**
"abc"_
```
* Both left and right-flanking:

```
 abc***def
"abc"_"def"
```
* Neither left nor right-flanking:

```
abc *** def
a _ b
```

(The idea of distinguishing left-flanking and right-flanking delimiter runs based on the character before and the character after comes from Roopesh Chander's [vfmd](http://www.vfmd.org/vfmd-spec/specification/#procedure-for-identifying-emphasis-tags). vfmd uses the terminology "emphasis indicator string" instead of "delimiter run," and its rules for distinguishing left- and right-flanking runs are a bit more complex than the ones given here.)

The following rules define emphasis and strong emphasis:

1. A single `*` character [can open emphasis](#can-open-emphasis) iff (if and only if) it is part of a [left-flanking delimiter run](#left-flanking-delimiter-run).
2. A single `_` character [can open emphasis](#can-open-emphasis) iff it is part of a [left-flanking delimiter run](#left-flanking-delimiter-run) and either (a) not part of a [right-flanking delimiter run](#right-flanking-delimiter-run) or (b) part of a [right-flanking delimiter run](#right-flanking-delimiter-run) preceded by punctuation.
3. A single `*` character [can close emphasis](#can-close-emphasis) iff it is part of a [right-flanking delimiter run](#right-flanking-delimiter-run).
4. A single `_` character [can close emphasis](#can-close-emphasis) iff it is part of a [right-flanking delimiter run](#right-flanking-delimiter-run) and either (a) not part of a [left-flanking delimiter run](#left-flanking-delimiter-run) or (b) part of a [left-flanking delimiter run](#left-flanking-delimiter-run) followed by punctuation.
5. A double `**` [can open strong emphasis](#can-open-strong-emphasis) iff it is part of a [left-flanking delimiter run](#left-flanking-delimiter-run).
6. A double `__` [can open strong emphasis](#can-open-strong-emphasis) iff it is part of a [left-flanking delimiter run](#left-flanking-delimiter-run) and either (a) not part of a [right-flanking delimiter run](#right-flanking-delimiter-run) or (b) part of a [right-flanking delimiter run](#right-flanking-delimiter-run) preceded by punctuation.
7. A double `**` [can close strong emphasis](#can-close-strong-emphasis) iff it is part of a [right-flanking delimiter run](#right-flanking-delimiter-run).
8. A double `__` [can close strong emphasis](#can-close-strong-emphasis) it is part of a [right-flanking delimiter run](#right-flanking-delimiter-run) and either (a) not part of a [left-flanking delimiter run](#left-flanking-delimiter-run) or (b) part of a [left-flanking delimiter run](#left-flanking-delimiter-run) followed by punctuation.
9. Emphasis begins with a delimiter that [can open emphasis](#can-open-emphasis) and ends with a delimiter that [can close emphasis](#can-close-emphasis), and that uses the same character (`_` or `*`) as the opening delimiter. The opening and closing delimiters must belong to separate [delimiter runs](#delimiter-run). If one of the delimiters can both open and close emphasis, then the sum of the lengths of the delimiter runs containing the opening and closing delimiters must not be a multiple of 3.
10. Strong emphasis begins with a delimiter that [can open strong emphasis](#can-open-strong-emphasis) and ends with a delimiter that [can close strong emphasis](#can-close-strong-emphasis), and that uses the same character (`_` or `*`) as the opening delimiter. The opening and closing delimiters must belong to separate [delimiter runs](#delimiter-run). If one of the delimiters can both open and close strong emphasis, then the sum of the lengths of the delimiter runs containing the opening and closing delimiters must not be a multiple of 3.
11. A literal `*` character cannot occur at the beginning or end of `*`-delimited emphasis or `**`-delimited strong emphasis, unless it is backslash-escaped.
12. A literal `_` character cannot occur at the beginning or end of `_`-delimited emphasis or `__`-delimited strong emphasis, unless it is backslash-escaped.

Where rules 1–12 above are compatible with multiple parsings, the following principles resolve ambiguity:

1. The number of nestings should be minimized. Thus, for example, an interpretation `<strong>...</strong>` is always preferred to `<em><em>...</em></em>`.
2. An interpretation `<strong><em>...</em></strong>` is always preferred to `<em><strong>..</strong></em>`.
3. When two potential emphasis or strong emphasis spans overlap, so that the second begins before the first ends and ends after the first ends, the first takes precedence. Thus, for example, `*foo _bar* baz_` is parsed as `<em>foo _bar</em> baz_` rather than `*foo <em>bar* baz</em>`.
4. When there are two potential emphasis or strong emphasis spans with the same closing delimiter, the shorter one (the one that opens later) takes precedence. Thus, for example, `**foo **bar baz**` is parsed as `**foo <strong>bar baz</strong>` rather than `<strong>foo **bar baz</strong>`.
5. Inline code spans, links, images, and HTML tags group more tightly than emphasis. So, when there is a choice between an interpretation that contains one of these elements and one that does not, the former always wins. Thus, for example, `*[foo*](bar)` is parsed as `*<a href="bar">foo*</a>` rather than as `<em>[foo</em>](bar)`.

These rules can be illustrated through a series of examples.

Rule 1:

[Example 328](#example-328)

```md
*foobar*
```

```html
<p><em>foobar</em></p>
```

This is not emphasis, because the opening `*` is followed by whitespace, and hence not part of a [left-flanking delimiter run](#left-flanking-delimiter-run):

[Example 329](#example-329)

```md
a*foobar*
```

```html
<p>a*foobar*</p>
```

This is not emphasis, because the opening `*` is preceded by an alphanumeric and followed by punctuation, and hence not part of a [left-flanking delimiter run](#left-flanking-delimiter-run):

[Example 330](#example-330)

```md
a*"foo"*
```

```html
<p>a*&quot;foo&quot;*</p>
```

Unicode nonbreaking spaces count as whitespace, too:

[Example 331](#example-331)

```md
* a *
```

```html
<p>* a *</p>
```

Intraword emphasis with `*` is permitted:

[Example 332](#example-332)

```md
foo*bar*
```

```html
<p>foo<em>bar</em></p>
```

[Example 333](#example-333)

```md
5*6*78
```

```html
<p>5<em>6</em>78</p>
```

Rule 2:

[Example 334](#example-334)

```md
_foobar_
```

```html
<p><em>foobar</em></p>
```

This is not emphasis, because the opening `_` is followed by whitespace:

[Example 335](#example-335)

```md
_foobar_
```

```html
<p>_foobar_</p>
```

This is not emphasis, because the opening `_` is preceded by an alphanumeric and followed by punctuation:

[Example 336](#example-336)

```md
a_"foo"_
```

```html
<p>a_&quot;foo&quot;_</p>
```

Emphasis with `_` is not allowed inside words:

[Example 337](#example-337)

```md
foo_bar_
```

```html
<p>foo_bar_</p>
```

[Example 338](#example-338)

```md
5_6_78
```

```html
<p>5_6_78</p>
```

[Example 339](#example-339)

```md
пристаням_стремятся_
```

```html
<p>пристаням_стремятся_</p>
```

Here `_` does not generate emphasis, because the first delimiter run is right-flanking and the second left-flanking:

[Example 340](#example-340)

```md
aa_"bb"_cc
```

```html
<p>aa_&quot;bb&quot;_cc</p>
```

This is emphasis, even though the opening delimiter is both left- and right-flanking, because it is preceded by punctuation:

[Example 341](#example-341)

```md
foo-_(bar)_
```

```html
<p>foo-<em>(bar)</em></p>
```

Rule 3:

This is not emphasis, because the closing delimiter does not match the opening delimiter:

[Example 342](#example-342)

```md
_foo*
```

```html
<p>_foo*</p>
```

This is not emphasis, because the closing `*` is preceded by whitespace:

[Example 343](#example-343)

```md
*foobar*
```

```html
<p>*foobar*</p>
```

A newline also counts as whitespace:

[Example 344](#example-344)

```md
*foobar
*
```

```html
<p>*foobar
*</p>
```

This is not emphasis, because the second `*` is preceded by punctuation and followed by an alphanumeric (hence it is not part of a [right-flanking delimiter run](#right-flanking-delimiter-run):

[Example 345](#example-345)

```md
*(*foo)
```

```html
<p>*(*foo)</p>
```

The point of this restriction is more easily appreciated with this example:

[Example 346](#example-346)

```md
*(*foo*)*
```

```html
<p><em>(<em>foo</em>)</em></p>
```

Intraword emphasis with `*` is allowed:

[Example 347](#example-347)

```md
*foo*bar
```

```html
<p><em>foo</em>bar</p>
```

Rule 4:

This is not emphasis, because the closing `_` is preceded by whitespace:

[Example 348](#example-348)

```md
_foobar_
```

```html
<p>_foobar_</p>
```

This is not emphasis, because the second `_` is preceded by punctuation and followed by an alphanumeric:

[Example 349](#example-349)

```md
_(_foo)
```

```html
<p>_(_foo)</p>
```

This is emphasis within emphasis:

[Example 350](#example-350)

```md
_(_foo_)_
```

```html
<p><em>(<em>foo</em>)</em></p>
```

Intraword emphasis is disallowed for `_`:

[Example 351](#example-351)

```md
_foo_bar
```

```html
<p>_foo_bar</p>
```

[Example 352](#example-352)

```md
_пристаням_стремятся
```

```html
<p>_пристаням_стремятся</p>
```

[Example 353](#example-353)

```md
_foo_bar_baz_
```

```html
<p><em>foo_bar_baz</em></p>
```

This is emphasis, even though the closing delimiter is both left- and right-flanking, because it is followed by punctuation:

[Example 354](#example-354)

```md
_(bar)_.
```

```html
<p><em>(bar)</em>.</p>
```

Rule 5:

[Example 355](#example-355)

```md
**foobar**
```

```html
<p><strong>foobar</strong></p>
```

This is not strong emphasis, because the opening delimiter is followed by whitespace:

[Example 356](#example-356)

```md
**foobar**
```

```html
<p>**foobar**</p>
```

This is not strong emphasis, because the opening `**` is preceded by an alphanumeric and followed by punctuation, and hence not part of a [left-flanking delimiter run](#left-flanking-delimiter-run):

[Example 357](#example-357)

```md
a**"foo"**
```

```html
<p>a**&quot;foo&quot;**</p>
```

Intraword strong emphasis with `**` is permitted:

[Example 358](#example-358)

```md
foo**bar**
```

```html
<p>foo<strong>bar</strong></p>
```

Rule 6:

[Example 359](#example-359)

```md
__foobar__
```

```html
<p><strong>foobar</strong></p>
```

This is not strong emphasis, because the opening delimiter is followed by whitespace:

[Example 360](#example-360)

```md
__foobar__
```

```html
<p>__foobar__</p>
```

A newline counts as whitespace:

[Example 361](#example-361)

```md
__
foobar__
```

```html
<p>__
foobar__</p>
```

This is not strong emphasis, because the opening `__` is preceded by an alphanumeric and followed by punctuation:

[Example 362](#example-362)

```md
a__"foo"__
```

```html
<p>a__&quot;foo&quot;__</p>
```

Intraword strong emphasis is forbidden with `__`:

[Example 363](#example-363)

```md
foo__bar__
```

```html
<p>foo__bar__</p>
```

[Example 364](#example-364)

```md
5__6__78
```

```html
<p>5__6__78</p>
```

[Example 365](#example-365)

```md
пристаням__стремятся__
```

```html
<p>пристаням__стремятся__</p>
```

[Example 366](#example-366)

```md
__foo,__bar__,baz__
```

```html
<p><strong>foo,<strong>bar</strong>,baz</strong></p>
```

This is strong emphasis, even though the opening delimiter is both left- and right-flanking, because it is preceded by punctuation:

[Example 367](#example-367)

```md
foo-__(bar)__
```

```html
<p>foo-<strong>(bar)</strong></p>
```

Rule 7:

This is not strong emphasis, because the closing delimiter is preceded by whitespace:

[Example 368](#example-368)

```md
**foobar**
```

```html
<p>**foobar**</p>
```

(Nor can it be interpreted as an emphasized `*foo bar *`, because of Rule 11.)

This is not strong emphasis, because the second `**` is preceded by punctuation and followed by an alphanumeric:

[Example 369](#example-369)

```md
**(**foo)
```

```html
<p>**(**foo)</p>
```

The point of this restriction is more easily appreciated with these examples:

[Example 370](#example-370)

```md
*(**foo**)*
```

```html
<p><em>(<strong>foo</strong>)</em></p>
```

[Example 371](#example-371)

```md
**Gomphocarpus(*Gomphocarpusphysocarpus*,syn.
*Asclepiasphysocarpa*)**
```

```html
<p><strong>Gomphocarpus(<em>Gomphocarpusphysocarpus</em>,syn.
<em>Asclepiasphysocarpa</em>)</strong></p>
```

[Example 372](#example-372)

```md
**foo"*bar*"foo**
```

```html
<p><strong>foo&quot;<em>bar</em>&quot;foo</strong></p>
```

Intraword emphasis:

[Example 373](#example-373)

```md
**foo**bar
```

```html
<p><strong>foo</strong>bar</p>
```

Rule 8:

This is not strong emphasis, because the closing delimiter is preceded by whitespace:

[Example 374](#example-374)

```md
__foobar__
```

```html
<p>__foobar__</p>
```

This is not strong emphasis, because the second `__` is preceded by punctuation and followed by an alphanumeric:

[Example 375](#example-375)

```md
__(__foo)
```

```html
<p>__(__foo)</p>
```

The point of this restriction is more easily appreciated with this example:

[Example 376](#example-376)

```md
_(__foo__)_
```

```html
<p><em>(<strong>foo</strong>)</em></p>
```

Intraword strong emphasis is forbidden with `__`:

[Example 377](#example-377)

```md
__foo__bar
```

```html
<p>__foo__bar</p>
```

[Example 378](#example-378)

```md
__пристаням__стремятся
```

```html
<p>__пристаням__стремятся</p>
```

[Example 379](#example-379)

```md
__foo__bar__baz__
```

```html
<p><strong>foo__bar__baz</strong></p>
```

This is strong emphasis, even though the closing delimiter is both left- and right-flanking, because it is followed by punctuation:

[Example 380](#example-380)

```md
__(bar)__.
```

```html
<p><strong>(bar)</strong>.</p>
```

Rule 9:

Any nonempty sequence of inline elements can be the contents of an emphasized span.

[Example 381](#example-381)

```md
*foo[bar](/url)*
```

```html
<p><em>foo<ahref="/url">bar</a></em></p>
```

[Example 382](#example-382)

```md
*foo
bar*
```

```html
<p><em>foo
bar</em></p>
```

In particular, emphasis and strong emphasis can be nested inside emphasis:

[Example 383](#example-383)

```md
_foo__bar__baz_
```

```html
<p><em>foo<strong>bar</strong>baz</em></p>
```

[Example 384](#example-384)

```md
_foo_bar_baz_
```

```html
<p><em>foo<em>bar</em>baz</em></p>
```

[Example 385](#example-385)

```md
__foo_bar_
```

```html
<p><em><em>foo</em>bar</em></p>
```

[Example 386](#example-386)

```md
*foo*bar**
```

```html
<p><em>foo<em>bar</em></em></p>
```

[Example 387](#example-387)

```md
*foo**bar**baz*
```

```html
<p><em>foo<strong>bar</strong>baz</em></p>
```

[Example 388](#example-388)

```md
*foo**bar**baz*
```

```html
<p><em>foo<strong>bar</strong>baz</em></p>
```

Note that in the preceding case, the interpretation

```md
<p><em>foo</em><em>bar<em></em>baz</em></p>
```

is precluded by the condition that a delimiter that can both open and close (like the `*` after `foo`) cannot form emphasis if the sum of the lengths of the delimiter runs containing the opening and closing delimiters is a multiple of 3.

The same condition ensures that the following cases are all strong emphasis nested inside emphasis, even when the interior spaces are omitted:

[Example 389](#example-389)

```md
***foo**bar*
```

```html
<p><em><strong>foo</strong>bar</em></p>
```

[Example 390](#example-390)

```md
*foo**bar***
```

```html
<p><em>foo<strong>bar</strong></em></p>
```

[Example 391](#example-391)

```md
*foo**bar***
```

```html
<p><em>foo<strong>bar</strong></em></p>
```

Indefinite levels of nesting are possible:

[Example 392](#example-392)

```md
*foo**bar*baz*bim**bop*
```

```html
<p><em>foo<strong>bar<em>baz</em>bim</strong>bop</em></p>
```

[Example 393](#example-393)

```md
*foo[*bar*](/url)*
```

```html
<p><em>foo<ahref="/url"><em>bar</em></a></em></p>
```

There can be no empty emphasis or strong emphasis:

[Example 394](#example-394)

```md
**isnotanemptyemphasis
```

```html
<p>**isnotanemptyemphasis</p>
```

[Example 395](#example-395)

```md
****isnotanemptystrongemphasis
```

```html
<p>****isnotanemptystrongemphasis</p>
```

Rule 10:

Any nonempty sequence of inline elements can be the contents of an strongly emphasized span.

[Example 396](#example-396)

```md
**foo[bar](/url)**
```

```html
<p><strong>foo<ahref="/url">bar</a></strong></p>
```

[Example 397](#example-397)

```md
**foo
bar**
```

```html
<p><strong>foo
bar</strong></p>
```

In particular, emphasis and strong emphasis can be nested inside strong emphasis:

[Example 398](#example-398)

```md
__foo_bar_baz__
```

```html
<p><strong>foo<em>bar</em>baz</strong></p>
```

[Example 399](#example-399)

```md
__foo__bar__baz__
```

```html
<p><strong>foo<strong>bar</strong>baz</strong></p>
```

[Example 400](#example-400)

```md
____foo__bar__
```

```html
<p><strong><strong>foo</strong>bar</strong></p>
```

[Example 401](#example-401)

```md
**foo**bar****
```

```html
<p><strong>foo<strong>bar</strong></strong></p>
```

[Example 402](#example-402)

```md
**foo*bar*baz**
```

```html
<p><strong>foo<em>bar</em>baz</strong></p>
```

[Example 403](#example-403)

```md
**foo*bar*baz**
```

```html
<p><strong>foo<em>bar</em>baz</strong></p>
```

[Example 404](#example-404)

```md
***foo*bar**
```

```html
<p><strong><em>foo</em>bar</strong></p>
```

[Example 405](#example-405)

```md
**foo*bar***
```

```html
<p><strong>foo<em>bar</em></strong></p>
```

Indefinite levels of nesting are possible:

[Example 406](#example-406)

```md
**foo*bar**baz**
bim*bop**
```

```html
<p><strong>foo<em>bar<strong>baz</strong>
bim</em>bop</strong></p>
```

[Example 407](#example-407)

```md
**foo[*bar*](/url)**
```

```html
<p><strong>foo<ahref="/url"><em>bar</em></a></strong></p>
```

There can be no empty emphasis or strong emphasis:

[Example 408](#example-408)

```md
__isnotanemptyemphasis
```

```html
<p>__isnotanemptyemphasis</p>
```

[Example 409](#example-409)

```md
____isnotanemptystrongemphasis
```

```html
<p>____isnotanemptystrongemphasis</p>
```

Rule 11:

[Example 410](#example-410)

```md
foo***
```

```html
<p>foo***</p>
```

[Example 411](#example-411)

```md
foo*\**
```

```html
<p>foo<em>*</em></p>
```

[Example 412](#example-412)

```md
foo*_*
```

```html
<p>foo<em>_</em></p>
```

[Example 413](#example-413)

```md
foo*****
```

```html
<p>foo*****</p>
```

[Example 414](#example-414)

```md
foo**\***
```

```html
<p>foo<strong>*</strong></p>
```

[Example 415](#example-415)

```md
foo**_**
```

```html
<p>foo<strong>_</strong></p>
```

Note that when delimiters do not match evenly, Rule 11 determines that the excess literal `*` characters will appear outside of the emphasis, rather than inside it:

[Example 416](#example-416)

```md
**foo*
```

```html
<p>*<em>foo</em></p>
```

[Example 417](#example-417)

```md
*foo**
```

```html
<p><em>foo</em>*</p>
```

[Example 418](#example-418)

```md
***foo**
```

```html
<p>*<strong>foo</strong></p>
```

[Example 419](#example-419)

```md
****foo*
```

```html
<p>***<em>foo</em></p>
```

[Example 420](#example-420)

```md
**foo***
```

```html
<p><strong>foo</strong>*</p>
```

[Example 421](#example-421)

```md
*foo****
```

```html
<p><em>foo</em>***</p>
```

Rule 12:

[Example 422](#example-422)

```md
foo___
```

```html
<p>foo___</p>
```

[Example 423](#example-423)

```md
foo_\__
```

```html
<p>foo<em>_</em></p>
```

[Example 424](#example-424)

```md
foo_*_
```

```html
<p>foo<em>*</em></p>
```

[Example 425](#example-425)

```md
foo_____
```

```html
<p>foo_____</p>
```

[Example 426](#example-426)

```md
foo__\___
```

```html
<p>foo<strong>_</strong></p>
```

[Example 427](#example-427)

```md
foo__*__
```

```html
<p>foo<strong>*</strong></p>
```

[Example 428](#example-428)

```md
__foo_
```

```html
<p>_<em>foo</em></p>
```

Note that when delimiters do not match evenly, Rule 12 determines that the excess literal `_` characters will appear outside of the emphasis, rather than inside it:

[Example 429](#example-429)

```md
_foo__
```

```html
<p><em>foo</em>_</p>
```

[Example 430](#example-430)

```md
___foo__
```

```html
<p>_<strong>foo</strong></p>
```

[Example 431](#example-431)

```md
____foo_
```

```html
<p>___<em>foo</em></p>
```

[Example 432](#example-432)

```md
__foo___
```

```html
<p><strong>foo</strong>_</p>
```

[Example 433](#example-433)

```md
_foo____
```

```html
<p><em>foo</em>___</p>
```

Rule 13 implies that if you want emphasis nested directly inside emphasis, you must use different delimiters:

[Example 434](#example-434)

```md
**foo**
```

```html
<p><strong>foo</strong></p>
```

[Example 435](#example-435)

```md
*_foo_*
```

```html
<p><em><em>foo</em></em></p>
```

[Example 436](#example-436)

```md
__foo__
```

```html
<p><strong>foo</strong></p>
```

[Example 437](#example-437)

```md
_*foo*_
```

```html
<p><em><em>foo</em></em></p>
```

However, strong emphasis within strong emphasis is possible without switching delimiters:

[Example 438](#example-438)

```md
****foo****
```

```html
<p><strong><strong>foo</strong></strong></p>
```

[Example 439](#example-439)

```md
____foo____
```

```html
<p><strong><strong>foo</strong></strong></p>
```

Rule 13 can be applied to arbitrarily long sequences of delimiters:

[Example 440](#example-440)

```md
******foo******
```

```html
<p><strong><strong><strong>foo</strong></strong></strong></p>
```

Rule 14:

[Example 441](#example-441)

```md
***foo***
```

```html
<p><strong><em>foo</em></strong></p>
```

[Example 442](#example-442)

```md
_____foo_____
```

```html
<p><strong><strong><em>foo</em></strong></strong></p>
```

Rule 15:

[Example 443](#example-443)

```md
*foo_bar*baz_
```

```html
<p><em>foo_bar</em>baz_</p>
```

[Example 444](#example-444)

```md
*foo__bar*bazbim__bam*
```

```html
<p><em>foo<strong>bar*bazbim</strong>bam</em></p>
```

Rule 16:

[Example 445](#example-445)

```md
**foo**barbaz**
```

```html
<p>**foo<strong>barbaz</strong></p>
```

[Example 446](#example-446)

```md
*foo*barbaz*
```

```html
<p>*foo<em>barbaz</em></p>
```

Rule 17:

[Example 447](#example-447)

```md
*[bar*](/url)
```

```html
<p>*<ahref="/url">bar*</a></p>
```

[Example 448](#example-448)

```md
_foo[bar_](/url)
```

```html
<p>_foo<ahref="/url">bar_</a></p>
```

[Example 449](#example-449)

```md
*<imgsrc="foo"title="*"/>
```

```html
<p>*<imgsrc="foo"title="*"/></p>
```

[Example 450](#example-450)

```md
**<ahref="**">
```

```html
<p>**<ahref="**"></p>
```

[Example 451](#example-451)

```md
__<ahref="__">
```

```html
<p>__<ahref="__"></p>
```

[Example 452](#example-452)

```md
*a`*`*
```

```html
<p><em>a<code>*</code></em></p>
```

[Example 453](#example-453)

```md
_a`_`_
```

```html
<p><em>a<code>_</code></em></p>
```

[Example 454](#example-454)

```md
**a<http://foo.bar/?q=**>
```

```html
<p>**a<ahref="http://foo.bar/?q=**">http://foo.bar/?q=**</a></p>
```

[Example 455](#example-455)

```md
__a<http://foo.bar/?q=__>
```

```html
<p>__a<ahref="http://foo.bar/?q=__">http://foo.bar/?q=__</a></p>
```

## 6.5  Links

A link contains [link text](#link-text) (the visible text), a [link destination](#link-destination) (the URI that is the link destination), and optionally a [link title](#link-title). There are two basic kinds of links in Markdown. In [inline links](#inline-link) the destination and title are given immediately after the link text. In [reference links](#reference-link) the destination and title are defined elsewhere in the document.

A [link text](#link-text) consists of a sequence of zero or more inline elements enclosed by square brackets (`[` and `]`). The following rules apply:

* Links may not contain other links, at any level of nesting. If multiple otherwise valid link definitions appear nested inside each other, the inner-most definition is used.
* Brackets are allowed in the [link text](#link-text) only if (a) they are backslash-escaped or (b) they appear as a matched pair of brackets, with an open bracket `[`, a sequence of zero or more inlines, and a close bracket `]`.
* Backtick [code spans](#code-span), [autolinks](#autolinks), and raw [HTML tags](#html-tag) bind more tightly than the brackets in link text. Thus, for example, ``[foo`]``` could not be a link text, since the second `]` is part of a code span.
* The brackets in link text bind more tightly than markers for [emphasis and strong emphasis](#emphasis-and-strong-emphasis). Thus, for example, `*[foo*](url)` is a link.

A [link destination](#link-destination) consists of either

* a sequence of zero or more characters between an opening `<` and a closing `>` that contains no spaces, line breaks, or unescaped `<` or `>` characters, or
* a nonempty sequence of characters that does not include ASCII space or control characters, and includes parentheses only if (a) they are backslash-escaped or (b) they are part of a balanced pair of unescaped parentheses that is not itself inside a balanced pair of unescaped parentheses.

A [link title](#link-title) consists of either

* a sequence of zero or more characters between straight double-quote characters (`"`), including a `"` character only if it is backslash-escaped, or
* a sequence of zero or more characters between straight single-quote characters (`'`), including a `'` character only if it is backslash-escaped, or
* a sequence of zero or more characters between matching parentheses (`(...)`), including a `)` character only if it is backslash-escaped.

Although [link titles](#link-title) may span multiple lines, they may not contain a [blank line](#blank-line).

An [inline link](#inline-link) consists of a [link text](#link-text) followed immediately by a left parenthesis `(`, optional [whitespace](#whitespace), an optional [link destination](#link-destination), an optional [link title](#link-title) separated from the link destination by [whitespace](#whitespace), optional [whitespace](#whitespace), and a right parenthesis `)`. The link's text consists of the inlines contained in the [link text](#link-text) (excluding the enclosing square brackets). The link's URI consists of the link destination, excluding enclosing `<...>` if present, with backslash-escapes in effect as described above. The link's title consists of the link title, excluding its enclosing delimiters, with backslash-escapes in effect as described above.

Here is a simple inline link:

[Example 456](#example-456)

```md
[link](/uri"title")
```

```html
<p><ahref="/uri"title="title">link</a></p>
```

The title may be omitted:

[Example 457](#example-457)

```md
[link](/uri)
```

```html
<p><ahref="/uri">link</a></p>
```

Both the title and the destination may be omitted:

[Example 458](#example-458)

```md
[link]()
```

```html
<p><ahref="">link</a></p>
```

[Example 459](#example-459)

```md
[link](<>)
```

```html
<p><ahref="">link</a></p>
```

The destination cannot contain spaces or line breaks, even if enclosed in pointy brackets:

[Example 460](#example-460)

```md
[link](/myuri)
```

```html
<p>[link](/myuri)</p>
```

[Example 461](#example-461)

```md
[link](</myuri>)
```

```html
<p>[link](&lt;/myuri&gt;)</p>
```

[Example 462](#example-462)

```md
[link](foo
bar)
```

```html
<p>[link](foo
bar)</p>
```

[Example 463](#example-463)

```md
[link](<foo
bar>)
```

```html
<p>[link](<foo
bar>)</p>
```

Parentheses inside the link destination may be escaped:

[Example 464](#example-464)

```md
[link](\(foo\))
```

```html
<p><ahref="(foo)">link</a></p>
```

One level of balanced parentheses is allowed without escaping:

[Example 465](#example-465)

```md
[link]((foo)and(bar))
```

```html
<p><ahref="(foo)and(bar)">link</a></p>
```

However, if you have parentheses within parentheses, you need to escape or use the `<...>` form:

[Example 466](#example-466)

```md
[link](foo(and(bar)))
```

```html
<p>[link](foo(and(bar)))</p>
```

[Example 467](#example-467)

```md
[link](foo(and\(bar\)))
```

```html
<p><ahref="foo(and(bar))">link</a></p>
```

[Example 468](#example-468)

```md
[link](<foo(and(bar))>)
```

```html
<p><ahref="foo(and(bar))">link</a></p>
```

Parentheses and other symbols can also be escaped, as usual in Markdown:

[Example 469](#example-469)

```md
[link](foo\)\:)
```

```html
<p><ahref="foo):">link</a></p>
```

A link can contain fragment identifiers and queries:

[Example 470](#example-470)

```md
[link](#fragment)

[link](http://example.com#fragment)

[link](http://example.com?foo=3#frag)
```

```html
<p><ahref="#fragment">link</a></p>
<p><ahref="http://example.com#fragment">link</a></p>
<p><ahref="http://example.com?foo=3#frag">link</a></p>
```

Note that a backslash before a non-escapable character is just a backslash:

[Example 471](#example-471)

```md
[link](foo\bar)
```

```html
<p><ahref="foo%5Cbar">link</a></p>
```

URL-escaping should be left alone inside the destination, as all URL-escaped characters are also valid URL characters. Entity and numerical character references in the destination will be parsed into the corresponding Unicode code points, as usual. These may be optionally URL-escaped when written as HTML, but this spec does not enforce any particular policy for rendering URLs in HTML or other formats. Renderers may make different decisions about how to escape or normalize URLs in the output.

[Example 472](#example-472)

```md
[link](foo%20b&auml;)
```

```html
<p><ahref="foo%20b%C3%A4">link</a></p>
```

Note that, because titles can often be parsed as destinations, if you try to omit the destination and keep the title, you'll get unexpected results:

[Example 473](#example-473)

```md
[link]("title")
```

```html
<p><ahref="%22title%22">link</a></p>
```

Titles may be in single quotes, double quotes, or parentheses:

[Example 474](#example-474)

```md
[link](/url"title")
[link](/url'title')
[link](/url(title))
```

```html
<p><ahref="/url"title="title">link</a>
<ahref="/url"title="title">link</a>
<ahref="/url"title="title">link</a></p>
```

Backslash escapes and entity and numeric character references may be used in titles:

[Example 475](#example-475)

```md
[link](/url"title\"&quot;")
```

```html
<p><ahref="/url"title="title&quot;&quot;">link</a></p>
```

Titles must be separated from the link using a [whitespace](#whitespace). Other [Unicode whitespace](#unicode-whitespace) like non-breaking space doesn't work.

[Example 476](#example-476)

```md
[link](/url "title")
```

```html
<p><ahref="/url%C2%A0%22title%22">link</a></p>
```

Nested balanced quotes are not allowed without escaping:

[Example 477](#example-477)

```md
[link](/url"title"and"title")
```

```html
<p>[link](/url&quot;title&quot;and&quot;title&quot;)</p>
```

But it is easy to work around this by using a different quote type:

[Example 478](#example-478)

```md
[link](/url'title"and"title')
```

```html
<p><ahref="/url"title="title&quot;and&quot;title">link</a></p>
```

(Note: `Markdown.pl` did allow double quotes inside a double-quoted title, and its test suite included a test demonstrating this. But it is hard to see a good rationale for the extra complexity this brings, since there are already many ways—backslash escaping, entity and numeric character references, or using a different quote type for the enclosing title—to write titles containing double quotes. `Markdown.pl`'s handling of titles has a number of other strange features. For example, it allows single-quoted titles in inline links, but not reference links. And, in reference links but not inline links, it allows a title to begin with `"` and end with `)`. `Markdown.pl` 1.0.1 even allows titles with no closing quotation mark, though 1.0.2b8 does not. It seems preferable to adopt a simple, rational rule that works the same way in inline links and link reference definitions.)

[Whitespace](#whitespace) is allowed around the destination and title:

[Example 479](#example-479)

```md
[link](/uri
"title")
```

```html
<p><ahref="/uri"title="title">link</a></p>
```

But it is not allowed between the link text and the following parenthesis:

[Example 480](#example-480)

```md
[link](/uri)
```

```html
<p>[link](/uri)</p>
```

The link text may contain balanced brackets, but not unbalanced ones, unless they are escaped:

[Example 481](#example-481)

```md
[link[foo[bar]]](/uri)
```

```html
<p><ahref="/uri">link[foo[bar]]</a></p>
```

[Example 482](#example-482)

```md
[link]bar](/uri)
```

```html
<p>[link]bar](/uri)</p>
```

[Example 483](#example-483)

```md
[link[bar](/uri)
```

```html
<p>[link<ahref="/uri">bar</a></p>
```

[Example 484](#example-484)

```md
[link\[bar](/uri)
```

```html
<p><ahref="/uri">link[bar</a></p>
```

The link text may contain inline content:

[Example 485](#example-485)

```md
[link*foo**bar**`#`*](/uri)
```

```html
<p><ahref="/uri">link<em>foo<strong>bar</strong><code>#</code></em></a></p>
```

[Example 486](#example-486)

```md
[![moon](moon.jpg)](/uri)
```

```html
<p><ahref="/uri"><imgsrc="moon.jpg"alt="moon"/></a></p>
```

However, links may not contain other links, at any level of nesting.

[Example 487](#example-487)

```md
[foo[bar](/uri)](/uri)
```

```html
<p>[foo<ahref="/uri">bar</a>](/uri)</p>
```

[Example 488](#example-488)

```md
[foo*[bar[baz](/uri)](/uri)*](/uri)
```

```html
<p>[foo<em>[bar<ahref="/uri">baz</a>](/uri)</em>](/uri)</p>
```

[Example 489](#example-489)

```md
![[[foo](uri1)](uri2)](uri3)
```

```html
<p><imgsrc="uri3"alt="[foo](uri2)"/></p>
```

These cases illustrate the precedence of link text grouping over emphasis grouping:

[Example 490](#example-490)

```md
*[foo*](/uri)
```

```html
<p>*<ahref="/uri">foo*</a></p>
```

[Example 491](#example-491)

```md
[foo*bar](baz*)
```

```html
<p><ahref="baz*">foo*bar</a></p>
```

Note that brackets that _aren't_ part of links do not take precedence:

[Example 492](#example-492)

```md
*foo[bar*baz]
```

```html
<p><em>foo[bar</em>baz]</p>
```

These cases illustrate the precedence of HTML tags, code spans, and autolinks over link grouping:

[Example 493](#example-493)

```md
[foo<barattr="](baz)">
```

```html
<p>[foo<barattr="](baz)"></p>
```

[Example 494](#example-494)

```md
[foo`](/uri)`
```

```html
<p>[foo<code>](/uri)</code></p>
```

[Example 495](#example-495)

```md
[foo<http://example.com/?search=](uri)>
```

```html
<p>[foo<ahref="http://example.com/?search=%5D(uri)">http://example.com/?search=](uri)</a></p>
```

There are three kinds of [reference link](#reference-link)s: [full](#full-reference-link), [collapsed](#collapsed-reference-link), and [shortcut](#shortcut-reference-link).

A [full reference link](#full-reference-link) consists of a [link text](#link-text) immediately followed by a [link label](#link-label) that [matches](#matches) a [link reference definition](#link-reference-definition) elsewhere in the document.

A [link label](#link-label) begins with a left bracket (`[`) and ends with the first right bracket (`]`) that is not backslash-escaped. Between these brackets there must be at least one [non-whitespace character](#non-whitespace-character). Unescaped square bracket characters are not allowed in [link labels](#link-label). A link label can have at most 999 characters inside the square brackets.

One label [matches](#matches) another just in case their normalized forms are equal. To normalize a label, perform the _Unicode case fold_ and collapse consecutive internal [whitespace](#whitespace) to a single space. If there are multiple matching reference link definitions, the one that comes first in the document is used. (It is desirable in such cases to emit a warning.)

The contents of the first link label are parsed as inlines, which are used as the link's text. The link's URI and title are provided by the matching [link reference definition](#link-reference-definition).

Here is a simple example:

[Example 496](#example-496)

```md
[foo][bar]

[bar]:/url"title"
```

```html
<p><ahref="/url"title="title">foo</a></p>
```

The rules for the [link text](#link-text) are the same as with [inline links](#inline-link). Thus:

The link text may contain balanced brackets, but not unbalanced ones, unless they are escaped:

[Example 497](#example-497)

```md
[link[foo[bar]]][ref]

[ref]:/uri
```

```html
<p><ahref="/uri">link[foo[bar]]</a></p>
```

[Example 498](#example-498)

```md
[link\[bar][ref]

[ref]:/uri
```

```html
<p><ahref="/uri">link[bar</a></p>
```

The link text may contain inline content:

[Example 499](#example-499)

```md
[link*foo**bar**`#`*][ref]

[ref]:/uri
```

```html
<p><ahref="/uri">link<em>foo<strong>bar</strong><code>#</code></em></a></p>
```

[Example 500](#example-500)

```md
[![moon](moon.jpg)][ref]

[ref]:/uri
```

```html
<p><ahref="/uri"><imgsrc="moon.jpg"alt="moon"/></a></p>
```

However, links may not contain other links, at any level of nesting.

[Example 501](#example-501)

```md
[foo[bar](/uri)][ref]

[ref]:/uri
```

```html
<p>[foo<ahref="/uri">bar</a>]<ahref="/uri">ref</a></p>
```

[Example 502](#example-502)

```md
[foo*bar[baz][ref]*][ref]

[ref]:/uri
```

```html
<p>[foo<em>bar<ahref="/uri">baz</a></em>]<ahref="/uri">ref</a></p>
```

(In the examples above, we have two [shortcut reference links](#shortcut-reference-link) instead of one [full reference link](#full-reference-link).)

The following cases illustrate the precedence of link text grouping over emphasis grouping:

[Example 503](#example-503)

```md
*[foo*][ref]

[ref]:/uri
```

```html
<p>*<ahref="/uri">foo*</a></p>
```

[Example 504](#example-504)

```md
[foo*bar][ref]

[ref]:/uri
```

```html
<p><ahref="/uri">foo*bar</a></p>
```

These cases illustrate the precedence of HTML tags, code spans, and autolinks over link grouping:

[Example 505](#example-505)

```md
[foo<barattr="][ref]">

[ref]:/uri
```

```html
<p>[foo<barattr="][ref]"></p>
```

[Example 506](#example-506)

```md
[foo`][ref]`

[ref]:/uri
```

```html
<p>[foo<code>][ref]</code></p>
```

[Example 507](#example-507)

```md
[foo<http://example.com/?search=][ref]>

[ref]:/uri
```

```html
<p>[foo<ahref="http://example.com/?search=%5D%5Bref%5D">http://example.com/?search=][ref]</a></p>
```

Matching is case-insensitive:

[Example 508](#example-508)

```md
[foo][BaR]

[bar]:/url"title"
```

```html
<p><ahref="/url"title="title">foo</a></p>
```

Unicode case fold is used:

[Example 509](#example-509)

```md
[Толпой][Толпой]isaRussianword.

[ТОЛПОЙ]:/url
```

```html
<p><ahref="/url">Толпой</a>isaRussianword.</p>
```

Consecutive internal [whitespace](#whitespace) is treated as one space for purposes of determining matching:

[Example 510](#example-510)

```md
[Foo
bar]:/url

[Baz][Foobar]
```

```html
<p><ahref="/url">Baz</a></p>
```

No [whitespace](#whitespace) is allowed between the [link text](#link-text) and the [link label](#link-label):

[Example 511](#example-511)

```md
[foo][bar]

[bar]:/url"title"
```

```html
<p>[foo]<ahref="/url"title="title">bar</a></p>
```

[Example 512](#example-512)

```md
[foo]
[bar]

[bar]:/url"title"
```

```html
<p>[foo]
<ahref="/url"title="title">bar</a></p>
```

This is a departure from John Gruber's original Markdown syntax description, which explicitly allows whitespace between the link text and the link label. It brings reference links in line with [inline links](#inline-link), which (according to both original Markdown and this spec) cannot have whitespace after the link text. More importantly, it prevents inadvertent capture of consecutive [shortcut reference links](#shortcut-reference-link). If whitespace is allowed between the link text and the link label, then in the following we will have a single reference link, not two shortcut reference links, as intended:

```md
[foo]
[bar]

[foo]: /url1
[bar]: /url2
```

(Note that [shortcut reference links](#shortcut-reference-link) were introduced by Gruber himself in a beta version of `Markdown.pl`, but never included in the official syntax description. Without shortcut reference links, it is harmless to allow space between the link text and link label; but once shortcut references are introduced, it is too dangerous to allow this, as it frequently leads to unintended results.)

When there are multiple matching [link reference definitions](#link-reference-definition), the first is used:

[Example 513](#example-513)

```md
[foo]:/url1

[foo]:/url2

[bar][foo]
```

```html
<p><ahref="/url1">bar</a></p>
```

Note that matching is performed on normalized strings, not parsed inline content. So the following does not match, even though the labels define equivalent inline content:

[Example 514](#example-514)

```md
[bar][foo\!]

[foo!]:/url
```

```html
<p>[bar][foo!]</p>
```

[Link labels](#link-label) cannot contain brackets, unless they are backslash-escaped:

[Example 515](#example-515)

```md
[foo][ref[]

[ref[]:/uri
```

```html
<p>[foo][ref[]</p>
<p>[ref[]:/uri</p>
```

[Example 516](#example-516)

```md
[foo][ref[bar]]

[ref[bar]]:/uri
```

```html
<p>[foo][ref[bar]]</p>
<p>[ref[bar]]:/uri</p>
```

[Example 517](#example-517)

```md
[[[foo]]]

[[[foo]]]:/url
```

```html
<p>[[[foo]]]</p>
<p>[[[foo]]]:/url</p>
```

[Example 518](#example-518)

```md
[foo][ref\[]

[ref\[]:/uri
```

```html
<p><ahref="/uri">foo</a></p>
```

Note that in this example `]` is not backslash-escaped:

[Example 519](#example-519)

```md
[bar\\]:/uri

[bar\\]
```

```html
<p><ahref="/uri">bar\</a></p>
```

A [link label](#link-label) must contain at least one [non-whitespace character](#non-whitespace-character):

[Example 520](#example-520)

```md
[]

[]:/uri
```

```html
<p>[]</p>
<p>[]:/uri</p>
```

[Example 521](#example-521)

```md
[
]

[
]:/uri
```

```html
<p>[
]</p>
<p>[
]:/uri</p>
```

A [collapsed reference link](#collapsed-reference-link) consists of a [link label](#link-label) that [matches](#matches) a [link reference definition](#link-reference-definition) elsewhere in the document, followed by the string `[]`. The contents of the first link label are parsed as inlines, which are used as the link's text. The link's URI and title are provided by the matching reference link definition. Thus, `[foo][]` is equivalent to `[foo][foo]`.

[Example 522](#example-522)

```md
[foo][]

[foo]:/url"title"
```

```html
<p><ahref="/url"title="title">foo</a></p>
```

[Example 523](#example-523)

```md
[*foo*bar][]

[*foo*bar]:/url"title"
```

```html
<p><ahref="/url"title="title"><em>foo</em>bar</a></p>
```

The link labels are case-insensitive:

[Example 524](#example-524)

```md
[Foo][]

[foo]:/url"title"
```

```html
<p><ahref="/url"title="title">Foo</a></p>
```

As with full reference links, [whitespace](#whitespace) is not allowed between the two sets of brackets:

[Example 525](#example-525)

```md
[foo]
[]

[foo]:/url"title"
```

```html
<p><ahref="/url"title="title">foo</a>
[]</p>
```

A [shortcut reference link](#shortcut-reference-link) consists of a [link label](#link-label) that [matches](#matches) a [link reference definition](#link-reference-definition) elsewhere in the document and is not followed by `[]` or a link label. The contents of the first link label are parsed as inlines, which are used as the link's text. The link's URI and title are provided by the matching link reference definition. Thus, `[foo]` is equivalent to `[foo][]`.

[Example 526](#example-526)

```md
[foo]

[foo]:/url"title"
```

```html
<p><ahref="/url"title="title">foo</a></p>
```

[Example 527](#example-527)

```md
[*foo*bar]

[*foo*bar]:/url"title"
```

```html
<p><ahref="/url"title="title"><em>foo</em>bar</a></p>
```

[Example 528](#example-528)

```md
[[*foo*bar]]

[*foo*bar]:/url"title"
```

```html
<p>[<ahref="/url"title="title"><em>foo</em>bar</a>]</p>
```

[Example 529](#example-529)

```md
[[bar[foo]

[foo]:/url
```

```html
<p>[[bar<ahref="/url">foo</a></p>
```

The link labels are case-insensitive:

[Example 530](#example-530)

```md
[Foo]

[foo]:/url"title"
```

```html
<p><ahref="/url"title="title">Foo</a></p>
```

A space after the link text should be preserved:

[Example 531](#example-531)

```md
[foo]bar

[foo]:/url
```

```html
<p><ahref="/url">foo</a>bar</p>
```

If you just want bracketed text, you can backslash-escape the opening bracket to avoid links:

[Example 532](#example-532)

```md
\[foo]

[foo]:/url"title"
```

```html
<p>[foo]</p>
```

Note that this is a link, because a link label ends with the first following closing bracket:

[Example 533](#example-533)

```md
[foo*]:/url

*[foo*]
```

```html
<p>*<ahref="/url">foo*</a></p>
```

Full and compact references take precedence over shortcut references:

[Example 534](#example-534)

```md
[foo][bar]

[foo]:/url1
[bar]:/url2
```

```html
<p><ahref="/url2">foo</a></p>
```

[Example 535](#example-535)

```md
[foo][]

[foo]:/url1
```

```html
<p><ahref="/url1">foo</a></p>
```

Inline links also take precedence:

[Example 536](#example-536)

```md
[foo]()

[foo]:/url1
```

```html
<p><ahref="">foo</a></p>
```

[Example 537](#example-537)

```md
[foo](notalink)

[foo]:/url1
```

```html
<p><ahref="/url1">foo</a>(notalink)</p>
```

In the following case `[bar][baz]` is parsed as a reference, `[foo]` as normal text:

[Example 538](#example-538)

```md
[foo][bar][baz]

[baz]:/url
```

```html
<p>[foo]<ahref="/url">bar</a></p>
```

Here, though, `[foo][bar]` is parsed as a reference, since `[bar]` is defined:

[Example 539](#example-539)

```md
[foo][bar][baz]

[baz]:/url1
[bar]:/url2
```

```html
<p><ahref="/url2">foo</a><ahref="/url1">baz</a></p>
```

Here `[foo]` is not parsed as a shortcut reference, because it is followed by a link label (even though `[bar]` is not defined):

[Example 540](#example-540)

```md
[foo][bar][baz]

[baz]:/url1
[foo]:/url2
```

```html
<p>[foo]<ahref="/url1">bar</a></p>
```

## 6.6  Images

Syntax for images is like the syntax for links, with one difference. Instead of [link text](#link-text), we have an [image description](#image-description). The rules for this are the same as for [link text](#link-text), except that (a) an image description starts with `![` rather than `[`, and (b) an image description may contain links. An image description has inline elements as its contents. When an image is rendered to HTML, this is standardly used as the image's `alt` attribute.

[Example 541](#example-541)

```md
![foo](/url"title")
```

```html
<p><imgsrc="/url"alt="foo"title="title"/></p>
```

[Example 542](#example-542)

```md
![foo*bar*]

[foo*bar*]:train.jpg"train&tracks"
```

```html
<p><imgsrc="train.jpg"alt="foobar"title="train&amp;tracks"/></p>
```

[Example 543](#example-543)

```md
![foo![bar](/url)](/url2)
```

```html
<p><imgsrc="/url2"alt="foobar"/></p>
```

[Example 544](#example-544)

```md
![foo[bar](/url)](/url2)
```

```html
<p><imgsrc="/url2"alt="foobar"/></p>
```

Though this spec is concerned with parsing, not rendering, it is recommended that in rendering to HTML, only the plain string content of the [image description](#image-description) be used. Note that in the above example, the alt attribute's value is `foo bar`, not `foo [bar](/url)` or `foo <a href="/url">bar</a>`. Only the plain string content is rendered, without formatting.

[Example 545](#example-545)

```md
![foo*bar*][]

[foo*bar*]:train.jpg"train&tracks"
```

```html
<p><imgsrc="train.jpg"alt="foobar"title="train&amp;tracks"/></p>
```

[Example 546](#example-546)

```md
![foo*bar*][foobar]

[FOOBAR]:train.jpg"train&tracks"
```

```html
<p><imgsrc="train.jpg"alt="foobar"title="train&amp;tracks"/></p>
```

[Example 547](#example-547)

```md
![foo](train.jpg)
```

```html
<p><imgsrc="train.jpg"alt="foo"/></p>
```

[Example 548](#example-548)

```md
My![foobar](/path/to/train.jpg"title")
```

```html
<p>My<imgsrc="/path/to/train.jpg"alt="foobar"title="title"/></p>
```

[Example 549](#example-549)

```md
![foo](<url>)
```

```html
<p><imgsrc="url"alt="foo"/></p>
```

[Example 550](#example-550)

```md
![](/url)
```

```html
<p><imgsrc="/url"alt=""/></p>
```

Reference-style:

[Example 551](#example-551)

```md
![foo][bar]

[bar]:/url
```

```html
<p><imgsrc="/url"alt="foo"/></p>
```

[Example 552](#example-552)

```md
![foo][bar]

[BAR]:/url
```

```html
<p><imgsrc="/url"alt="foo"/></p>
```

Collapsed:

[Example 553](#example-553)

```md
![foo][]

[foo]:/url"title"
```

```html
<p><imgsrc="/url"alt="foo"title="title"/></p>
```

[Example 554](#example-554)

```md
![*foo*bar][]

[*foo*bar]:/url"title"
```

```html
<p><imgsrc="/url"alt="foobar"title="title"/></p>
```

The labels are case-insensitive:

[Example 555](#example-555)

```md
![Foo][]

[foo]:/url"title"
```

```html
<p><imgsrc="/url"alt="Foo"title="title"/></p>
```

As with reference links, [whitespace](#whitespace) is not allowed between the two sets of brackets:

[Example 556](#example-556)

```md
![foo]
[]

[foo]:/url"title"
```

```html
<p><imgsrc="/url"alt="foo"title="title"/>
[]</p>
```

Shortcut:

[Example 557](#example-557)

```md
![foo]

[foo]:/url"title"
```

```html
<p><imgsrc="/url"alt="foo"title="title"/></p>
```

[Example 558](#example-558)

```md
![*foo*bar]

[*foo*bar]:/url"title"
```

```html
<p><imgsrc="/url"alt="foobar"title="title"/></p>
```

Note that link labels cannot contain unescaped brackets:

[Example 559](#example-559)

```md
![[foo]]

[[foo]]:/url"title"
```

```html
<p>![[foo]]</p>
<p>[[foo]]:/url&quot;title&quot;</p>
```

The link labels are case-insensitive:

[Example 560](#example-560)

```md
![Foo]

[foo]:/url"title"
```

```html
<p><imgsrc="/url"alt="Foo"title="title"/></p>
```

If you just want bracketed text, you can backslash-escape the opening `!` and `[`:

[Example 561](#example-561)

```md
\!\[foo]

[foo]:/url"title"
```

```html
<p>![foo]</p>
```

If you want a link after a literal `!`, backslash-escape the `!`:

[Example 562](#example-562)

```md
\![foo]

[foo]:/url"title"
```

```html
<p>!<ahref="/url"title="title">foo</a></p>
```

## 6.7  Autolinks

[Autolink](#autolink)s are absolute URIs and email addresses inside `<` and `>`. They are parsed as links, with the URL or email address as the link label.

A [URI autolink](#uri-autolink) consists of `<`, followed by an [absolute URI](#absolute-uri) not containing `<`, followed by `>`. It is parsed as a link to the URI, with the URI as the link's label.

An [absolute URI](#absolute-uri), for these purposes, consists of a [scheme](#scheme) followed by a colon (`:`) followed by zero or more characters other than ASCII [whitespace](#whitespace) and control characters, `<`, and `>`. If the URI includes these characters, they must be percent-encoded (e.g. `%20` for a space).

For purposes of this spec, a [scheme](#scheme) is any sequence of 2–32 characters beginning with an ASCII letter and followed by any combination of ASCII letters, digits, or the symbols plus ("+"), period ("."), or hyphen ("-").

Here are some valid autolinks:

[Example 563](#example-563)

```md
<http://foo.bar.baz>
```

```html
<p><ahref="http://foo.bar.baz">http://foo.bar.baz</a></p>
```

[Example 564](#example-564)

```md
<http://foo.bar.baz/test?q=hello&id=22&boolean>
```

```html
<p><ahref="http://foo.bar.baz/test?q=hello&amp;id=22&amp;boolean">http://foo.bar.baz/test?q=hello&amp;id=22&amp;boolean</a></p>
```

[Example 565](#example-565)

```md
<irc://foo.bar:2233/baz>
```

```html
<p><ahref="irc://foo.bar:2233/baz">irc://foo.bar:2233/baz</a></p>
```

Uppercase is also fine:

[Example 566](#example-566)

```md
<MAILTO:FOO@BAR.BAZ>
```

```html
<p><ahref="MAILTO:FOO@BAR.BAZ">MAILTO:FOO@BAR.BAZ</a></p>
```

Note that many strings that count as [absolute URIs](#absolute-uri) for purposes of this spec are not valid URIs, because their schemes are not registered or because of other problems with their syntax:

[Example 567](#example-567)

```md
<a+b+c:d>
```

```html
<p><ahref="a+b+c:d">a+b+c:d</a></p>
```

[Example 568](#example-568)

```md
<made-up-scheme://foo,bar>
```

```html
<p><ahref="made-up-scheme://foo,bar">made-up-scheme://foo,bar</a></p>
```

[Example 569](#example-569)

```md
<http://../>
```

```html
<p><ahref="http://../">http://../</a></p>
```

[Example 570](#example-570)

```md
<localhost:5001/foo>
```

```html
<p><ahref="localhost:5001/foo">localhost:5001/foo</a></p>
```

Spaces are not allowed in autolinks:

[Example 571](#example-571)

```md
<http://foo.bar/bazbim>
```

```html
<p>&lt;http://foo.bar/bazbim&gt;</p>
```

Backslash-escapes do not work inside autolinks:

[Example 572](#example-572)

```md
<http://example.com/\[\>
```

```html
<p><ahref="http://example.com/%5C%5B%5C">http://example.com/\[\</a></p>
```

An [email autolink](#email-autolink) consists of `<`, followed by an [email address](#email-address), followed by `>`. The link's label is the email address, and the URL is `mailto:` followed by the email address.

An [email address](#email-address), for these purposes, is anything that matches the [non-normative regex from the HTML5 spec](https://html.spec.whatwg.org/multipage/forms.html#e-mail-state-(type=email)):

```
/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?
(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/
```

Examples of email autolinks:

[Example 573](#example-573)

```md
<foo@bar.example.com>
```

```html
<p><ahref="mailto:foo@bar.example.com">foo@bar.example.com</a></p>
```

[Example 574](#example-574)

```md
<foo+special@Bar.baz-bar0.com>
```

```html
<p><ahref="mailto:foo+special@Bar.baz-bar0.com">foo+special@Bar.baz-bar0.com</a></p>
```

Backslash-escapes do not work inside email autolinks:

[Example 575](#example-575)

```md
<foo\+@bar.example.com>
```

```html
<p>&lt;foo+@bar.example.com&gt;</p>
```

These are not autolinks:

[Example 576](#example-576)

```md
<>
```

```html
<p>&lt;&gt;</p>
```

[Example 577](#example-577)

```md
<http://foo.bar>
```

```html
<p>&lt;http://foo.bar&gt;</p>
```

[Example 578](#example-578)

```md
<m:abc>
```

```html
<p>&lt;m:abc&gt;</p>
```

[Example 579](#example-579)

```md
<foo.bar.baz>
```

```html
<p>&lt;foo.bar.baz&gt;</p>
```

[Example 580](#example-580)

```md
http://example.com
```

```html
<p>http://example.com</p>
```

[Example 581](#example-581)

```md
foo@bar.example.com
```

```html
<p>foo@bar.example.com</p>
```

## 6.8  Raw HTML

Text between `<` and `>` that looks like an HTML tag is parsed as a raw HTML tag and will be rendered in HTML without escaping. Tag and attribute names are not limited to current HTML tags, so custom tags (and even, say, DocBook tags) may be used.

Here is the grammar for tags:

A [tag name](#tag-name) consists of an ASCII letter followed by zero or more ASCII letters, digits, or hyphens (`-`).

An [attribute](#attribute) consists of [whitespace](#whitespace), an [attribute name](#attribute-name), and an optional [attribute value specification](#attribute-value-specification).

An [attribute name](#attribute-name) consists of an ASCII letter, `_`, or `:`, followed by zero or more ASCII letters, digits, `_`, `.`, `:`, or `-`. (Note: This is the XML specification restricted to ASCII. HTML5 is laxer.)

An [attribute value specification](#attribute-value-specification) consists of optional [whitespace](#whitespace), a `=` character, optional [whitespace](#whitespace), and an [attribute value](#attribute-value).

An [attribute value](#attribute-value) consists of an [unquoted attribute value](#unquoted-attribute-value), a [single-quoted attribute value](#single-quoted-attribute-value), or a [double-quoted attribute value](#double-quoted-attribute-value).

An [unquoted attribute value](#unquoted-attribute-value) is a nonempty string of characters not including spaces, `"`, `'`, `=`, `<`, `>`, or `````.

A [single-quoted attribute value](#single-quoted-attribute-value) consists of `'`, zero or more characters not including `'`, and a final `'`.

A [double-quoted attribute value](#double-quoted-attribute-value) consists of `"`, zero or more characters not including `"`, and a final `"`.

An [open tag](#open-tag) consists of a `<` character, a [tag name](#tag-name), zero or more [attributes](#attribute), optional [whitespace](#whitespace), an optional `/` character, and a `>` character.

A [closing tag](#closing-tag) consists of the string `</`, a [tag name](#tag-name), optional [whitespace](#whitespace), and the character `>`.

An [HTML comment](#html-comment) consists of `<!--` + _text_ + `-->`, where _text_ does not start with `>` or `->`, does not end with `-`, and does not contain `--`. (See the [HTML5 spec](http://www.w3.org/TR/html5/syntax.html#comments).)

A [processing instruction](#processing-instruction) consists of the string `<?`, a string of characters not including the string `?>`, and the string `?>`.

A [declaration](#declaration) consists of the string `<!`, a name consisting of one or more uppercase ASCII letters, [whitespace](#whitespace), a string of characters not including the character `>`, and the character `>`.

A [CDATA section](#cdata-section) consists of the string `<![CDATA[`, a string of characters not including the string `]]>`, and the string `]]>`.

An [HTML tag](#html-tag) consists of an [open tag](#open-tag), a [closing tag](#closing-tag), an [HTML comment](#html-comment), a [processing instruction](#processing-instruction), a [declaration](#declaration), or a [CDATA section](#cdata-section).

Here are some simple open tags:

[Example 582](#example-582)

```md
<a><bab><c2c>
```

```html
<p><a><bab><c2c></p>
```

Empty elements:

[Example 583](#example-583)

```md
<a/><b2/>
```

```html
<p><a/><b2/></p>
```

[Whitespace](#whitespace) is allowed:

[Example 584](#example-584)

```md
<a/><b2
data="foo">
```

```html
<p><a/><b2
data="foo"></p>
```

With attributes:

[Example 585](#example-585)

```md
<afoo="bar"bam='baz<em>"</em>'
_booleanzoop:33=zoop:33/>
```

```html
<p><afoo="bar"bam='baz<em>"</em>'
_booleanzoop:33=zoop:33/></p>
```

Custom tag names can be used:

[Example 586](#example-586)

```md
Foo<responsive-imagesrc="foo.jpg"/>
```

```html
<p>Foo<responsive-imagesrc="foo.jpg"/></p>
```

Illegal tag names, not parsed as HTML:

[Example 587](#example-587)

```md
<33><__>
```

```html
<p>&lt;33&gt;&lt;__&gt;</p>
```

Illegal attribute names:

[Example 588](#example-588)

```md
<ah*#ref="hi">
```

```html
<p>&lt;ah*#ref=&quot;hi&quot;&gt;</p>
```

Illegal attribute values:

[Example 589](#example-589)

```md
<ahref="hi'><ahref=hi'>
```

```html
<p>&lt;ahref=&quot;hi'&gt;&lt;ahref=hi'&gt;</p>
```

Illegal [whitespace](#whitespace):

[Example 590](#example-590)

```md
<a><
foo><bar/>
```

```html
<p>&lt;a&gt;&lt;
foo&gt;&lt;bar/&gt;</p>
```

Missing [whitespace](#whitespace):

[Example 591](#example-591)

```md
<ahref='bar'title=title>
```

```html
<p>&lt;ahref='bar'title=title&gt;</p>
```

Closing tags:

[Example 592](#example-592)

```md
</a></foo>
```

```html
<p></a></foo></p>
```

Illegal attributes in closing tag:

[Example 593](#example-593)

```md
</ahref="foo">
```

```html
<p>&lt;/ahref=&quot;foo&quot;&gt;</p>
```

Comments:

[Example 594](#example-594)

```md
foo<!--thisisa
comment-withhyphen-->
```

```html
<p>foo<!--thisisa
comment-withhyphen--></p>
```

[Example 595](#example-595)

```md
foo<!--notacomment--twohyphens-->
```

```html
<p>foo&lt;!--notacomment--twohyphens--&gt;</p>
```

Not comments:

[Example 596](#example-596)

```md
foo<!-->foo-->

foo<!--foo--->
```

```html
<p>foo&lt;!--&gt;foo--&gt;</p>
<p>foo&lt;!--foo---&gt;</p>
```

Processing instructions:

[Example 597](#example-597)

```md
foo<?phpecho$a;?>
```

```html
<p>foo<?phpecho$a;?></p>
```

Declarations:

[Example 598](#example-598)

```md
foo<!ELEMENTbrEMPTY>
```

```html
<p>foo<!ELEMENTbrEMPTY></p>
```

CDATA sections:

[Example 599](#example-599)

```md
foo<![CDATA[>&<]]>
```

```html
<p>foo<![CDATA[>&<]]></p>
```

Entity and numeric character references are preserved in HTML attributes:

[Example 600](#example-600)

```md
foo<ahref="&ouml;">
```

```html
<p>foo<ahref="&ouml;"></p>
```

Backslash escapes do not work in HTML attributes:

[Example 601](#example-601)

```md
foo<ahref="\*">
```

```html
<p>foo<ahref="\*"></p>
```

[Example 602](#example-602)

```md
<ahref="\"">
```

```html
<p>&lt;ahref=&quot;&quot;&quot;&gt;</p>
```

## 6.9  Hard line breaks

A line break (not in a code span or HTML tag) that is preceded by two or more spaces and does not occur at the end of a block is parsed as a [hard line break](#hard-line-break) (rendered in HTML as a `<br />` tag):

[Example 603](#example-603)

```md
foo
baz
```

```html
<p>foo<br/>
baz</p>
```

For a more visible alternative, a backslash before the [line ending](#line-ending) may be used instead of two spaces:

[Example 604](#example-604)

```md
foo\
baz
```

```html
<p>foo<br/>
baz</p>
```

More than two spaces can be used:

[Example 605](#example-605)

```md
foo
baz
```

```html
<p>foo<br/>
baz</p>
```

Leading spaces at the beginning of the next line are ignored:

[Example 606](#example-606)

```md
foo
bar
```

```html
<p>foo<br/>
bar</p>
```

[Example 607](#example-607)

```md
foo\
bar
```

```html
<p>foo<br/>
bar</p>
```

Line breaks can occur inside emphasis, links, and other constructs that allow inline content:

[Example 608](#example-608)

```md
*foo
bar*
```

```html
<p><em>foo<br/>
bar</em></p>
```

[Example 609](#example-609)

```md
*foo\
bar*
```

```html
<p><em>foo<br/>
bar</em></p>
```

Line breaks do not occur inside code spans

[Example 610](#example-610)

```md
`code
span`
```

```html
<p><code>codespan</code></p>
```

[Example 611](#example-611)

```md
`code\
span`
```

```html
<p><code>code\span</code></p>
```

or HTML tags:

[Example 612](#example-612)

```md
<ahref="foo
bar">
```

```html
<p><ahref="foo
bar"></p>
```

[Example 613](#example-613)

```md
<ahref="foo\
bar">
```

```html
<p><ahref="foo\
bar"></p>
```

Hard line breaks are for separating inline content within a block. Neither syntax for hard line breaks works at the end of a paragraph or other block element:

[Example 614](#example-614)

```md
foo\
```

```html
<p>foo\</p>
```

[Example 615](#example-615)

```md
foo
```

```html
<p>foo</p>
```

[Example 616](#example-616)

```md
###foo\
```

```html
<h3>foo\</h3>
```

[Example 617](#example-617)

```md
###foo
```

```html
<h3>foo</h3>
```

## 6.10  Soft line breaks

A regular line break (not in a code span or HTML tag) that is not preceded by two or more spaces or a backslash is parsed as a [softbreak](#softbreak). (A softbreak may be rendered in HTML either as a [line ending](#line-ending) or as a space. The result will be the same in browsers. In the examples here, a [line ending](#line-ending) will be used.)

[Example 618](#example-618)

```md
foo
baz
```

```html
<p>foo
baz</p>
```

Spaces at the end of the line and beginning of the next line are removed:

[Example 619](#example-619)

```md
foo
baz
```

```html
<p>foo
baz</p>
```

A conforming parser may render a soft line break in HTML either as a line break or as a space.

A renderer may also provide an option to render soft line breaks as hard line breaks.

## 6.11  Textual content

Any characters not given an interpretation by the above rules will be parsed as plain textual content.

[Example 620](#example-620)

```md
hello$.;'there
```

```html
<p>hello$.;'there</p>
```

[Example 621](#example-621)

```md
Fooχρῆν
```

```html
<p>Fooχρῆν</p>
```

Internal spaces are preserved verbatim:

[Example 622](#example-622)

```md
Multiplespaces
```

```html
<p>Multiplespaces</p>
```

# Appendix: A parsing strategy

In this appendix we describe some features of the parsing strategy used in the CommonMark reference implementations.

## Overview

Parsing has two phases:

1. In the first phase, lines of input are consumed and the block structure of the document—its division into paragraphs, block quotes, list items, and so on—is constructed. Text is assigned to these blocks but not parsed. Link reference definitions are parsed and a map of links is constructed.
2. In the second phase, the raw text contents of paragraphs and headings are parsed into sequences of Markdown inline elements (strings, code spans, links, emphasis, and so on), using the map of link references constructed in phase

At each point in processing, the document is represented as a tree of **blocks**. The root of the tree is a `document` block. The `document` may have any number of other blocks as **children**. These children may, in turn, have other blocks as children. The last child of a block is normally considered **open**, meaning that subsequent lines of input can alter its contents. (Blocks that are not open are **closed**.) Here, for example, is a possible document tree, with the open blocks marked by arrows:

```tree
-> document
  -> block_quote
       paragraph
         "Lorem ipsum dolor\nsit amet."
    -> list (type=bullet tight=true bullet_char=-)
         list_item
           paragraph
             "Qui *quodsi iracundia*"
      -> list_item
        -> paragraph
             "aliquando id"
```

## Phase 1: block structure

Each line that is processed has an effect on this tree. The line is analyzed and, depending on its contents, the document may be altered in one or more of the following ways:

1. One or more open blocks may be closed.
2. One or more new blocks may be created as children of the last open block.
3. Text may be added to the last (deepest) open block remaining on the tree.

Once a line has been incorporated into the tree in this way, it can be discarded, so input can be read in a stream.

For each line, we follow this procedure:

1. First we iterate through the open blocks, starting with the root document, and descending through last children down to the last open block. Each block imposes a condition that the line must satisfy if the block is to remain open. For example, a block quote requires a `>` character. A paragraph requires a non-blank line. In this phase we may match all or just some of the open blocks. But we cannot close unmatched blocks yet, because we may have a [lazy continuation line](#lazy-continuation-line).
2. Next, after consuming the continuation markers for existing blocks, we look for new block starts (e.g. `>` for a block quote). If we encounter a new block start, we close any blocks unmatched in step 1 before creating the new block as a child of the last matched block.
3. Finally, we look at the remainder of the line (after block markers like `>`, list markers, and indentation have been consumed). This is text that can be incorporated into the last open block (a paragraph, code block, heading, or raw HTML).

Setext headings are formed when we see a line of a paragraph that is a [setext heading underline](#setext-heading-underline).

Reference link definitions are detected when a paragraph is closed; the accumulated text lines are parsed to see if they begin with one or more reference link definitions. Any remainder becomes a normal paragraph.

We can see how this works by considering how the tree above is generated by four lines of Markdown:

```md
> Lorem ipsum dolor
sit amet.
> - Qui *quodsi iracundia*
> - aliquando id
```

At the outset, our document model is just

```tree
-> document
```

The first line of our text,

```md
> Lorem ipsum dolor
```

causes a `block_quote` block to be created as a child of our open `document` block, and a `paragraph` block as a child of the `block_quote`. Then the text is added to the last open block, the `paragraph`:

```tree
-> document
  -> block_quote
    -> paragraph
         "Lorem ipsum dolor"
```

The next line,

```md
sit amet.
```

is a "lazy continuation" of the open `paragraph`, so it gets added to the paragraph's text:

```tree
-> document
  -> block_quote
    -> paragraph
         "Lorem ipsum dolor\nsit amet."
```

The third line,

```md
> - Qui *quodsi iracundia*
```

causes the `paragraph` block to be closed, and a new `list` block opened as a child of the `block_quote`. A `list_item` is also added as a child of the `list`, and a `paragraph` as a child of the `list_item`. The text is then added to the new `paragraph`:

```tree
-> document
  -> block_quote
       paragraph
         "Lorem ipsum dolor\nsit amet."
    -> list (type=bullet tight=true bullet_char=-)
      -> list_item
        -> paragraph
             "Qui *quodsi iracundia*"
```

The fourth line,

```md
> - aliquando id
```

causes the `list_item` (and its child the `paragraph`) to be closed, and a new `list_item` opened up as child of the `list`. A `paragraph` is added as a child of the new `list_item`, to contain the text. We thus obtain the final tree:

```tree
-> document
  -> block_quote
       paragraph
         "Lorem ipsum dolor\nsit amet."
    -> list (type=bullet tight=true bullet_char=-)
         list_item
           paragraph
             "Qui *quodsi iracundia*"
      -> list_item
        -> paragraph
             "aliquando id"
```

## Phase 2: inline structure

Once all of the input has been parsed, all open blocks are closed.

We then "walk the tree," visiting every node, and parse raw string contents of paragraphs and headings as inlines. At this point we have seen all the link reference definitions, so we can resolve reference links as we go.

```tree
document
  block_quote
    paragraph
      str "Lorem ipsum dolor"
      softbreak
      str "sit amet."
    list (type=bullet tight=true bullet_char=-)
      list_item
        paragraph
          str "Qui "
          emph
            str "quodsi iracundia"
      list_item
        paragraph
          str "aliquando id"
```

Notice how the [line ending](#line-ending) in the first paragraph has been parsed as a `softbreak`, and the asterisks in the first list item have become an `emph`.

### An algorithm for parsing nested emphasis and links

By far the trickiest part of inline parsing is handling emphasis, strong emphasis, links, and images. This is done using the following algorithm.

When we're parsing inlines and we hit either

* a run of `*` or `_` characters, or
* a `[` or `![`

we insert a text node with these symbols as its literal content, and we add a pointer to this text node to the [delimiter stack](#delimiter-stack).

The [delimiter stack](#delimiter-stack) is a doubly linked list. Each element contains a pointer to a text node, plus information about

* the type of delimiter (`[`, `![`, `*`, `_`)
* the number of delimiters,
* whether the delimiter is "active" (all are active to start), and
* whether the delimiter is a potential opener, a potential closer, or both (which depends on what sort of characters precede and follow the delimiters).

When we hit a `]` character, we call the _look for link or image_ procedure (see below).

When we hit the end of the input, we call the _process emphasis_ procedure (see below), with `stack_bottom` = NULL.

#### _look for link or image_

Starting at the top of the delimiter stack, we look backwards through the stack for an opening `[` or `![` delimiter.

* If we don't find one, we return a literal text node `]`.
* If we do find one, but it's not _active_, we remove the inactive delimiter from the stack, and return a literal text node `]`.
* If we find one and it's active, then we parse ahead to see if we have an inline link/image, reference link/image, compact reference link/image, or shortcut reference link/image.
  - If we don't, then we remove the opening delimiter from the delimiter stack and return a literal text node `]`.
  - If we do, then
    + We return a link or image node whose children are the inlines after the text node pointed to by the opening delimiter.
    + We run _process emphasis_ on these inlines, with the `[` opener as `stack_bottom`.
    + We remove the opening delimiter.
    + If we have a link (and not an image), we also set all `[` delimiters before the opening delimiter to _inactive_. (This will prevent us from getting links within links.)

#### _process emphasis_

Parameter `stack_bottom` sets a lower bound to how far we descend in the [delimiter stack](#delimiter-stack). If it is NULL, we can go all the way to the bottom. Otherwise, we stop before visiting `stack_bottom`.

Let `current_position` point to the element on the [delimiter stack](#delimiter-stack) just above `stack_bottom` (or the first element if `stack_bottom` is NULL).

We keep track of the `openers_bottom` for each delimiter type (`*`, `_`). Initialize this to `stack_bottom`.

Then we repeat the following until we run out of potential closers:

* Move `current_position` forward in the delimiter stack (if needed) until we find the first potential closer with delimiter `*` or `_`. (This will be the potential closer closest to the beginning of the input – the first one in parse order.)
* Now, look back in the stack (staying above `stack_bottom` and the `openers_bottom` for this delimiter type) for the first matching potential opener ("matching" means same delimiter).
* If one is found:
  - Figure out whether we have emphasis or strong emphasis: if both closer and opener spans have length >= 2, we have strong, otherwise regular.
  - Insert an emph or strong emph node accordingly, after the text node corresponding to the opener.
  - Remove any delimiters between the opener and closer from the delimiter stack.
  - Remove 1 (for regular emph) or 2 (for strong emph) delimiters from the opening and closing text nodes. If they become empty as a result, remove them and remove the corresponding element of the delimiter stack. If the closing node is removed, reset `current_position` to the next element in the stack.
* If none in found:
  - Set `openers_bottom` to the element before `current_position`. (We know that there are no openers for this kind of closer up to and including this point, so this puts a lower bound on future searches.)
  - If the closer at `current_position` is not a potential opener, remove it from the delimiter stack (since we know it can't be a closer either).
  - Advance `current_position` to the next element in the stack.

After we're done, we remove all delimiters above `stack_bottom` from the delimiter stack.
