## Edulife IT School — multi-page site

A production-ready, multi-page static site built from a single-page design, structured so a
developer can drop a dynamic backend underneath it without touching the markup.

- **Design language:** cream canvas · navy ink · teal primary · amber / coral accents
- **Typography:** Fraunces (display serif) + Manrope (UI sans)
- **Pages:** Home · About Us · Admission · Teachers · Notice · Events · Gallery · Blog · Contact · 404
- **Detail pages:** one per item, generated from the JSON — `blog/<slug>.html`,
  `notice/<slug>.html`, `event/<slug>.html`, plus the runtime `article.html?type=&slug=` route
- **No framework, no build toolchain required** — one small Node script assembles the pages.
- **Requirements:** Node 18+ (developed on Node 20). There are **no dependencies** — no
  `npm install`, no bundler. `node build.js` is the whole toolchain.

---

## Quick start

```bash
cd edulife-it-school

npm run build      # = node build.js && cp -r .build/. .   (optional — pages are committed)
npm run preview    # = npx serve .   → http://localhost:8000
```

Prefer no npm? The underlying commands are just:

```bash
node build.js && cp -r .build/. .   # rebuild the pages
python3 -m http.server 8000         # preview
```

> **Want your own repo?** See **`PUSH-TO-GITHUB.md`** for the exact `git init` → `git push`
> commands and the GitHub Pages setup.

---

## 1. Folder structure

```
edulife-it-school/
├── index.html              ← generated output (do not edit by hand)
├── about.html              ← generated output
├── admission.html          ← generated output
├── teachers.html           ← generated output
├── notice.html             ← generated output
├── events.html             ← generated output
├── gallery.html            ← generated output
├── blog.html               ← generated output
├── contact.html            ← generated output
├── 404.html                ← generated output
├── article.html            ← generated: the runtime detail route
├── sitemap.xml
├── robots.txt
│
├── blog/<slug>.html        ← GENERATED, one file per blog post
├── notice/<slug>.html      ← GENERATED, one file per notice
├── event/<slug>.html       ← GENERATED, one file per event
│
├── data/                   ← CONTENT LAYER — swap these for your API
│   ├── site.json           brand, nav, contact, campuses, hero, pillars,
│   │                       stats, features, admission steps, classes we offer,
│   │                       closing CTA, initiatives (the Edulife family), footer
│   ├── teachers.json       { leadership: [...], staff: [...] }
│   ├── notices.json        { items: [...] }
│   ├── events.json         { items: [...] }
│   ├── gallery.json        { items: [...] }
│   └── blog.json           { items: [...] }
│
├── layouts/
│   └── base.html           ← the shell: <head>, header slot, footer slot,
│                             script tags, {{CONTENT}} slot
│
├── components/             ← reusable partials
│   ├── header.html         sticky header + generated nav
│   ├── drawer.html         slide-in mobile nav panel + dimmed/blurred scrim
│   ├── footer.html         footer + generated link columns
│   └── lightbox.html       gallery image viewer
│
├── pages/                  ← PAGE BODIES (source of truth for markup)
│   ├── index.html · about.html · admission.html · teachers.html
│   ├── notice.html · events.html · gallery.html · blog.html
│   ├── contact.html · 404.html
│   ├── detail.html         THE detail template — one file, all three families
│   └── article.html        the runtime route shell (?type=&slug=)
│
├── assets/
│   ├── css/styles.css      all styling (design tokens at the top)
│   ├── js/renderers.js     list AND detail markup, SHARED by Node + browser
│   ├── js/data-loader.js   the static-JSON ↔ REST seam
│   ├── js/main.js          page behaviours (nav, reveal, filters, lightbox, share)
│   ├── js/article.js       the runtime detail route (?type=&slug=)
│   ├── js/chat.js          live-chat loader — EDIT THE 2 IDs HERE (§9)
│   └── images/             all imagery
│
├── build.js                ← static site generator (node build.js)
├── check.js                ← quality-gate checks (`node check.js`)
├── LICENSE                 ← MIT
├── PUSH-TO-GITHUB.md       ← push-to-your-own-repo + GitHub Pages walkthrough
├── .gitignore
└── .github/workflows/      ← deploy-pages.yml (GitHub Pages CI)
```

### Why `pages/` and the root `.html` files both exist

