# HtmlToXml
Utility to convert HTML to XML, including recovery from unclosed tags and other adjustments.

## OverView

I wrote HtmlToXml because I was unable to find an existing .NET component for converting HTML to XML. Several facilities will consume HTML and provide a DOM or XML version of the input, but my need was somewhat different: I had to convert user-supplied HTML to XHTML for inclusion in an EPUB book. (Unlike browsers, EPUB readers will reject an XHTML document that is not a valid XML document.)

## Goals

The primary goal is to avoid errors when reading the text after it has been included in an XML document.

The secondary goal is to produce XHTML that mimics what a web browser would render after interpreting the original HTML.

## Features

- Converts HTML named entities to XML decimal character references, i.e., "`&mdash;`" is converted to "`&#8212;`".

- Converts tags for HTML "[void elements](https://html.spec.whatwg.org/multipage/syntax.html#void-elements "WHATWG specification")", such as IMG, BR, and META, to self-closed XML tags.

- Converts unquoted attribute values to quoted values.

- Closes unclosed tags, with special handling for elements that HTML specifies are closed by subsequent elements, such as P, LI, and TD.

- Closes unclosed elements that would otherwise produce an improperly nested result.

- Ignores end tags with no matching start tag. Such tags may occur due to the processing associated with other features.

- Removes HTML comments.

- Removes the DOCTYPE statement.

- Wraps text inside SCRIPT and STYLE elements inside CDATA tags which are themselves wrapped in /* to */. That makes the text valid in both XHTML and HTML contexts.

- Treats elements with unrecognized names as block elements, except for   names that include a namespace, which are treated as inline elements. This affects unclosed paragraph processing: `<o>` will close an open paragraph, but `<o:p>` will not.

## Usage

```csharp
using HtmlToXml;

var html = new HtmlConverter();
var xmlText = html.Convert("<p>This is a test.");
```

`xmlText` would equal "`<p>This is a test.</p>`" on output.

HtmlConverter works with HTML substrings. If the input to HtmlConverter.Convert is not wrapped in the HTML element ("`<html> ... </html>`"), the result may or may not have an outer element that encloses the remaining elements. Before you load the result into XmlDocument or XDocument, you may have to wrap it with a document element.

## Examples

A `'>'` character that is not used to end a tag
is encoded.

```html
I: <p>File > Save As</p>
O: <p>File &gt; Save As</p>
```

Any open elements are closed when the end of the
input is reached.

```html
I: <p>Unclosed paragraph.
O: <p>Unclosed paragraph.</p>
```

Open paragraph elements are closed when the next
block element is encountered.

```html
I: <p>Paragraph one.<p>Paragraph two.
O: <p>Paragraph one.</p><p>Paragraph two.</p>
```

BR elements and other void elements become
self-closing XML elements.

```html
I: <p>A simple<br>paragraph.</p>
O: <p>A simple<br/>paragraph.</p>
```

HTML that is already valid XML is left unchanged, though
minor variations may be introduced.

```html
I: <p>A simple<br />paragraph.</p>
O: <p>A simple<br/>paragraph.</p>
```

Inline elements inside a block element are closed
when the block ends, but not because they are inline
elements. They are closed to honor XML's nesting rules.

```html
I: <p>Paragraph <b>one.</p>
O: <p>Paragraph <b>one.</b></p>
```

Open LI elements are closed when its next sibling
is opened.

```html
I: <ul><li>Item 1<li>Item 2</ul>
O: <ul><li>Item 1</li><li>Item 2</li></ul>
```

An open LI element is not closed by any block element,
only by a sibling LI. The DIV is a child of the prior
LI.

```html
I: <ul><li>Item 1<li>Item 2<div>Here
O: <ul><li>Item 1</li><li>Item 2<div>Here</div></li></ul>
```

Nested lists will be handled properly as long as the
start and end tags for the lists (UL or OL) are present.

```html
I: <ul><li>Item 1<ul><li>Item 1.1</li></ul><li>Item 2</li></ul>
O: <ul><li>Item 1<ul><li>Item 1.1</li></ul></li><li>Item 2</li></ul>

I: <ul><li>Item 1<ul><li>Item 1.1</ul><li>Item 2</li></ul>
O: <ul><li>Item 1<ul><li>Item 1.1</li></ul></li><li>Item 2</li></ul>
```

Named HTML entities are converted to decimal entities.

```html
I: <p>Extra.&nbsp; Spacing.</p>
O: <p>Extra.&#160; Spacing.</p>
```

HTML comments are removed.

```html
I: <p>You <!--ain't-->got it.</p>
O: <p>You got it.</p>
```

Unquoted parameter values are quoted.

```html
I: <img alt="" height=1 width=1 src="image.jpg">
O: <img alt="" height="1" width="1" src="image.jpg"/>
```

Unencoded ampersands in parameter values are encoded.

```html
I: <a href="foo?doo&ret=no">Text</a>
O: <a href="foo?doo&amp;ret=no">Text</a>
```

The `<o>` tag closes the P element because an
unknown element is treated as a block element.

```html
I: <p><o>test</o></p>
O: <p></p><o>test</o>
```

The `<o:p>` tag does not close the P element
because an unknown element with a namespace is
treated as an inline element.

```html
I: <p><o:p>test</o:p></p>
O: <p><o:p>test</o:p></p>
```

Unclosed table cells (TD and TH) are closed.

```html
I: <table><th>H1<th>H2<tbody><td>C1<td>C2
O: <table><th>H1</th><th>H2</th><tbody><td>C1</td><td>C2</td></tbody></table>
```

Nested tables are handled properly as long as the nested tables are closed. 

```html
I: <table><th>H1<th>H2<tbody><td>C1
   <table><tr><td>T2.C1</table><td>C2</table>
O: <table><th>H1</th><th>H2</th><tbody><td>C1
   <table><tr><td>T2.C1</td></tr></table></td>
   <td>C2</td></tbody></table>
```

Text inside SCRIPT and STYLE elements is wrapped in `<![CDATA[` and `]]>` to hide text that does not follow XML rules.

```html
I: <style>p > em { color:red; }</style>
O: <style><![CDATA[p > em { color:red; }]]></style>
```

## Credits

Includes a modified version of [TextParser](http://www.blackbeltcoder.com/Articles/strings/a-text-parsing-helper-class), a class originally written by Jonathan Wood.
