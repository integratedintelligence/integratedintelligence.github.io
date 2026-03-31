# Integrated Intelligence Institute — Quarto Website Review

**Date:** March 31, 2026
**Scope:** Source files (`_quarto.yml`, `*.qmd`) and live rendering at `integratedintelligenceinstitute.org`

---

## 1. Critical Rendering Bugs

### 1.1 Raw HTML displayed as text on the homepage (`index.qmd`)

The most visible defect: the hero-section buttons and the "What we do" card interiors render as raw HTML source code rather than formatted elements. On the live site, visitors see literal strings like `<a class="btn-iii" href="apply.qmd">Apply for Fellowships</a>` and `<h3>Fellowships</h3>`.

**Root cause.** Pandoc (which Quarto uses internally) distinguishes *block-level* HTML elements (`<div>`, `<p>`, `<h1>`–`<h6>`, `<table>`, etc.) from *inline* elements (`<a>`, `<span>`, `<em>`, etc.). A line that begins with an inline tag like `<a>` is not recognized as a raw HTML block; Pandoc escapes it and renders the markup as visible text. Similarly, `<h3>` and `<p>` tags nested inside `<div>` blocks can fail to parse when Pandoc's block-vs-inline heuristics are disrupted by the surrounding container structure.

**Recommended fix.** Replace the raw HTML cards and buttons with Quarto-native constructs. For the buttons, wrap them in a `<div>` or `<p>` block so Pandoc sees a block-level parent. For the three-column cards, use a Quarto grid layout (`::: {.grid}` / `::: {.g-col-4}`) or Quarto's built-in card shortcodes instead of raw `<div class="iii-card">`. Example:

```markdown
::: {.grid}
::: {.g-col-4}
### Fellowships
Funded research placements and OPT/CPT-aligned opportunities...
:::
::: {.g-col-4}
### Training
Short courses and bootcamps in ML, data engineering...
:::
::: {.g-col-4}
### Research & Policy
Collaborative studies, open data resources...
:::
:::
```

For the CTA buttons, use Quarto's link-button class or wrap them properly:

```html
<div class="mt-4">
<p><a class="btn-iii" href="/apply/">Apply for Fellowships</a>
<a class="btn btn-outline-light ms-2" href="/programs/">Explore Programs</a></p>
</div>
```

### 1.2 Footer year shortcode not rendering

Every page footer displays the literal string `© {{< year >}} Integrated Intelligence Institute` instead of the current year. The `{{< year >}}` syntax is not a built-in Quarto shortcode. Either install a shortcode extension that provides it, define a custom Lua filter, or simply hardcode the year:

```yaml
page-footer:
  left: "© 2026 Integrated Intelligence Institute"
```

Alternatively, create a custom shortcode at `_extensions/year/year.lua`:

```lua
return {
  ['year'] = function(args, kwargs)
    return pandoc.Str(os.date("%Y"))
  end
}
```

---

## 2. Broken and Misconfigured Links

### 2.1 News callout points to Notion (`index.qmd`, line 56)

The "Featured updates" callout links to `https://www.notion.so/news/` — an apparent copy-paste artifact from Notion. It should read:

```markdown
See our [news](/news/) for announcements, deadlines, and events.
```

### 2.2 Partners page uses raw `<a>` with `.qmd` href (`partners.qmd`)

