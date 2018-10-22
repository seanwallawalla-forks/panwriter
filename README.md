# Panwriter

Panwriter is a distraction-free markdown editor with two unique features:

1. Tight integration with pandoc for import/export to/from plenty of file formats (including HTML, docx, LaTeX and EPUB).
2. Preview pane that can show pages – including page breaks etc. Layout adjustments are done in-file using CSS, and are immediately reflected in the preview.


## Usage

### Export preview to PDF

Select `File -> 'Print / PDF'` and `PDF -> 'Save as PDF'` in the print dialog (exact naming might depend on your OS).

This will export exactly what’s shown in the preview, and not use pandoc at all.

By adding a `style` field to your YAML metadata, you can change the styling of the preview and immediately see the changes. (You can later save your CSS as a theme, see [Document types](#document-types-themes) below.)

    ---
    title: my document
    style: |
      @page {
        size: A4;
        margin-top: 2cm;
      }
      body {
        font-size: 20px; /* set base */
      }
      h1 {
        font-size: 1.5em; /* scale relative to base */
      }
    ---

(To include that CSS when exporting to HTML/EPUB with pandoc, you would have to use a custom pandoc template with the snippet `<style>$style$</style>`. We’ll try to make this more straight-forward in the future.)

### Export via pandoc

Select `File -> Export` and choose a format.

If you have a YAML metadata block, like in the following example, Panwriter will look at the extension of the filename you chose in the dialog, and look up the corresponding key in the `output` YAML metadata, for example when exporting the following markdown to `test.html`:

    ---
    title: my document
    pdf-format: latex  # optional
    output:
      html:
        katex: true  # for math output
        include-in-header:
          - foo.css
          - bar.js
      latex:
        pdf-engine: xelatex
        toc: true
        toc-depth: 3
        template: letter.tex
        metadata:
          fontsize: 12pt
      epub:
        to: epub2  # default would be epub3
    ---

    # my document

this command will be executed:

    pandoc --toc --include-in-header foo.css --include-in-header bar.js --output test.html --to html --standalone

There are two exceptions to the rule that the key in the `output` YAML is the file extension:

1. When exporting to a `.tex` file, the key should be named `latex`.
2. When exporting to a `.pdf` file, the key for Panwriter to look up in the `output` YAML can be specified with the `pdf-format` key (see example above). Default is also `latex`, but you can also use `context`, `html`, `ms`, `beamer`, `revealjs`, etc.  In fact, you could set it to anything, if you had a corresponding key in the `output` YAML with a `to:` field. See also [Creating a PDF with pandoc](http://pandoc.org/MANUAL.html#creating-a-pdf).

### Default CSS and YAML

Panwriter will look for `~/.panwriter/default.css` to load CSS for the preview. If that file is not found, it will use sensible defaults.

If you put some YAML in `~/.panwriter/default.yaml`, Panwriter will merge this with the YAML in your input file (to determine the commandline arguments to call pandoc with) and add the `--metadata-file` option. The YAML should be in the same format as above.

### Document types / themes

You can e.g. put `type: letter` in the YAML of your input document. In that case, Panwriter will look for `~/.panwriter/letter.yaml` and `~/.panwriter/letter.css` instead of `default.yaml` and `default.css`.

### Markdown syntax

We use `markdown-it` for the preview pane, which is fully [CommonMark](https://commonmark.org/)-compliant. We also added a bunch of plugins, to make the preview behave as much as pandoc as possible (including attributes, [`fenced_divs`](http://pandoc.org/MANUAL.html#extension-fenced_divs), `definition_lists`, `footnotes`, `implicit-figures`, `subscript`, `superscript`, `yaml_metadata_block` and `tex_math_dollars`). We explicitly don't support `raw_html` or `raw_tex`, since everything should be doable with the `fenced_divs`, `bracketed_spans` and `raw_attribute` extensions.

However, there might still be minor differences between the preview and `File -> 'Print / PDF'` on one hand, and `File -> Export` on the other.

Things we should emulate in the preview, but for which there are [no markdown-it plugins yet](https://github.com/atom-community/markdown-preview-plus/wiki/markdown-it-vs.-pandoc):

- `bracketed_spans` [markdown-it-span/issue](https://github.com/pnewell/markdown-it-span/issues/2)
- `grid_tables`: grid tables are the only ones in pandoc, that can have e.g. a list in a cell
- [`raw_attribute`](http://pandoc.org/MANUAL.html#extension-raw_attribute): we should probably just strip them from preview
- backslash at end of paragraph, e.g. `![](foo.png) \` An ugly workaround that already works is `![](foo.png) &nbsp;`

Pandoc markdown supports a few more things which will not render correctly in the preview, but which are not so commonly used. However, you can still use them in your markdown file, and export via pandoc will work.


## About CSS for print

Unfortunately, still no browser fully implements the CSS specs for paged media (paged media are print or PDF). Therefore, Panwriter's preview is powered by [pagedjs](https://gitlab.pagedmedia.org/tools/pagedjs) – a collection of paged media polyfills by [pagedmedia.org](https://pagedmedia.org). Some background on using CSS for print:

- [Motivating article on A List Apart](https://alistapart.com/article/building-books-with-css3)
- [Print-CSS resources, tools](https://print-css.rocks)
- [W3C Paged Media Module](https://www.w3.org/TR/css-page-3/)
- [W3C Generated Content for Paged Media](https://www.w3.org/TR/css-gcpm-3/)


## Develop

    # Install JavaScript dependencies
    npm install

    # Install PureScript dependencies
    bower install

    # Compile PureScript
    pulp --watch build

    # rebuild packaged pagedjs (usually not necessary)
    npm install -g browserify
    cd previewFrame
    clear; cd pagedjs && npm run-script compile && cd .. && browserify previewFrame.js -o previewFrame.bundle.js

    # Run the app
    npm start


### TODOs before first release

- polish
    - show filename (but hide chrome on typing)
    - “show/hide preview” button
    - toggle paged on/off in preview
    - sync scroll, or at least scroll on click
- build package
- decide whether to do CSS-in-YAML with later YAML-metadata-slider/[color-picker]-like-GUI

[color-picker]: https://easylogic.github.io/codemirror-colorpicker/

### Possible TODOs

- GUI popup on file import: at least allow to set `-f`, `-t`, `--track-changes` and `--extract-media`
- Windows, Linux versions
- [Variable substitution in body](https://github.com/jgm/pandoc/issues/1950#issuecomment-427671251)
- Editor:
    - adjust font-size on editor window resize
    - spell check
    - find/replace


## Powered by

Panwriter is powered by (amongst other open source libraries):

- [pandoc](http://pandoc.org/MANUAL.html) (import/export)
- [Electron](https://electronjs.org/docs/tutorial/application-architecture) (app framework)
- [CodeMirror](https://codemirror.net) (editor)
- For the preview pane:
    - [pagedjs](https://gitlab.pagedmedia.org/tools/pagedjs)
    - [markdown-it](https://github.com/markdown-it/markdown-it#markdown-it)
    - [KaTeX](https://katex.org)
