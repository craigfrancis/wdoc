
# PDF Alternative Using HTML (ZIP/GZIP)

Create a ZIP file, which contains an `index.html` and any other assets (e.g. images), and change the extension to ".wdoc"

Browsers then open this file, and display it like a web page.

Unlike **PDF**, **EPUB**, and **MHTML**, this format is *very* easy to create, and the browser *must* apply some extra restrictions on what the document can do (for security reasons, and so users can trust it).

This might be possible as a [WebPackage](https://github.com/WICG/webpackage/issues/576), so long as network access can be blocked.

---

# Justification

Sometimes you need create documents that can be emailed, printed, archived, etc.

Some examples include:

- Invoices.
- Terms and conditions.
- Contracts.
- Reports, assessments, statistics.
- Bank statements.
- Archiving a webpage.

These are often created as **PDF** files, as they:

- Ensure a consistent visual representation (looks the same).
- Fairly small file size.
- Contains everything they need in the file (e.g. custom fonts).
- Can be read offline, and be archived.
- Not built for easy editing.
- Usually good for security (excluding issues in PDF reader software).

Unfortunately **PDF**'s do have some problems:

1. Do not work well on small screens (e.g. an A4 document will often cause horizontal/vertical scrolling).
2. Requires custom software to convert the source document into a PDF file (most systems are ok at creating HTML, but this is difficult to convert to PDF).
3. Content *can* be tagged for assistive devices (e.g. screen readers), but because it isn't automatically applied, it is rarely used (e.g. identifying heading text), leaving the document less accessible.
4. The user cannot easily change the font size (different to zooming in, in that the line length does not change, resulting in horizontal scrolling).
5. The user cannot easily change the font itself (e.g. applying the [OpenDyslexic](http://opendyslexic.org/) font).
6. The user cannot easily change the colours for fonts/backgrounds (e.g. colour contrast).
7. Often you cannot copy/paste, or search the text in the document (e.g. when you have 2 columns of text, or when every character has been individually placed on the canvas).
8. When authoring documents, the positioning of elements is done from the bottom left of the canvas, so the vertical positioning needs to consider the amount of text the element contains, its font size, how the text wraps, etc.

**HTML** can fix these problems:

1. Can use CSS and media queries to work well on small and large screens.
2. If you're starting with an HTML document, you don't need to do anything else.
3. Developers are already conditioned (most of the time) to use semantic elements such as H1-6, Paragraphs, etc.
4. Some browsers allow you to change the font size (e.g. Firefox).
5. Most browsers have extensions for users to easily change the font in use (e.g. [OpenDyslexic](http://opendyslexic.org/get-it-free/)).
6. Extensions can also fix colour contrast issues.
7. Browsers are much better at supporting copy/paste, and allowing the content to be indexed (e.g. searching).
8. When authoring documents, the positioning of elements in HTML is rarely done with absolute positioning, as CSS provides better layout features (e.g. flexbox), which is generally easier and more intuitive.

But we can't just email a **HTML** document to someone because:

- It can request any other file on the computer (although some browsers do have some protections against this).
- It can request files from the internet, allowing the author to retrieve content from their server (meaning content can be changed, which is not good for legal documents, e.g. invoices, terms and conditions, etc).
- It can contain tracking code (JavaScript, images, etc) that allow the author to track when the file is opened (and possibly other events).
- If you email the document, you cannot keep resources (like images) as separate files, they will need to be inlined (difficult, and can cause encoding problems).

An alternative to **HTML** is **MHTML**, but this is pretty much the same (where the file is [formatted like an email](./alternatives/mhtml/hello-world.mht)). This means you still have the same problems, but it will also need a program to parse the source HTML, include/encode the required resources, and update the HTML to point to those resources.

Another alternative is **DOCX** (MS Office), or **OpenDocument**, but these have their own problems:

- Perceived by the end user as easily editable, and often opened in a word processing program to reinforce this idea.
- Use their own specifications, which are fairly difficult to read/understand for developers who are typically used to HTML.

Then there is **EPUB**:

- These documents are often seen as publications (e.g. books), that you download and store in your e-reader. This is not appropriate for many documents that PDFs are currently used for, such as contracts.
- Works on the assumption that you want multi-paged content, for example, most e-readers (on a large screen) will present the document with 2 pages, resembling a book layout.
- Requires a [Table of Contents](http://www.idpf.org/epub/30/spec/epub30-contentdocs.html#sec-xhtml-nav-def-types-toc).
- Requires a special program to package up the document (e.g. the [IDPF Java program](https://github.com/IDPF/epub3-samples)).
- Contains a fair amount of meta data (e.g. a list of all the XHTML documents in the OPF `<manifest>` and `<spine>`), which is extra work for developers to maintain, for little to no value in some types of documents.
- It can allow [remote content](http://www.idpf.org/epub/30/spec/epub30-publications.html#sec-resource-locations), which is not appropriate for the documents listed above.

And finally **PWP**, which is related to **EPUB**:

(For those unfamiliar with the format, these documents are referenced from a [central URL](https://www.w3.org/TR/pwp/#identification), and uses the Open Web *Platform* to distribute the content. This allows you to refer to the publications canonical source).

This format allows you to download the publication, and [store it for offline use](https://www.w3.org/TR/pwp/#package). But new versions can be pulled from the source URL (not good for Terms and Conditions, Invoices etc). And it shares the same issues as the EPUB format.

---

# A Possible Solution

Start with an `index.html` file in a folder, and include the needed resources (e.g. images/css).

Then use ZIP/GZIP to package it:

	zip -r ./example.wdoc ./example/

And it's done.

## End User's Experience

A Word Processor could just save to this file format. From a users point of view, this is the same as exporting to a PDF file.

This should be fairly easy to implement, as most word processors already have the ability to save as a HTML file.

This file format can also be used by browsers in their "Save Web Page as" feature. Most browsers only provide the option to save the "HTML only" (no resources), or to put the resources into a folder next to the HTML file (which can be lost by the user later).

## Using The File Format

The file would open in the user's preferred web browser... just double click.

Or it could be viewed within email clients in a similar way.

The file could be downloaded from a web site, sent via email, stored on a USB drive (just like any other file).

## The Browsers

Most of the Open Web *Technology* is already in place for this to work.

The browsers will need to get to the `index.html` from the ZIP (which the author may password protect, like a PDF), and any additional resources (also from the ZIP).

So this file can be referenced like any local file:

	file:///tmp/example.wdoc

And the resources within it can be referenced with:

	<img src="./img/logo.jpg" />

	  file:///tmp/example.wdoc/img/logo.jpg

	<link rel="stylesheet" href="/css/styles.css" />

	  file:///tmp/example.wdoc/css/styles.css

	<a href="../../../../../../page2.html">Page 2</a>

	  file:///tmp/example.wdoc/page2.html

Noting that the last two still cannot reference anything outside of the ZIP file, in the same way as resources are currently handled on websites.

Firefox already has some of this in place with the `jar` protocol:

	jar:file:///tmp/example.wdoc!/index.html

The browser can also use the existing functionality for [Content Security Policies](https://en.wikipedia.org/wiki/Content_Security_Policy), to block outbound connections, and block resources that are not in the ZIP file. If implemented on a website, it would look something like:

	default-src 'none';
	style-src 'self';
	font-src 'self';
	script-src 'self';
	img-src 'self';
	referrer no-referrer;
	frame-ancestors 'none'

For security reasons the browser should not include [unsafe-inline](https://www.w3.org/TR/CSP/#directive-script-src) in the `script-src`, just for following good programming practices ([example](./demonstrations/inline-javascript/index.html)). However if it was included, it would not cause any security problems, as the rest of the document is sandboxed anyway.

It could also be argued that it should block JavaScript from accessing the current date/time, so we don't have content that changes after a certain point in time (keeping legal documents in mind).

As to limiting the resources to the ZIP file, the logic needed will also work with the [sub-origin](https://w3c.github.io/webappsec-suborigins/) proposal.

Finally the browser would have to block access to local storage (including cookies), so each time the document is opened, it's like being opened for the first time. Also possible to implement with existing functionality (e.g. private browsing mode).

That's not to say that the browser (not the JavaScript in the document) couldn't do things like remember annotations for the user.

## Security

The most important thing about this file format is *security*.

We need IT departments to feel very comfortable allowing them onto their network.

And for virus/spam filters not to rely on heuristics to determine if the file contains malicious code (like they do when looking for macros in MS Word documents).

---

# Miscellaneous

You will notice that the idea of packaging files up into a ZIP file is a similar approach that Microsoft Word did with the docx format, Java with JAR, PHP with PHAR, etc.

It should be possible to include more than one HTML file, if you want multiple pages within the document.

It has the potential to support multiple languages... or perhaps just use `lang="en"` and some CSS to show/hide.

The example file extension is **wdoc** (web document), but could be anything.

## Other alternatives

These file formats are very similar to this proposal, but do not include the necessary restrictions mentioned above:

- [MAFF](http://maf.mozdev.org/), also a ZIP file, with some [extra meta data](./alternatives/maff/hello-world/).
- [Webarchive](https://en.wikipedia.org/wiki/Webarchive), based on the binary plist format, introduced with Safari.
- [WAR](https://docs.oracle.com/javaee/7/tutorial/packaging003.htm), a JAR file for distributing a web application.
- [ARC](http://archive.org/web/researcher/ArcFileFormat.php) and [WARC](http://archive-access.sourceforge.net/warc/), formats for aggregate web page archival.

## Feature Requests

- [Chrome](https://crbug.com/575677)
- [Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1237990)
- [Edge](https://wpdev.uservoice.com/forums/257854-microsoft-edge-developer/suggestions/11443002-webpage-zip-as-alternative-to-pdf)
- [Safari](https://bugs.webkit.org/show_bug.cgi?id=153953)

## Mailing Lists

- [W3C Public-DigiPub-IG](https://lists.w3.org/Archives/Public/public-digipub-ig/2016Jan/0089.html)
- [W3C Public-WebAppSec](https://lists.w3.org/Archives/Public/public-webappsec/2016Jan/0063.html)