The inline HTML `<a href="contact.qmd">Contact us</a>` is rendered with the external-link icon (because `link-external-icon: true` in the config applies to anything Quarto can't resolve as an internal link). Replace with a Quarto markdown link:

```markdown
Interested in partnering? [Contact us](/contact/) to co-design a program or host fellows.
```

### 2.3 `site-url` uses `www` subdomain; live site does not

`_quarto.yml` declares `site-url: https://www.integratedintelligenceinstitute.org`, but the site resolves at the bare domain without `www`. This mismatch affects canonical URLs, OpenGraph `og:url` tags, and sitemap entries. Update to:

```yaml
site-url: https://integratedintelligenceinstitute.org
```

(Or configure a DNS/redirect so `www` resolves to the bare domain, and keep the YAML as-is.)

---

## 3. Contact Page Issues (`contact.qmd`)

- **Email and GitHub URL are plain text, not hyperlinks.** Use markdown links:
  ```markdown
  - Email: [integratedintelligence1@gmail.com](mailto:integratedintelligence1@gmail.com)
  - GitHub: [integratedintelligence](https://github.com/integratedintelligence)
  ```
- **"Address:" is empty.** Either populate it or remove the line. A trailing colon with no content looks like an oversight to visitors.
- **Gmail address for an institutional site.** Consider a domain-based email (e.g., `info@integratedintelligenceinstitute.org`) for professionalism and credibility.

---

## 4. Stale or Ambiguous Dates (`apply.qmd`)

- "Winter 2026 Internship (Jan–Mar): applications open Oct 25" — this deadline passed roughly five months ago (October 2025). If the cohort has been filled, replace with status text ("Cohort filled — next cycle opens ..."). If not, clarify with a full date including year.
- "Summer 2026 Internship (Jun–Aug): applications open April 1" — this is tomorrow. Confirm the portal is ready to go live, or update the date.
- "Submit via our portal (coming soon)" — if the portal is still not live as of the April 1 deadline, applicants have no submission mechanism. This needs resolution urgently.

---

## 5. Content Thinness

Several pages consist of only a few lines and give the impression of an early-stage draft rather than a functioning institutional website:

| Page | Approximate word count | Concern |
|------|----------------------|---------|
| About | ~50 words | No team bios, no institutional history, no governance structure |
| Partners | ~25 words | No named partners, logos, or testimonials |
| Programs | ~60 words | No learning outcomes, instructor names, or syllabi |
| Research | ~50 words | No publications, project descriptions, or named investigators |
| Contact | ~15 words | Incomplete (missing address), no contact form |

**Recommendations:**

- **About:** Add a mission statement with more detail, founding context, advisory board or leadership team, and institutional affiliations.
- **Partners:** Name at least a few partner organizations (with permission), add logos, and describe the nature of each collaboration.
- **Programs:** Expand each program with learning objectives, duration details, prerequisites, and past cohort highlights.
- **Research:** List at least working-paper titles, link to the Zenodo/GitHub repositories mentioned, and name principal investigators.
- **Contact:** Add a contact form (Quarto supports embedding Google Forms, Formspree, or similar).

---

## 6. Structural and Configuration Suggestions

### 6.1 Missing `description` metadata on individual pages

Only the site-level `description` is set in `_quarto.yml`. Adding `description:` to each page's YAML front matter improves SEO (populates `<meta name="description">`) and produces better OpenGraph/Twitter card previews when pages are shared individually.

### 6.2 No favicon or logo visible

The config references `images/favicon.png` and `images/logo.svg`, but neither renders in the browser tab or navbar. Verify that these files exist in the repository and are deployed correctly.

### 6.3 TOC on the homepage is unnecessary

With `toc: true` set globally, the homepage displays an "On this page" sidebar with just two entries ("What we do" and "Featured updates"). For a landing page with `page-layout: full`, suppress the TOC:

```yaml
---
title: ""
page-layout: full
toc: false
---
```

### 6.4 News listing has only one post

The listing page works, but a single announcement from October 2025 ("We are live!") is five months old. Regular content updates — even brief ones announcing application windows or new partnerships — would signal that the institute is active.

### 6.5 SCSS theme file not included

`_quarto.yml` references `styles/theme.scss`, which presumably defines the `.hero`, `.grid-3`, `.iii-card`, and `.btn-iii` classes. If this file is incomplete or missing from the deployment, those classes have no effect and the homepage layout degrades (as observed). Ensure the SCSS file is committed and contains proper definitions for all custom classes.

---

## 7. Summary of Recommended Actions (by priority)

1. **Fix raw HTML rendering on homepage** — convert to Quarto-native grid/div syntax
2. **Fix the footer year shortcode** — hardcode or implement a Lua shortcode
3. **Correct the broken Notion link** in the index callout
4. **Update stale application dates** and confirm the portal is ready
5. **Fix Contact page** — add hyperlinks, remove or populate the empty address field
6. **Fix the Partners page link** — use markdown syntax instead of raw `<a>`
7. **Reconcile `site-url`** with the actual domain (drop `www`)
8. **Suppress TOC on the homepage**
9. **Expand thin content** across About, Partners, Programs, Research, and Contact
10. **Add page-level `description` metadata** for SEO
11. **Verify favicon and logo deployment**
12. **Publish fresh News posts** to show institutional activity
