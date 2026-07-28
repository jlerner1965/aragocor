# Aragocor Minerals — Site

Eight-page marketing site for AragoCor Minerals LLC. **Plain static HTML.** No framework, no build step, no runtime JavaScript. Every page is fully rendered in the file — open one in a browser and it works.

## View it

```bash
npm run dev      # http://localhost:8080
# or
python3 -m http.server 8080
```

`index.html` also opens directly from the filesystem, since nothing is fetched at runtime.

## Deploy

Vercel:
```bash
npx vercel --prod
```

Netlify:
```bash
npx netlify-cli deploy --prod --dir .
```

Or drag the folder onto either dashboard. `vercel.json` sets clean URLs, so `/products` serves `products.html`.

## Pages

| URL | File |
|---|---|
| `/` | `index.html` |
| `/products` | `products.html` |
| `/industries` | `industries.html` |
| `/sustainability` | `sustainability.html` |
| `/resources` | `resources.html` |
| `/technical-data-sheet` | `technical-data-sheet.html` |
| `/about` | `about.html` |
| `/contact` | `contact.html` |

## Before you go live

1. **Wire the contact form.** `contact.html` posts to `https://formspree.io/f/YOUR_FORM_ID`. Swap in a real Formspree ID, or replace the `action` with your own endpoint. On Netlify you can instead delete the `action` and add `data-netlify="true"` plus a hidden `form-name` field.
2. **Set the real domain** in `sitemap.xml` and `robots.txt` — both currently assume `aragocorminerals.com`.
3. **Check the claims.** The pages assert a 1998 founding, three continents shipped, third-party verification on 100% of lots, OMRI listing, and specific CaCO₃/heavy-metal figures. These came out of the design draft. Verify each one against your actual documentation before this is public — spec claims on a B2B minerals site are the kind of thing buyers hold you to.
4. **Fonts** load from Google Fonts via `_ds/.../colors_and_type.css`. Self-host them if you want zero third-party requests.

## Structure

```
index.html …  contact.html   fully-rendered pages
site.css                     responsive overrides
assets/                      images, logos, PDFs
_ds/                         design system CSS (tokens, type scale)
_source/                     original Claude Design templates + build.mjs
```

`_source/` is kept for regeneration only — nothing in it ships. See `CLAUDE.md`.
