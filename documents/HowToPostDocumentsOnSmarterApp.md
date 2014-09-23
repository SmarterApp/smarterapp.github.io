---
title: How to Post Documents on SmarterApp.org
dateCreated: 2014-09-04 
date: 2014-09-05
version: v1.0
layout: document
author: Brandt Redd
---
SmarterApp is hosted on GitHub.com using GitHub Pages. Accordingly, SmarterApp benefits from the source code management features of Git and GitHub.

To post updates on SmarterApp you need to be familiar with using Git for code management. If you are not yet familiar with Git, there are numerous resources out on the web. The ["Getting Started" chapter of <em>Pro Git</em>](http://git-scm.com/book/en/Getting-Started) is a particularly good choice. 

GitHub Pages uses Jekyll for site generation, and Markdown (the Kramdown variant) for document markup. You don't need to be familiar with these tools unless you're doing something complicated. Nevertheless, the following references may be helpful:

* [GitHub Pages](https://pages.github.com/)
* [Jekyll](http://jekyllrb.com/docs/home/)
* [Kramdown Syntax](http://kramdown.gettalong.org/syntax.html)

# Installation

In theory, you can do everything you need by using the built-in text editor and file upload capabilities of GitHub. It practice, it's easier to make a local copy, edit things there and then sync with GitHub.

To do so, you need to install is a version of Git that's appropriate for your operating system. [GitHub for Windows](https://windows.github.com/) and [GitHub for Mac](https://mac.github.com/) are good choices.

Many SmarterApp documents use Markdown format. You can edit Markdown documents using any text editor. However, using a Markdown editor can help with syntax and spell checking. This document was written using [MarkdownPad for Windows](http://markdownpad.com/). Mac users might check out [MacDown for OS X](http://macdown.uranusjr.com/)


## Setup

Using Git, make a clone of the SmarterApp.org website repository. The master copy is located at [https://github.com/SmarterApp/smarterapp.github.io](https://github.com/SmarterApp/smarterapp.github.io) . The clone will be located on your local hard drive.

Since this is a public repository, anyone can read the source code of SmarterApp.org and make copies (forks) of it. But only those with write access can post changes back to the live site.

# Framework

Now that you have a clone of the site, you can get a feel for how previous documents have been posted. Documents in their distribution format are in the "documents" directory. The same documents in their original format (if distribution is different) are in the "_original_documents" directory. Abstracts for specification documents are in the "_specs" directory and abstracts for architecture documents are in the "_arch" directory.

## Document Format

**PDF:** Most existing SmarterApp documents are distributed in PDF format. This is the appropriate choice for documents that originate in word processing systems such as Microsoft Word.

**Spreadsheets and Such:** Interactive documents such as spreadsheets are usually posted in their original form (e.g. Microsoft Excel).

**Sample Data:** Sample data is usually collected in a .zip file though sometimes it is posted in its original format (such as .csv or .xml). GitHub is not appropriate for large files, particularly binary encoded files. Files like these must be posted on another host site with links from SmarterApp. The [Sample Assessment Item Package](http://www.smarterapp.org/specs/AssessmentItemPackage.html) is an example of this.

**Markdown:** Over time, we hope to shift to Markdown format for most new documents. That's because Markdown benefits from the version control and change management features of Git and GitHub better than the other formats. The document you're reading is a good example. See the tips below for writing documents in Markdown.

## Document Filenames

Filenames should not have any spaces in them and should not include any date or version information. The URL to access a document will be derived from its filename – hence, no spaces in the name. When accessing a file from it's URL, the latest version should always be delivered. We rely on Git's version control features to preserve previous versions and gain access to them. This is why version numbers or dates should not be included in the filename.

# Posting a Document and Its Abstract

To post a new document or update an existing one, do the following:

## 1. Post the Document

Name the document according to the guidelines above and copy it into the "documents" directory in your clone of "smarterapp.github.io". If this is an update to an existing document then replace the old document with the new. (The version control features of Git will enable access to previously posted versions.)

## 2. Post the Original Document

If a document is being posted in PDF format, the original format (e.g. Microsoft Word) of the document should be given the same name as the PDF (except for the extension) and copied into the "_original_documents" folder. If this is an update to an existing document then replace the old with the new.

## 3. Post the Abstract

The abstract of the document will be manifested in the document listings under [Specifications](http://www.smarterapp.org/specifications.html) or [Architecture](http://www.smarterapp.org/architecture.html). Abstracts for specifications are in the "_specs" directory of "smarterapp.github.io". Abstracts for architecture documents are in the "_arch" directory.

The easiest way to create an abstract is to copy an existing abstract and then update it to reflect the new document. For updates to existing documents, the existing abstract should also be updated, replacing the old abstract.

As with documents, the name of the abstract should **not** contain version numbers or dates. It's a good idea to use the same filename as the document (except for the extension) though this rule has not always been followed in the past.

Abstracts are in Markdown format with [YAML](https://github.com/Animosity/CraftIRC/wiki/Complete-idiot's-introduction-to-yaml) metadata at the beginning. The metadata includes the title, release date,  Here's an example:

    ---
    title: Life, the Universe, and Everything 
    date: 2014-09-04
    docurl: /documents/UniversalAnswer.pdf
    status: Release
    version: v42.0
    ---
    An API for submitting queries to the Deep Thought computer. This is
    an asynchronous API as the answers to queries may not be returned
    immediately.

*Title* is self-explanatory; *date* should be the date that the document was approved or released.

The *docurl* should be the URL for the document you posted in step 1. If the document is posted on SmarterApp.org then it should be relative to the root of the site (as in the above example). If the document is posted elsewhere on the web (as with large sample data files) then the URL should be absolute.

Typical values for *status* are: *preview*, *draft*, *review*, and *release*.

*Version* should be set to the version number of the document itself. Typically, pre-release documents have values between 0.1 and 0.9 while released documents have versions equal to or grater than 1.0.

The body of the document is usually plain text. You can add markup such as bold, underline, hypertext links and so forth using either Markdown syntax or HTML.

*As of this writing (September 2014) there is a bug in Jekyll that prevents Markdown syntax from being interpreted in the document listings (e.g. [Specifications](http://www.smarterapp.org/specifications.html)). Until this gets fixed, you should use HTML markup in abstracts. An example is [Smarter Balanced Adaptive Algorithm](http://www.smarterapp.org/specs/AdaptiveAlgorithm.html) which includes a link.*

## Commit the Files to GitHub

Once the document is in place and the abstract is ready, use Git to commit the documents to your local repository and then sync your local repository with GitHub. It takes Jekyll a few seconds to process your updates and then they'll appear on [www.SmarterApp.org](http://www.smarterapp.org).

# Additional Tips

The following information is not essential for posting documents but may come in handy when doing more advanced work.

## Writing Documents in Markdown

This document itself is a good example of how to write SmarterApp documents in Markdown format. You can look at the source version in the GitHub repository to see how the metadata, headings and links are designated.

A document should start with YAML metadata followed by the body of the text. Here's an example:

    ---
    title: My awesome document 
    date: 2014-09-04
    version: v1.0
    layout: document
    ---
    Some introductory text.

    # A level one heading
    ## A level two heading.

When using Markdown, the original document (with a ".md" extension) should be posted in the "documents" directory. When Jekyll processes the document it will convert it into HTML and add layout information such as the Title section. Accordingly, the link from the abstract should use an ".html" extension.

## Previewing Your Changes using Jekyll

If you want to preview your additions to the SmarterApp website before posting them live, you have two options: You can install a local copy of Jekyll or you can create a fork of the original repository.

To install a copy of Jekyll on your local computer, [follow the instructions here.](https://help.github.com/articles/using-jekyll-with-pages)

By default, Jekyll will render your site to the "_site" directory. This directory should *not* be checked into Git or GitHub so edit your ".gitignore" file to exclude it from Git's attention.

To create a fork, read on...

## Forking SmarterApp.org

Creating a fork of SmarterApp.org will let you play around with your own copy, commit changes, see how things look on GitHub Pages and then, when you're ready, submit the changes to the primary SmarterApp.org repository with a "pull request". You can do this even if you don't have write privileges to SmarterApp.org. In that case, someone with write privileges can review the changes in your pull request and fold them into the live copy.

Before creating a fork, you need to decide where the forked repository will go. Any user or organization can have a special repository that will be manifest in GitHub pages. For example, of a user named "fred" creates a repository named "fred.github.io" then that repository will be rendered in GitHub Pages at http://fred.github.io . Likewise, if an organization named "acme" creates a repository named "acme.github.io" it will be rendered in GitHub Pages at http://acme.github.io". So, if you don't already have a GitHub Pages repository associated with your account, the easiest thing will be to use that. Otherwise, you can create an organization on GitHub just for this purpose.

(You can also create github pages for any repository by creating a branch called "gh-pages". However, that approach is more complicated.)

In the following instructions, substitute your user or organization name for "fred". To create a fork, do the following:

1. Navigate to the [SmarterApp.github.io](https://github.com/SmarterApp/smarterapp.github.io) repository and click the "fork" button in the upper right.
2. Specify "fred" as the destination of your fork.
3. Now, navigate to your new fork.
4. Delete the CNAME file (otherwise it will try and fail to publish your version at http://www.smarterapp.org).
5. In the _config.yml file change *domain* from "www.smarterapp.org" to "fred.github.io".
6. Commit the above changes.
7. Then rename the new fork from "smarterapp.github.io" to "fred.github.io".
8. Wait about 10 minutes and the fork will be available at http://fred.github.io .

Now you can make any changes, commit them to your forked repository and see if they all look correct. As soon as you are confident in the results, [follow these instructions](https://help.github.com/articles/creating-a-pull-request) to create a pull request.