`pages/*.html` are the **sources**. `build.js` wraps each one in `layouts/base.html`,
injects the generated header/footer and renders every `{{BLOCK_*}}` placeholder from
`data/*.json`, writing the finished page to `.build/`, which is then copied to the
project root. The root files are the **artifacts you deploy** — never edit them directly,
your changes will be overwritten on the next build.

---

## 2. Routing model

Plain multi-page routing. One URL = one file, no client-side router, so the site is
crawlable, works on any static host, and opens correctly from `file://`.

| Route | File | Nav key |
|---|---|---|
| `/` | `index.html` | `home` |
| `/about.html` | `about.html` | `about` |
| `/admission.html` | `admission.html` | `admission` |
| `/teachers.html` | `teachers.html` | `teachers` |
| `/notice.html` | `notice.html` | `notice` |
| `/events.html` | `events.html` | `events` |
| `/gallery.html` | `gallery.html` | `gallery` |
| `/blog.html` | `blog.html` | `blog` |
| `/contact.html` | `contact.html` | `contact` |
| `/blog/:slug` | `blog/<slug>.html` | `blog` |
| `/notice/:slug` | `notice/<slug>.html` | `notice` |
| `/event/:slug` | `event/<slug>.html` | `events` |
| `/article.html?type=&slug=` | `article.html` | (set at runtime) |
| (any 404) | `404.html` | `404` |

The **nav key** lives on `<body data-page="…">` and is the only thing that determines
which nav item gets `class="active"`. Links in `data/site.json > nav` are plain relative
`.html` paths, so nothing needs rewriting when you deploy under a sub-path.

> **Moving to clean URLs** (`/about` instead of `/about.html`) means changing the `href`
> values in `data/site.json > nav`, `footer.columns` and any in-page link, then
> configuring rewrites on your host. All the hrefs come from one place, so it is a
> search-and-replace, not a refactor.

### Detail routes (single blog post · notice · event)

Every blog post, notice and event has **its own page**. The pattern is identical
across the three families and each family is a folder:

| Route | File on disk | Source | Slug field |
|---|---|---|---|
| `/blog/:slug` | `blog/<slug>.html` | `data/blog.json` → `items[]` | `slug` |
| `/notice/:slug` | `notice/<slug>.html` | `data/notices.json` → `items[]` | `slug` |
| `/event/:slug` | `event/<slug>.html` | `data/events.json` → `items[]` | `slug` |

All three use **one template**, `pages/detail.html`. It carries a single token,
`{{DETAIL_CONTENT}}`, which `build.js` fills by calling
`renderers.js → renderDetail(type, item, ctx)`. No article markup is written by
hand and there is no per-type template to drift out of sync. The item's siblings
in the same array supply the previous/next pager and the related grid, so nothing
is duplicated in the JSON.

> Detail pages are one level deep, so their asset and link paths need a `../`.
> `build.js` applies this automatically: the generated page is run through
> `rebase(html, 1)`, which prefixes every site-root-relative `href`/`src` (and
> `og:image`) and leaves absolute URLs, `#` anchors and already-relative paths
> alone. Never hand-write paths in a detail template.

#### Two entry points, one template

| Entry point | What it is | Why it exists |
|---|---|---|
| `blog/<slug>.html` etc. | a real, static file per item | crawlable, shareable, works with no JavaScript |
| `article.html?type=&slug=` | one URL that can serve any item | the shape a dynamic backend needs |

`assets/js/article.js` powers the second: it reads `?type=` and `?slug=`, loads
that family through `data-loader.js`, finds the matching item and renders it with
the *same* detail renderers. A missing or unknown slug produces an honest
not-found panel with real links onward, never a blank page.

#### Converting to clean, backend-owned routes

Point your host's rewrite rules at the runtime entry:

| Rewrite | To |
|---|---|
| `/blog/:slug` | `/article.html?type=blog&slug=:slug` |
| `/notice/:slug` | `/article.html?type=notice&slug=:slug` |
| `/event/:slug` | `/article.html?type=event&slug=:slug` |

Then tell the renderers to emit extensionless links:

```js
EDU.render.setLinkBase('/');   // blog/<slug>.html  →  /blog/<slug>
```

That one call updates `detailHref()` for the listing cards, the pager and the
related grid at once. Everything else stays as it is.

#### How to add a new post, notice or event

