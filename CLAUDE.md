# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

A **single-page marketing website** for *Lara's Jewelry*, a family-owned Los
Angeles fine-jewelry store (est. 1982). The entire site is one self-contained
file: **`index.html`** (~3,300 lines). There is no backend, no e-commerce, and
no checkout — the goal is to showcase the showroom and drive phone calls and
in-store visits.

## Tech stack

- **Pure static HTML** — no framework, no build step.
- **CSS** lives in a single `<style>` block in `<head>`. No external
  stylesheet, no preprocessor.
- **JavaScript** is vanilla, in a single `<script>` block just before
  `</body>`. No libraries, no bundler.
- **Fonts**: Google Fonts — *Cormorant Garamond* (serif headings) and *Inter*
  (sans body), loaded via `<link>`.
- **Images**: all hotlinked from external hosts (`larasjewelry.com`
  WordPress uploads and Yelp's CDN). Nothing is stored in the repo.

There is **no `package.json`, no dependencies, no tests, and no lint config.**
The repo is just `index.html` plus git metadata.

## Run / preview

Open the file directly, or serve it locally:

```bash
# simplest: open in a browser
open index.html        # macOS / xdg-open on Linux

# or serve over http (needed for some browser features)
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to build, compile, or install. "Deploy" = publish
`index.html` as a static file.

## Page structure (top to bottom)

All sections are anchored by `id` and reachable from the nav. In order:

| Section | `id` | Notes |
|---|---|---|
| Header / nav | `#header`, `#nav` | Sticky; collapses to a hamburger on mobile |
| Hero | — | Headline + call/explore CTAs |
| Trust bar | — | Animated stat counters (40+, 4.9, GIA, 100%) |
| Shop | `#shop` | Tabbed product catalog (see below) |
| Our Story | `#story` | Brand history |
| Gallery | `#gallery` | Links out to Yelp photos |
| Custom Design | `#custom` | |
| Browse by Cut | — | Diamond-cut tiles that deep-link into the shop |
| Reviews | `#reviews` | |
| Sell Gold | `#sell` | A `.services` section |
| Services | `#services` | |
| CTA banner | — | |
| Visit | `#visit` | Address, hours, contact, embedded Google Map |
| Footer | — | Link columns + socials |
| Mobile CTA bar | — | Fixed "Call Now / Visit" bar shown on small screens |

## The Shop: tabs and sub-tabs

This is the most structured part of the page and the part most likely to need
edits. It has **3 top-level categories**, each split into sub-tabs, with **135
product cards** total.

- **Engagement Rings** (`data-category="engagement"`) — sub-tabs by cut:
  `brilliant`, `princess`, `oval`, `emerald`, `cushion`, `radiant`, `pear`
- **Wedding Bands** (`data-category="bands"`) — `womens`, `mens`
- **Fine Jewelry** (`data-category="fine"`) — `earrings`, `necklaces`

### How the markup is wired

- Top tab buttons: `<button class="shop-tab" data-category="...">`
- One content panel per category: `<div class="shop-content" data-category="...">`
- Inside each panel, sub-tab buttons:
  `<button class="sub-tab" data-parent="<category>" data-subcat="<subcat>">`
- One panel per sub-tab:
  `<div class="subcategory-content" data-parent="<category>" data-subcat="<subcat>">`
- The currently-visible tab/panel carries the `active` class. Exactly one
  `shop-tab` and one `sub-tab` per group should be `active` at a time.

### Product card pattern

Every product is the same block. Copy this exactly when adding one:

```html
<div class="product-card fade-in">
    <div class="product-image">
        <img src="https://larasjewelry.com/.../image-768x768.jpg"
             alt="Descriptive product name" loading="lazy">
    </div>
    <div class="product-info">
        <div class="product-name">GIA 1.00 ct. Round Brilliant Diamond Engagement Ring</div>
        <div class="product-prices">
            <span class="product-price-original">$13,930</span>
            <span class="product-price-sale">$8,900</span>
        </div>
    </div>
</div>
```

- Cards live inside a `.product-grid` within the matching
  `.subcategory-content` panel.
- The `fade-in` class is required for the scroll-reveal animation. Stagger
  groups of four with `fade-in-delay-1`, `fade-in-delay-2`, `fade-in-delay-3`
  (then repeat from no-delay) so cards cascade in.
- `alt` text doubles as the product description — keep it accurate.
- Both a `product-price-original` (struck-through) and `product-price-sale`
  are expected.

### Deep links into the shop

"Browse by Cut" tiles and footer links jump straight to a category/sub-tab:

```html
<a href="#shop" data-shop-tab="engagement" data-filter-cut="cushion">…</a>
```

`data-shop-tab` selects the category; optional `data-filter-cut` clicks the
matching sub-tab. This is handled in the JS at the bottom of the file — if you
add a category or sub-tab, the existing handlers pick it up automatically as
long as the `data-*` attributes match.

## JavaScript behaviors

All in the single `<script>` block. Each is small and independent:

- **Fade-in reveals** — `IntersectionObserver` adds `.visible` to `.fade-in`
  elements as they scroll into view.
- **Sticky header** — toggles `.scrolled` past 60px.
- **Mobile menu** — `#mobileToggle` toggles `.open`/`.active` and locks body
  scroll.
- **Smooth scroll** — intercepts `a[href^="#"]`, offsetting for the sticky
  header height.
- **Trust-bar counters** — animates numbers containing `+`.
- **Shop tab / sub-tab switching** — see the Shop section above.

## CSS conventions

- **Design tokens** are CSS custom properties in `:root` (top of the
  `<style>` block). Use them instead of hardcoding values:
  - Gold palette: `--gold`, `--gold-light`, `--gold-dark`, `--gold-shimmer`
  - Neutrals: `--black`, `--charcoal`, `--dark-gray`, `--medium-gray`,
    `--light-gray`, `--cream`, `--warm-white`, `--white`
  - Type: `--serif` (Cormorant Garamond), `--sans` (Inter)
  - Layout: `--max-width` (1200px), `--transition`
- **Class naming** is kebab-case, loosely BEM-flavored
  (`.product-card`, `.product-price-sale`, `.subcategory-content`).
- **Animation helpers**: `.fade-in` (+ `.fade-in-delay-1/2/3`); keyframes
  `shimmer`, `float`, `pulseGlow`.
- **Layout** uses `.container` (centered, `--max-width`, 24px gutters).
- **Responsive** breakpoints, in `@media (max-width: …)` blocks near the end of
  the style block: **1024px**, **768px**, **480px**.

## Formatting / editing conventions

- **4-space indentation**, matching the existing file.
- Keep everything **in `index.html`** — do not split CSS/JS into separate files
  or introduce a build tool unless explicitly asked. The single-file design is
  intentional (easy to host anywhere).
- `<img>` tags use `loading="lazy"`; external links use
  `target="_blank" rel="noopener"`.
- Because content (prices, products, reviews) is hand-maintained HTML, make
  bulk product edits carefully and consistently — the card markup must stay
  identical across all 135 entries for styling and animation to work.

## Business facts (keep consistent across the page)

These appear in multiple places (hero, nav CTA, visit section, footer, mobile
bar). If one changes, update **all** occurrences:

- **Phone**: (213) 623-0919 — used as `tel:+12136230919`
- **Email**: info@larasjewelry.com
- **Address**: 550 South Hill Street, Suite 570 (5th floor), International
  Jewelry Plaza, Los Angeles, CA 90013
- **Hours**: Mon–Fri 10–4, Sat 10–3, Sun closed
- **Established**: 1982 · founders George & Nawal Garabet
- **Rating shown**: 4.9 stars on Yelp

> Note: a recent commit corrected the phone number (623-7717 → 623-0919).
> Phone, address, and hours are the kind of details that drift — verify before
> changing them, and change every instance.

## Git workflow

- Active development branch for this work: `claude/claude-md-docs-datYX`.
- Commit with clear, descriptive messages; push with
  `git push -u origin <branch>`; open a **draft PR** after pushing.
- Do not force-push or push to `main` without explicit permission.
