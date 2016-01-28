
# PDF alternative using HTML (ZIP/GZIP)

Sometimes you need create documents that can be emailed, printed, archived, etc.

Some examples include:

- Invoices.
- Terms and conditions.
- Contracts.
- Reports, assessments, statistics.
- Bank statements.

These are often created as **PDF** files, as they:

- Ensure a consistent visual representation (looks the same).
- Fairly small file size.
- Contains everything they need in the file (e.g. custom fonts).
- Can be read offline, and be archived.
- Not really built for easy editing.
- Do not cause many security problems (ish).

Unfortunately PDF's do have some problems:

1. Do not work well on small screens (e.g. an A4 document will often cause horizontal/vertical scrolling).
2. Requires custom software to convert the source document into a PDF file (most systems are ok at creating HTML, but this is difficult to convert to PDF).
3. Content can be tagged for assistive devices, but this is rarely used (e.g. this text is a heading).
4. Cannot really change the font size (different to zooming in).
5. Cannot really change the font itself (e.g. applying the [OpenDyslexic](http://opendyslexic.org/) font).
6. Cannot really change the colours for fonts/backgrounds (e.g. colour contrast).
7. Often you cannot copy/paste, or search the text in the document (e.g. when you have 2 columns of text, or when every character has been individually placed on the canvas).
8. Positioning of elements is difficult, as it's done from the bottom left of the canvas (so font size, and text wrapping needs to be considered).

HTML can fix these problems:

1. Can use CSS and media queries to work well on small and large screens.
2. If you're starting with HTML, you don't need to do anything else.
3. Developers are already conditioned (most of the time) to use semantic elements such as H1-6, Paragraphs, etc.
4. Some browsers allow you to change the font size (e.g. Firefox).
5. Most browsers have extensions for users to easily change the font in use (e.g. [OpenDyslexic](http://opendyslexic.org/get-it-free/)).
6. Extensions can also fix colour contrast issues.
7. Browsers are much at supporting copy/paste, or for the content to be indexed (e.g. searching).
8. Positioning of elements is rarely done with absolute units, instead other CSS layout features are used (e.g. flexbox).

But we can't just email a HTML document to someone:

- It can request any other file on the computer (although some browsers do have some protections against this).
- It can request files from the internet, allowing the author to retrive content from their server (i.e. content can be changed).
- It can contain tracking code (JavaScript, images, etc) that allow the author to track when the file is opened (and possibly other events).
- If you email the document, you cannot keep resources (like images) as separate files, they will need to be inlined (difficult, and can cause encoding problems).

An alternative to HTML is MHTML, but this is pretty much the same (where the file is formatted like an email). This means you still have the same problems, but it will also need a program to parse the source HTML, include/encode the required resources, and update the HTML to point to those resources.

Another alternative is DOCX (MS Office), or OpenDocument, but these have their own problems:

- Typically seen as something that is editable, and often opened in a word processing program.
- Use their own specifications, which are fairly difficult to read/understand for developers who are typically used to HTML.

Then there is EPUP:

- These documents are often seen as publications (e.g. books), that you download and store in your e-reader.
- Works on the assumption that you want multi-paged content.
- Needs a [Table of Contents](http://www.idpf.org/epub/30/spec/epub30-contentdocs.html#sec-xhtml-nav-def-types-toc), and ideally a [cover-image](http://www.idpf.org/epub/30/spec/epub30-publications.html#sec-item-property-values).
- Requires a special program to package up the document (e.g. the [IDPF Java program](https://github.com/IDPF/epub3-samples)).
- Contains a fair amount of meta data (e.g. a list of all the XHTML documents in the OPF `<manifest>` and `<spine>`).
- It can allow [remote content](http://www.idpf.org/epub/30/spec/epub30-publications.html#sec-resource-locations), but this is disabled by default.

And finally PWP, which is related to EPUB:

- Documents are referenced from a [central URL](https://www.w3.org/TR/pwp/#identification), and uses the Open Web *Platform* to distribute the content.
- This allows you to refer to the publications canonical source.
- The copy you download can be [stored for offline use](https://www.w3.org/TR/pwp/#package), but new versions can be pulled from the source URL (not good for Terms and Conditions, but useful for authors to correct typos, and genrally keeping the document up to date).

---

# A possible solution

## Manually creating a new file

A developer creates an `index.html` file in a folder, and includes the needed resources (e.g. images/css).

They then use ZIP/GZIP to package it:

	zip -r ./my-file.hdoc ./my-file/

And they're done :-)

## Automatically creating a new file

A Word Processor can just save to this file format, like they do with PDFs.

This can also be used by browsers in their "Save Web Page as" feature. Most browsers only provide the option to save the "HTML only" (no resources), or to put the resources into a folder next to the HTML file (which is often lost later).

## Using the file format

The file will open in the users preferred web browser... just double click :-)

Or it could be viewed within email clients in a simular way.

The file can be downloaded from a web site, sent via email, stored on a USB drive (just like any other file).

## The browsers

Most of the Open Web *Technology* is already in place for this to work.

The browsers will need to get to the `index.html` from the ZIP (which may be password protected), and any additional resources (also from the ZIP).

FireFox does have some of this in place, which looks roughly like:

	jar:file:///.../<strong>my-file.hdoc</strong>!/index.html

The browser will also need to enforce a [Content Security Policy](https://en.wikipedia.org/wiki/Content_Security_Policy), one that blocks outbound connections, something like:

	default-src 'none';
	style-src 'self';
	font-src 'self';
	script-src 'self';
	img-src 'self';
	referrer no-referrer;
	frame-ancestors 'none'

I know, I'm not going to get away with script-src without [unsafe-inline](https://www.w3.org/TR/CSP/#directive-script-src), but I can still hope :-)

The browser will have to block resources that are not in the ZIP file, some of which can be done with the sub-origin proposals.

Finally the browser will have to block access to local storage (inc cookies), so each time the document is opened, it's like being opened for the first time.

That's not to say that the browser (not the JavaScript in the document) couldn't do things like remember annotations for the user.

It might also have to block JavaScript from accessing the current date/time, so we don't have content that changes after a certain point in time (keep in mind those legal documents).

## Security

The most important thing about this file format is *security*.

We need IT departments to feel very comfortable allowing them onto their network.

And for virus/spam filters not to rely on huristics to determine if the file contains malicious code (like they do looking for macros in MS Word documents).

---

# Miscellaneous

You will notice this is a similar approach that Microsoft Word did with the docx format, Java with JAR, PHP with PHAR, etc.

It should be possible to include more than one HTML file, if you want multiple pages within the document.

Maybe it can support multiple languages... or just use `lang="en"` and some CSS to show/hide?

The `hdoc` extension was stolen from [hdoc.crzt.fr](http://hdoc.crzt.fr/www/co/hdoc.html), which is similar to EPUB.

## Bug/feature requests

- [Chrome](https://crbug.com/575677)
- [Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1237990)
- [Edge](https://wpdev.uservoice.com/forums/257854-microsoft-edge-developer/suggestions/11443002-webpage-zip-as-alternative-to-pdf)

## Mailing lists

- [W3C Public-DigiPub-IG](https://lists.w3.org/Archives/Public/public-digipub-ig/2016Jan/0089.html)
- [W3C Public-WebAppSec](https://lists.w3.org/Archives/Public/public-webappsec/2016Jan/0063.html)
