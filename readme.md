
# PDF alternative using HTML (ZIP/GZIP)

Sometimes you need create documents that can be emailed, printed, archived, etc.

Some examples include:

- Invoices.
- Terms and conditions.
- Contracts.
- Reports, assessments, statistics.
- Bank statements.

These are often created as PDF files, as they:

- Ensure a consistent visual representation (looks the same).
- Fairly small file size.
- Contains everything they need in the file (e.g. custom fonts).
- Can be read offline, and be archived.
- Not really built for easy editing.
- Do not cause many security problems (ish).

Unfortunately PDF's do have some problems:

1. Do not work well on small screens (e.g. an A4 document will often cause horizontal/vertical scrolling).
2. The content can be tagged for assistive devices, this is rarely used (e.g. this text is a heading).
3. Cannot really change the font size (different to zooming in).
4. Cannot really change the font itself (e.g. applying the [OpenDyslexic](http://opendyslexic.org/) font).
5. Cannot really change the colours for fonts/backgrounds (e.g. colour contrast).
6. Often you cannot copy/paste, or search the text in the document (e.g. when you have 2 columns of text, or when every character has been individually placed on the canvas).
7. Positioning of elements is difficult, as it's done from the bottom left of the canvas (so font size, and text wrapping needs to be considered).
8. Requires custom software to convert the source document into a PDF file (most systems are ok at creating HTML, but this is difficult to convert to PDF).

So if we instead look at HTML, this can fix these problems:

1. Can use CSS and media queries to work well on small and large screens.
2. Developers are already conditioned (some of the time) to use semantic elements such as H1-6, Paragraphs, etc.
3. Some browsers allow you to change the font size (e.g. Firefox).
4. Most browsers have extensions for users to easily change the font in use (e.g. [OpenDyslexic](http://opendyslexic.org/get-it-free/)).
5. Extensions can also fix colour contrast issues.
6. While still not perfect, browsers have a much better chance at allowing copy/paste, or for the content to be indexed (e.g. searching).
7. Positioning of elements is rarely done with absolute positioning, instead using other CSS features for layout (e.g. flexbox).
8. If you're starting with HTML, you don't need to do anything else.

But we can't just email a HTML document to someone:

- The HTML can reference any other file on the computer (although some browsers do have some protections against this).
- The HTML can reference files from the internet, allowing the author to retrive content from their server (i.e. content can be changed).
- The page can contain tracking code (JavaScript, images, etc) that allow the author to track when the file is opened (and possibly other events).
- If you email the document, you cannot keep resources (like images) as separate files, they will need to be inlined (difficult, and can cause encoding problems).

An alternative to HTML is MHTML, but this is pretty much the same setup (where the file is formatted like an email). This means you still have the same problems, but will also need a program to parse the source HTML, include/encode the required resources, and update the HTML to point to those resources.

Another alternative is DOCX (MS Office), or OpenDocument, but these have their own problems:

- Typically seen as something that is editable, and often opened in a word processing program.
- Use their own specifications, which are fairly difficult to read/understand for developers who are typically used to HTML.

Then there is EPUP:

- These documents are often seen as publications (e.g. books), that you download and store in your e-reader.
- Works on the assumption that you want multi-paged content.
- Needs a [Table of Contents](http://www.idpf.org/epub/30/spec/epub30-contentdocs.html#sec-xhtml-nav-def-types-toc), and ideally a [cover-image](http://www.idpf.org/epub/30/spec/epub30-publications.html#sec-item-property-values).
- Requires a special program to package up the document (e.g. the [IDPF Java](https://github.com/IDPF/epub3-samples) program).
- Contains a fair amount of meta data (e.g. a list of all the XHTML documents in the OPF `<manifest>` and `<spine>`).
- It can allow [remote content](http://www.idpf.org/epub/30/spec/epub30-publications.html#sec-resource-locations), but this is disabled by default.

And finally PWP, which is related to EPUB:

- Documents are referenced from a [central URL](https://www.w3.org/TR/pwp/#identification), and uses the Open Web Platform to distribute the content.
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

The file will open in the users preferred browser.

Or it could be viewed within email clients, etc.

The file can be sent to someone via email, downloaded from a web page, stored on a USB drive... just like any other file.

## The browsers

Most of the technology is already in place for this to work.

The browsers will need to get to the `index.html` from the ZIP (which may be password protected), and any additional resources (also from the ZIP).

FireFox does have some of this in place, which looks roughly like:

	jar:file:///.../my-file.hdoc!/index.html

The browser will also need to enforce a Content Security Policy, one that blocks outbound connections, something like:

	default-src 'none';
	style-src 'self';
	font-src 'self';
	script-src 'self';
	img-src 'self';
	referrer no-referrer;
	frame-ancestors 'none'

I know, I'm not going to get away with script-src without unsafe-inline, but I can still hope :-)

The browser will have to block resources that are not in the ZIP file, some of which can be done with the sub-origin proposals.

Finally the browser will have to block access to local storage (inc cookies), so each time the document is opened, it's like being opened for the first time.

That's not to say that the browser (not the JavaScript in the document) couldn't do things like remember annotations for the user.

It might also have to block JavaScript from accessing the current date/time, so we don't have content that changes after a certain point in time (keep in mind those legal documents).

## Security

The most important thing about this file format is security.

We need IT departments to feel very comfortable allowing them onto their network.

And for virus/spam filters not to rely on huristics to determine if the file contains malicious code (like they do looking for macros in MS Word documents).

## Miscellaneous

You will notice this is a similar approach that Microsoft Word did with the docx format, Java with JAR, PHP with PHAR, etc.

Maybe support for multiple languages... or just use `lang="en"` and some CSS to show/hide?

---

# Bug reports

Chrome
Firefox
Safari
Edge
