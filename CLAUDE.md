# Working in this repo

Plain static HTML site. No framework, no bundler, no runtime JS. Edit the `.html` files directly — that is the intended workflow.

## Two things to know before editing

**1. Styling is inline.** Every element carries a `style=""` attribute. There is no Tailwind, no CSS modules, no utility classes. Colors come from CSS custom properties defined in `_ds/aragocor-minerals-design-system-*/colors_and_type.css` and re-declared in a `:root` block in each page's `<head>`. Use the tokens, not raw hex:

`--ink #0E3540` · `--tide #526A70` · `--shore #7C97A0` · `--sand #D7C59C` · `--bone #F7F4ED` · `--paper #FFFFFF` · `--root`/`--leaf #49613C`
Type: `--font-serif` (Source Serif 4, headings) · `--font-sans` (Geist, body) · `--font-mono` (JetBrains Mono, labels and data)
Also available: `--shadow-1/2/3`, `--ease-tide`, `--border`, `--border-strong`, `--fg-1/2/3`, `--ink-04` through `--ink-80`.

**2. The nav and footer are duplicated across all eight pages.** They were shared components in the original design and got inlined during the static build. If you change one, change all eight, or regenerate (below). Nav markup lives inside `<div class="dc-sitenav">`, footer inside `<div class="dc-sitefooter">`. The active-page underline is a bare `<div>` with `height: 2px; background: var(--bone)` — present only on the current page's nav item.

## Responsive

`site.css` is the only hand-written stylesheet. It overrides inline styles at narrow viewports using `!important` (author `!important` beats inline declarations). Elements are tagged during the build:

- `r-shell` — the 1440px page container; reduced padding on mobile
- `r-grid-2` / `r-grid-3` — card and content grids; collapse to one column
- `r-data` — data-table grids; font shrinks but columns stay
- `r-h1` / `r-h2` — headings; font-size steps down
- `r-hero` — fixed-height hero blocks; become auto-height
- `r-tablewrap` — wrapper around `<table>`; scrolls horizontally

If you add markup by hand, add the matching class or it will not respond.

## Regenerating from the design source

`_source/` holds the original Claude Design `.dc.html` templates and `build.mjs`, the transpiler that produced these pages. It resolves `<dc-import>`, expands `<sc-for>` and `<sc-if>`, evaluates the `renderVals()` data blocks, applies the responsive tagging, wires the contact form, patches the data-sheet nav bar, and emits static HTML.

```bash
node _source/build.mjs _source .
```

The build is idempotent — every transformation lives in `build.mjs`, so regenerating reproduces the shipped pages exactly. It **overwrites** the eight page files, so hand edits to them are lost. Pick one lane: either edit the HTML directly and stop regenerating, or edit `_source/` and always regenerate.

For a site this size, editing the HTML directly is usually the better path. The `_source/` route earns its keep only if the content lists (industry cards, spec tables, rate tables) need frequent bulk changes.

### The build verifies itself

`build.mjs` runs a `verify()` pass on every page and exits non-zero on failure. It checks:

- `<style>` and `</style>` balance **in the head**
- `<div>` / `</div>` balance in the body
- no unresolved `{{ }}` bindings or `sc-for` / `sc-if` / `dc-import` tags
- at least 500 characters of visible text

That last two matter more than they look. An earlier version of this build deduplicated head fragments line by line, which silently dropped a repeated `</style>`. The unclosed `<style>` then swallowed the entire body as CSS text and every page rendered as an empty cream rectangle — valid HTML, correct background color, zero content. Grepping for page text still found it, because the text was in the file; it just wasn't reachable. If you touch the head assembly, re-run the build and confirm it exits 0, and sanity-check with a parser rather than a grep:

```bash
python3 -c "
from html.parser import HTMLParser
class P(HTMLParser):
    skip=0; text=[]
    def handle_starttag(s,t,a):
        if t in ('style','script'): s.skip+=1
    def handle_endtag(s,t):
        if t in ('style','script'): s.skip=max(0,s.skip-1)
    def handle_data(s,d):
        if not s.skip and d.strip(): s.text.append(d.strip())
p=P(); p.feed(open('index.html').read()); print(len(' '.join(p.text)), 'chars renderable')
"
```

## Don't

- Add a build step or framework unless the site outgrows eight static pages.
- Move pages into subdirectories without fixing the relative `assets/` and `_ds/` paths.
- Hardcode hex values that already exist as tokens.
