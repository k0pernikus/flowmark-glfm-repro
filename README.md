# flowmark GLFM round-trip reproduction

Three GitLab Flavored Markdown constructs do not survive a `flowmark` pass. Each file
in `glfm/` is a minimal input for one of them.

Pinned: `flowmark 0.7.3`, via `uvx` in the `format` task.

## Run

```bash
mise install
mise run format
git diff
```

`git diff` shows what `flowmark` wrote.

## Expected vs actual

`flowmark` is expected to round-trip all three files byte-identically.

### `glfm/table-of-contents.md`

`[[_TOC_]]` becomes `[[*TOC*]]`.

GitLab renders `[[_TOC_]]` as a table of contents:

```html
<ul class="section-nav"><li><a href="#title">Title</a>...</li></ul>
```

`[[…]]` is GitLab's wikilink syntax, so `[[*TOC*]]` renders as a link to a page named
`*TOC*`, and the table of contents is gone:

```html
<p><a href="*TOC*" data-wikilink="true">*TOC*</a></p>
```

### `glfm/description-list.md`

The line break between term and description is removed, so no line begins with `:`.

Before, GitLab renders a description list:

```html
<dl><dt>Coffee</dt><dd>A hot beverage.</dd></dl>
```

After, it is a paragraph:

```html
<p>Coffee : A hot beverage.</p>
```

### `glfm/multiline-blockquote.md`

The `>>>` fence becomes `> > > ` (three blockquote markers, plus trailing whitespace).

Before, GitLab renders one blockquote holding both paragraphs:

```html
<blockquote><p>A quoted message</p><p>spanning multiple blocks.</p></blockquote>
```

After, the content is no longer quoted, and six empty nested blockquotes are emitted
around it:

```html
<blockquote><blockquote><blockquote></blockquote></blockquote></blockquote>
<p>A quoted message</p>
<p>spanning multiple blocks.</p>
<blockquote><blockquote><blockquote></blockquote></blockquote></blockquote>
```

Rendered HTML above is from GitLab's `/markdown` REST API with `gfm: true`.