1. **Add one object** to the right array in `data/`:

   ```json
   {
     "slug": "my-new-post",
     "title": "My new post",
     "category": "Parenting",
     "date": "2026-10-20",
     "dateLabel": "20 Oct 2026",
     "readTime": "4 min read",
     "author": "Nusrat Jahan",
     "authorRole": "English & Phonetics",
     "authorImage": "assets/images/t-sadia.jpg",
     "tags": ["Reading"],
     "image": "assets/images/u-reading.jpg",
     "alt": "Description of the image",
     "excerpt": "One-sentence summary, used on cards and in search results.",
     "body": [
       { "type": "lead", "text": "An opening paragraph in a larger serif face." },
       { "type": "p", "text": "Body copy." },
       { "type": "h2", "text": "A section heading" },
       { "type": "list", "items": ["First point", "Second point"] }
     ]
   }
   ```

2. **Rebuild**: `node build.js && cp -r .build/* .`  
   A page appears at `blog/my-new-post.html`, its listing card links to it, and
   the item automatically joins its siblings' pager and related grid. No route
   registration, no template edit.

3. **Notices and events** take the same shape with their own extra fields:
   notices add `priority` (`Normal` · `Important` · `Urgent`), `department`,
   `audience[]`, `day`, `monthYear`; events add `day`, `month`, `monthYear`,
   `venue`, `time`, `endTime`, `audience[]`, `registration` and an optional
   `agenda[]` of `{ "time", "item" }`.

#### The article body contract

`body` is an array of blocks. This is the complete vocabulary — add a `case` to
`detailBody()` in `renderers.js` to extend it:

| `type` | Fields | Renders as |
|---|---|---|
| `lead` | `text` | opening paragraph, larger serif, amber rule |
| `p` | `text` | body paragraph |
| `h2` | `text` | section heading |
| `h3` | `text` | sub-heading |
| `list` | `items[]` | bulleted list |
| `quote` | `text`, `cite` | dark pull-quote with attribution |
| `callout` | `title`, `text` | amber highlighted aside |
| `image` | `image`, `alt`, `caption` | full-width figure |

Every block and every card passes through `esc()`, so text typed by a person can
never inject markup.

---

## 3. Build, preview & deploy

### 3.1 Build

```bash
cd edulife-it-school

node build.js          # writes .build/*.html, the blog/ notice/ event/ detail
                       # folders, + sitemap.xml + robots.txt
cp -r .build/. .       # publish the generated pages over the root copies
```

The published root `.html` files **are committed to the repo** — they are what every static
host serves, and they mean the site works with no build step at all. Re-run the two commands
after editing anything in `pages/`, `components/`, `layouts/`, `data/` or `assets/`.

> `cp -r .build/. .` (or `.build/*`) — the `-r` matters. Once the build emits the
> `blog/<slug>.html`, `notice/<slug>.html` and `event/<slug>.html` detail folders, a plain
> `cp .build/*.html .` silently drops all of them while still reporting success.

### 3.2 Preview locally

