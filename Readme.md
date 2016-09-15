# Controlling Pandoc Using Rake

This directory contains a `Rakefile` that can compile your manuscript for you, using [Rake](http://rake.rubyforge.org/), the Ruby Automated Build System.

Rake allows users to run predefined "tasks" that automate certain repetitive command lines. In this document, you will find the directory structure this Rakefile expects, some notes on using YAML metadata to title your chapters, and a list of tasks you can run using Rake.

## Quick Start

If you want to quickly get started with this Rakefile, you need to install ruby (should be included on OSX and Linux) andinstall Rake (run `gem install rake`). Then, follow these quick points:

1. Chapters should be named starting with numbers and stored in the `chapters/` directory (for example: `my-awesome-book/chapters/01-introduction.md`).
	* The name after the numbers is entirely up to you!
	* I recommend zero padding (adding a leading zero to single digit numbers) for clearer file sorting.
1. Footnotes can be stored in the `my-awesome-book/chapters/notes/` directory and the number must match the chapter the notes are associated with (for example: `my-awesome-book/chapters/notes/01-notes.md`).
1. Sources can be exported as a BibLaTeX file from Zotero and stored in a .bib file of the same name as the book's root directory (for example: `my-awesome-book/my-awesome-book.bib`)
1. Compile the whole book by running `rake docx` from the command line
1. Compile individual chapters by running `rake ch1:docx` from the command line.

## Expected Directory Structure

This `Rakefile` expects a fairly minimal, though opinionated, directory structure for writing book manuscripts in Pandoc/Markdown.

### Storing Chapters

Individual chapter files are stored in a directory named `chapters` that contains one Markdown file per book chapter.

**Chapter files must begin with numbers for this Rakefile to recognize them**. Additionally, I find it useful to zero-pad (ie. 1 becomes "01" while 10 remains "10") my chapter numbers to make sure they sort correctly in OSX's Finder. This is optional.

So, for instance, a simple book might have the following directory structure:

* `chapters/`
	* `00-frontmatter.md`
	* `01-introduction.md`
	* `02-chapter1.md`
	* `03-chapter2.md`
	* `99-backmatter.md`
	
What comes after the numbers is entirely arbitrary from the perspective of this Rakefile. Name the files as you will, but **file names must not contain spaces**!

### Writing Related Articles

If you are planning on developing articles from chapters of your book, these can be stored in a directory 

## Writing Your Book

This section will detail a few aspects regarding expected content for each chapter file (mostly dealing with Pandoc's use of YAML metadata).

### Structuring a Chapter

This Rakefile will correctly parse Pandoc metadata (which is written in YAML) within the book. So, for instance, when compiling the whole book, `title` metadata in each chapter will be converted to chapter titles by the Rakefile. When compiling chapters stand-alone, however, the `title` will be retained as the article title.

For instance, a basic chapter file might look like the following:

~~~ markdown
---
title: "Chapter 1: The Journey Begins"
author:
- Software X. User
---

My manuscript begins!
~~~

There is no need to use a header 1 markdown tag (`#`) at the beginning of the file. This Rakefile will handle that for you.

### Titling the Whole Book

One opinionated aspect of this Rakefile to not, the `title` metadata of the first file will be taken as **the title of the entire book**. This is why I generally recommend making use of a frontmatter file.

### Using citations

This Rakefile expects citations to be stored in a BibLaTeX file (such as can be exported from [Zotero](http://zotero.org)). This file can be named the same as the directory your manuscript is stored in. 

For instance, if you were storing your book manuscript in a directory named `my-awesome-book/`, creating a BibLaTex file named `my-awesome-book.bib` within that directory would automatically add citations to your manuscript (assuming you are using Pandoc's citeproc support).

Pandoc's citeproc support expects [csl files](https://github.com/citation-style-language/styles) to be stored in the .csl directory of your `$HOME` directory (so `/Users/username/.csl` on OSX or Linux).

#### Customizing Citations: Different Citation Standards

This Rakefile is configured to use Chicago short note format out of the box. Editing the `Rakefile` in a text editor, change the `$csl` variable at the top of the file to the name of the CSL format you would rather use (for instance "chicago-fullnote-bibliography" for full note support).

#### Customizing Citations: Global Bibliography

If you use a global BibLaTex bibliography, edit the `Rakefile` and uncomment (remove the `#` at the beginning of the line) the defintion of the `$bibliography` variable. Change that variable's value to the full path of your global bibliography.

## Compiling Your Book

There are a number of tasks you can use to compile your book using this Rakefile. These tasks involve performing a fixed set of operations on a number of modes. The modes are ways of interacting with your manuscript and the operations are interactions to be performed.

### Three Modes of Interacting With Your Book

This document defines three modes:

1. As stand-alone articles (see "Writing Related Articles" above)
1. As individual chapters
1. As a whole manuscript

### Operations To Perform On Your Book

This Rakefile can compile your document to a number of file formats supported by Pandoc. To compile them, for the operation passed to Rake (discussed below), you would type the file extension you are compiling to. 

Supported file formats:

* `docx` --- Microsoft Word format
* `pdf` --- PDF (Pandoc requires LaTeX to be installed)
* `epub` --- The ePub format
* `html` --- HTML suitable for posting online

#### Compiling the Whole Manuscript

To compile the entire manuscript, run `rake <operation name>` where `<operation name>` is replaced with the file format you want your book compiled as. For example: `rake docx` for MS Word.

#### Compiling Individual Chapters

To compile an individual chapter, run `rake ch<chapter number>:<operation name>` where `<chapter number>` is the number of your chapter file minus any zero padding and `<operation name>` is replaced with the file format you want your book compiled as. For example: `rake ch1:docx` to compile a file named `chapters/01-introduction.md` to MS Word.

#### Compiling Stand-alone Articles

To compile a stand-alone article, run `rake articles:<article tag>:<operation name>` where `<article tag>` is the name of the file the article is stored in and `<operation name>` is replaced with the file format you want your book compiled as. For example: `rake articles:my-article:docx` to compile a file named `articles/my-article.md` to MS Word.