Any static file server works; open the printed URL (usually <http://localhost:8000>).

```bash
npx serve .            # or:  python3 -m http.server 8000
```

Serve it over `http://` rather than double-clicking the files: the `data/*.json` fetches and
the `article.html?type=&slug=` runtime route need a real origin, not `file://`.

### 3.3 Deploy — GitHub Pages (including a repo sub-path)

**This is the "base path" case.** The site is built with **relative URLs only** —
`assets/css/styles.css`, `about.html`, `blog/<slug>.html` — and there is **not a single
root-absolute `/assets/...` reference anywhere** (verified: 0 occurrences). That is
deliberate, and it means the project runs correctly both at a domain root and at a
**sub-path** such as `https://<user>.github.io/<repo>/`.

**So for a normal GitHub Pages project site you change nothing.** No `<base>` tag, no build
config, no find-and-replace. The only places a base path can leak in are:

| Place | What to do |
| --- | --- |
| `sitemap.xml` `<loc>` values | Relative by default. For production make them **absolute**, including the sub-path: `https://<user>.github.io/<repo>/about.html`. |
| `robots.txt` | Add a `Sitemap: https://…` line with the absolute URL if you want one. |
| `layouts/base.html` `<link rel="canonical">` | Relative (a build token). Set the absolute production origin if canonical URLs matter to you. |
| A `<base href="…">` tag | **Do not add one.** It would break the relative detail routes under `blog/` and `notice/`. |

Enable it in one of two ways:

**Option A — GitHub Actions (recommended; rebuilds on every push).** A ready-made workflow
ships at `.github/workflows/deploy-pages.yml`. Set **Settings → Pages → Build and
deployment → Source: GitHub Actions**, then push to `main`.

**Option B — deploy straight from the branch.** Push to `main`, then choose
**Settings → Pages → Source: Deploy from a branch → Branch: `main`**, folder **`/` (root)**.
This works because the built pages are committed, so GitHub needs to run nothing.

On a **custom domain** (or a user/organisation site at `https://<user>.github.io/`) the site
is served from the root and every relative path still resolves — again nothing to change.

### 3.4 Deploy — Vercel / Netlify

Both serve the repo root as a static site out of the box: no framework preset and no build
required (or set the build command to `node build.js && cp -r .build/. .` if you would
rather they rebuild on deploy).

- **Vercel:** import the repo → Framework Preset **Other** → Build Command empty (or the
  above) → Output Directory `.`
- **Netlify:** New site from Git → Build command empty (or the above) → Publish directory `.`
  → set the 404 page to `/404.html`

Any static host behaves the same way (S3 + CloudFront, nginx, Cloudflare Pages…). Point the
404 handler at `404.html`.

---

## 4. How to add a page

1. **Create the body** — copy an existing file in `pages/`, e.g. `pages/academics.html`.
   Keep the `.page-hero` block at the top for a consistent entry point.

2. **Register it** in `build.js`, in the `PAGES` array:

   ```js
   { src: 'academics.html', out: 'academics.html', key: 'academics',
     title: 'Academics — Edulife IT School',
     description: 'Curriculum, subjects and how learning is organised.' }
   ```

   `key` must match a `nav[].key` in `data/site.json` if you want the nav item highlighted.

3. **Add it to the navigation** — `data/site.json > nav` (this feeds both the desktop nav
   and the mobile drawer), and to `footer.columns` if it belongs there.

4. **Add a new list?** Add the JSON in `data/`, add a renderer in
   `assets/js/renderers.js`, register the view in `assets/js/data-loader.js > VIEWS`,
   and add a `BLOCK_*` entry in `build.js`:

   ```js
   BLOCK_SUBJECTS: () => R.renderList(data.subjects, R.subjectCard)
   ```

   Then reference `{{BLOCK_SUBJECTS}}` from the page body.

5. Rebuild: `node build.js && cp -r .build/* .`

> **Adding a blog post, notice or event is different** from adding a page — you
> never touch `pages/` at all. You add one object with a `slug` and a `body` to
> the matching JSON file, and `build.js` generates the page. See
> *Detail routes → How to add a new post, notice or event* above.

---

## 5. Connecting a backend / CMS

The site is designed so this is a **configuration change, not a rewrite**.

### Step 1 — the data contract

Every source maps to one endpoint. JSON files and REST responses have identical shapes.

| Source | Static file | List endpoint | Detail endpoint |
|---|---|---|---|
| Site / global | `data/site.json` | `GET /site` | — |
| Teachers | `data/teachers.json` | `GET /teachers` | — |
| Notices | `data/notices.json` | `GET /notices` | `GET /notices/:slug` |
| Events | `data/events.json` | `GET /events` | `GET /events/:slug` |
| Gallery | `data/gallery.json` | `GET /gallery` | — |
| Blog | `data/blog.json` | `GET /blog` | `GET /blog/:slug` |

### Step 2 — switch the loader over

Edit the `config` block at the top of `assets/js/data-loader.js`:

```js
var config = {
  useStaticJson: false,                          // ← was true
  dataDir: 'data/',
  apiBase: 'https://api.your-school.example/v1', // ← your backend
  endpoints: {
    site:     { json: 'site.json',     api: '/site' },
    teachers: { json: 'teachers.json', api: '/teachers' },
    notices:  { json: 'notices.json',  api: '/notices' },
    events:   { json: 'events.json',   api: '/events' },
    gallery:  { json: 'gallery.json',  api: '/gallery' },
    blog:     { json: 'blog.json',     api: '/blog' }
  }
};
```

### Step 3 — mark up the page as live

Each list container already declares where it gets its content from:

```html
<div class="teachers"
     data-source="teachers"    <!-- endpoint key        -->
     data-pick="staff"         <!-- path into payload   -->
     data-view="teachers"      <!-- renderer name       -->
     data-limit="8">           <!-- optional cap        -->
  …pre-rendered cards…         <!-- shipped by the build -->
</div>
```

Set the page live with `<body data-live="true">` (a token in `layouts/base.html`). The
loader then fetches each source once, renders through the renderer named by `data-view`,
and replaces only the container's children.

**Graceful degradation is built in:** because `build.js` already wrote real cards into
every list, the page is complete and indexable before JavaScript runs. If the API is slow
or down, the loader logs the error and leaves the build-time markup in place — a failed
request never blanks a page.

### Step 4 — form submissions

Both forms (`admission.html`, `contact.html`) are **client-side demos** carrying
`data-validate`; they validate in the browser and store nothing. To post them for real,
add an action and a fetch call in the `initForms()` handler in `assets/js/main.js`:

```js
fetch('/api/admission', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(Object.fromEntries(new FormData(form)))
}).then(/* show .form-success */);
```

The success banner (`.form-success`) and its styles already exist.

> ⚠️ Never point these forms at a third-party relay or a mailbox you do not control.

---

## 6. Theming

All design decisions live in the `:root` block at the top of `assets/css/styles.css`:

```css
--ink:#101B2D;        /* navy — headings, footer, dark panels */
--cream:#FBF7F0;      /* page canvas */
--teal:#0E7C6B;       /* primary actions, links, accents */
--amber:#F5A524;      /* highlight, accents */
--coral:#E4572E;      /* secondary accent, form errors */
--radius / --radius-lg / --shadow-* / --maxw / --ease
```

Change those and the whole site re-themes. Fonts are loaded once in `layouts/base.html`.

### Closing admission section (`.cta-sec` / `.cta-cols`) — homepage

Since **v13** the homepage's closing band is no longer a contact + buttons CTA panel. It is a
**two-column admission section** that lets a guardian read the process and submit an inquiry
without leaving the homepage. Copy lives in `data/site.json → ctaSteps`.

| Column | Contents |
| --- | --- |
| `.cta-left` | eyebrow, headline, intro line, then the **five-step journey** (`.steps` → `.step`), rendered by `stepCard()` |
| `.cta-right` | the **admission inquiry form** (`.form-card`) — student / parent names, phone, email, class dropdown, preferred campus, message, submit |

**The five steps are shared, not duplicated.** `ctaSteps` deliberately does **not** store the step
list: the steps come from the shared `admissionSteps` array that `/admission` renders too, so the
two pages can never describe a different process. The form's class dropdown is generated from
`classes.items` (item `BLOCK_CLASS_OPTIONS` / `classOption()`), so adding a class updates the grid
and every dropdown at once.

**The equal-height rule — do not "simplify" it.** The brief requires the two columns to end on
the same line, so `.cta-cols` is a **CSS grid with `align-items:stretch`** and each column carries
**`height:100%`**. The grid sizes both columns to the tallest one and the shorter column's card
fills the remainder. No fixed height, no JS measurement, correct at every viewport. Two things
keep it honest:

- `align-items:stretch` is written out explicitly even though it is the grid default — it is the
  mechanism, so it must survive future edits.
- Below **860px** the grid collapses to one column (`.cta-cols{grid-template-columns:1fr}`) and
  `height:100%` becomes a no-op. **Equal height stops applying on purpose** — once the columns are
  stacked there is nothing to align, and forcing a shared height would only create empty space.

**Bottom-anchor the submit block (v11–v12 lesson, still the rule).** The right column is a flex
column whose submit block carries `margin-top:auto`, so the button sits at the bottom edge of the
card. Do **not** instead put `margin-top:auto` on the buttons alone: that pins only them down and
leaves the copy above stuck at the top, opening a ~330px dead gap at 1280/1440 — the exact defect
v11–v12 fixed. Because the form is much taller than the step list, the grid stretches the left
panel down to the form's bottom, and it is the **panel** (not the column) that carries the surface,
so the extra area reads as intentional padding rather than a void.

**One closing section, not two (v12 fix — still true).** The homepage once carried the closing band
twice: an "Admission teaser" (`.pad-sm` + `.admission-grid` + `.campus-card`) *and* the `.cta-sec`
panel. The teaser was removed from `pages/index.html`. `.admission-grid` / `.campus-card` are
**still used** by `pages/contact.html` and `pages/admission.html`, so their CSS must stay.

Editing the copy: change `ctaSteps.*` in `data/site.json` and rebuild; the five steps themselves
are edited in `admissionSteps` (shared with the Admission page).

### The Edulife family (`.ini-sec` / `.initiatives`) — homepage

The section directly below the closing CTA lists the other initiatives: **Edulife IT, Edulife
Agency, Edulife Kids, Edulife Publications, FoodLife, Edulife Tech**.

Content lives in `data/site.json → initiatives`:

```json
{
  "eyebrow": "The Edulife family",
  "title": "More than a school.",
  "text": "…",
  "items": [
    { "id": "edulife-it", "monogram": "IT", "name": "Edulife IT",
      "tagline": "…", "href": "index.html", "tone": "teal" }
  ]
}
```

`initiativeCard()` in `assets/js/renderers.js` renders each entry as a single `<a>` tile (so the
whole card is the tap target): a **two-letter monogram** tinted by `tone` (`teal` / `amber` /
`coral`), the name, the one-line `tagline`, and an arrow. **Adding an initiative needs no artwork
and no markup** — add an object to the array, run `node build.js`, done.

- Grid: **3 columns** desktop → **2 columns** at ≤1024px → **1 column** at ≤620px.
- The container carries the usual `data-source` / `data-pick` / `data-view` attributes, so it can
  be hydrated from the API later exactly like every other list on the site (see §6).
- Visually distinct from the footer (light cards on cream, its own section heading) while reusing
  the same card radius, border and shadow tokens.

### Classes we offer (`.classes-sec` / `.classes`) — Admission page

The Admission page lists the seven classes the school admits into — **Nursery, Junior One, Class
One, Class Two, Class Three, Class Four, Class Five** — as a grid of level cards, placed after the
admission steps and before the FAQ so it reads as part of the admission journey.

Content lives in `data/site.json → classes`:

```json
{
  "eyebrow": "Classes we offer",
  "title": "Seven classes, one clear path.",
  "text": "…",
  "badge": "Nursery to Class Five",
  "items": [
    { "id": "nursery", "mark": "N", "name": "Nursery", "age": "Age 3–4",
      "focus": "…", "tone": "teal" }
  ]
}
```

`classCard()` in `assets/js/renderers.js` renders each entry as a card carrying a large Fraunces
**level mark** (`mark`) tinted by `tone` (`teal` / `amber` / `coral`), an **age chip**, the class
name and a one-line `focus`. **Adding or renaming a class needs no markup** — edit the array, run
`node build.js`, done.

- Grid: **4 columns** desktop → **2 columns** at ≤1024px → **1 column** at ≤620px. The breakpoints
  are explicit rather than `auto-fit`, so the seven cards always break into tidy rows instead of
  leaving a single orphan on a wide screen.
- The container carries the usual `data-source` / `data-pick` / `data-view` attributes, so it can
  be hydrated from the API later exactly like every other list on the site (see §6).
- The admission inquiry form's **Class** dropdown lists the same seven levels — keep the two in
  sync when a class is added.

---

## 7. Accessibility & behaviour notes

- Respects `prefers-reduced-motion` — all reveal/float/marquee animation is disabled.
- Print stylesheet hides nav, drawer, scrim, progress bar, decorations and the chat widget.
- The mobile drawer is a proper `role="dialog"` + `aria-modal` panel: focus moves into it on
  open, is trapped while open (Tab/Shift+Tab cycle inside), returns to the hamburger on close,
  and it closes on Escape, scrim tap, the close button or a nav link. The page behind it is
  scroll-locked without losing the reading position (`position:fixed` + saved offset).
- Gallery tiles are keyboard reachable (`tabindex`, Enter opens the lightbox, Escape closes).
- Filter chips carry `aria-pressed`; empty filter results show an explicit message
  rather than a blank grid.
- Icon-only controls (`burger`, `drawerClose`, `toTop`, lightbox close) all have `aria-label`.

## 8. Known follow-ups for a real launch

- `sitemap.xml` uses relative URLs — make them absolute (including any repo sub-path) on deploy.
  See §3.3.
- The school's real logo is now in place: `assets/images/logo-seal.png` (circular seal,
  used as the header/drawer/footer brand mark and the favicon) and
  `assets/images/logo-lockup.png` (horizontal lockup, available for print/OG use).
  For a production icon set, generate 16/32/180/512px PNGs plus an `.ico` from the seal.
- Press logos and permit scans are placeholders — swap in licensed assets. They now appear
  **only on the About page**; the homepage "Official recognition" section was removed, so the
  `.trust` / `.permit` / `.press` styles are shared with About and must not be deleted.
- Contact details (`+880 1800 000000`, `hello@edulife.edu.bd`) are placeholders.
- The runtime detail route (`article.html?type=&slug=`) sets its own title and
  description client-side. Once you serve real dynamic routes, have the backend
  render `<title>` / `<meta name="description">` for the item so crawlers and
  social scrapers see the right values without running JavaScript.
- `blog.html` etc. still render every item at build time. When you connect an
  API, mark those pages `data-live="true"` so the lists hydrate from the
  endpoint (the static markup stays as the no-JS fallback).

---

## 9. Live chat (tawk.to)

Every page of the site loads `assets/js/chat.js`, which embeds the **tawk.to** live-chat
widget. tawk.to is a free live-chat / helpdesk service: a small chat bubble appears
bottom-right on every page, visitors type a message, and staff reply from a dashboard or
the phone app. The free plan covers unlimited agents, unlimited concurrent chats and one
website property — enough for a school office.

**✅ The widget is now CONFIGURED and live site-wide.** The real IDs are already in
place:

```js
var TAWK_PROPERTY_ID = '6ace6f47d0f2b934caf86b75';   /* Property ID */
var TAWK_WIDGET_ID   = '1k4r65co4';                   /* Widget ID   */
```

which loads the standard embed at `https://embed.tawk.to/6ace6f47d0f2b934caf86b75/1k4r65co4`.
The chat bubble appears bottom-right on every page.

### Change the IDs later (or move to a new tawk.to property)

1. **Find your IDs:** `Administration → Channels → Chat Widget → Direct Chat Link`.
   The link looks like this:

   ```
   https://tawk.to/chat/6a1b2c3d4e5f6a7b8c9d0e1f/1i2j3k4l
                       └──── PROPERTY ID ─────┘ └ WIDGET ID ┘
   ```
2. **Paste those two values** into `assets/js/chat.js` — the top of the file is a
   clearly marked config block, and these are the only two lines you touch:

   ```js
   var TAWK_PROPERTY_ID = '6ace6f47d0f2b934caf86b75';   /* ← Property ID */
   var TAWK_WIDGET_ID   = '1k4r65co4';                   /* ← Widget ID   */
   ```

   For the link above that would be the `6a1b…0e1f` part and the `1i2j…k4l` part.
3. **Rebuild** so the new script is stamped into every page: `node build.js && cp -r .build/* .`
   (Both are needed — chat.js is part of the cache-busting hash, so editing it changes the
   `?v=` stamp on every page. Use `-r` on the copy: it must carry the `blog/`, `notice/`
   and `event/` subdirectories too.)

> **Safety net:** if the constants are ever left as placeholders (starting with `TAWK_`),
> `chat.js` loads nothing at all — no widget, no console errors, no failing requests — so
> a typo can never break the site, it just silently disables chat.

### How the school answers chats

- **Dashboard:** log in at <https://dashboard.tawk.to/> — the *Inbox* shows live chats and
  the messaging history.
- **Mobile app:** install the free **tawk.to** app (iOS / Android) and sign in with the same
  account; replies can be sent from the phone.
- **While nobody is online**, tawk.to shows an offline form and emails the transcript, so no
  enquiry is lost.

### Layout notes

- `chat.js` pins the bubble at `z-index: 1090`, deliberately **below** the mobile drawer
  (`z-index: 1100`), so an open menu is never covered by the chat bubble.
- On phones the back-to-top button is moved to the **bottom-left** (see the `max-width:560px`
  block in `styles.css`) so it can never overlap the bottom-right chat bubble.
- The widget is hidden in print stylesheets.
- To change the bubble's corner or offset, edit the `Tawk_API.customStyle.visibility` block
  in `chat.js` (`br` = bottom-right, `bl` = bottom-left, etc.).
