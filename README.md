# letter.css – Quarto HTML Stylesheet for US Letter Documents

A professional CSS stylesheet for rendering Quarto Markdown documents as US Letter-sized (8.5" × 11") print-ready HTML with footnotes anchored to their originating pages, self-contained embedding, and zero external CDN dependencies.

## Features

- **US Letter Format** — 8.5" × 11" with 1.0" margins on all sides
- **Footnotes on Originating Pages** — Endnotes automatically relocate to the page where they're referenced, not collected at document end
- **Print-to-PDF Optimized** — Tested with Chrome headless and standard browser print dialogs
- **Self-Contained** — All fonts and resources embedded; no external CDN loads. Safe for AWS hosting.
- **LaTeX-Style Text Sizing** — Inline span classes for `HUGE`, `large`, `small`, `tiny`, etc.
- **Academic Formatting** — 11pt Arial body text, 0.5" abstract margins (1.5" total inset), 8pt footnotes and code
- **Table of Contents** — Fixed-position sidebar TOC on screen; hidden during print
- **Clean Screen Simulation** — Stacked Letter sheets on gray background for realistic preview

## Basic YAML Setup

```yaml
---
title: "Your Document Title"
author: "Author Name"
date: 2026-09-25
format:
  html:
    css: "path/to/letter.css"
    theme: none
    embed-resources: true
    toc: true
    toc-location: right
    include-after-body: "path/to/page-footnotes.html"
editor: source
---
```

### Key YAML Details

- **`css: "path/to/letter.css"`** — Points to this stylesheet
- **`theme: none`** — Disables Quarto's default theme so letter.css takes full control
- **`embed-resources: true`** — Self-contained HTML; all CSS, images, fonts inline
- **`toc-location: right`** — Positions table of contents as a fixed sidebar on screen
- **`include-after-body: "path/to/page-footnotes.html"`** — Required for footnote relocation script

## Important Quirks & Workarounds

### 1. Title Block Relocated onto Page 1

Quarto auto-generates a `#title-block-header` element from your YAML `title`/`author`/`date` fields. It renders as a sibling of the `.page` divs (not nested inside them), so **a script (`page-footnotes.html`) moves its children (`h1.title`, `p.author`, `p.date`) into the start of the first `.page` at render time, then removes the now-empty `#title-block-header` wrapper** — no need to retype metadata in the markdown body, and YAML stays the single source of truth.

```markdown
---
title: "Your Document Title"
author: "Author Name"
date: 2026-09-25
format:
  html:
    css: "path/to/letter.css"
    theme: none
    embed-resources: true
    toc: true
    toc-location: right
    include-after-body: "path/to/page-footnotes.html"
---

::: {.page}
## First Content Heading
Body text starts here — no need to retype the title/author/date.
:::
```

**Requires JavaScript** (same as footnote relocation, below). If JS doesn't run — e.g. with non-JS PDF engines like WeasyPrint — the title block will render unstyled, outside the page stack.

> **CSS gotcha:** Because the `#title-block-header` wrapper is *removed*, not just relocated, any styling for the title/author/date must target `.page > h1.title`, `.page > p.author`, and `.page > p.date` directly — selectors scoped under `#title-block-header` will silently never match once the script runs. Title renders bold, left-aligned, at normal body size (11pt, 1.2 line height); author/date render at 11pt with 1.2 line height and zero margin between them for tight spacing.

### 2. Page Structure

All content must be wrapped in `::: {.page}` blocks (Quarto divs). Each block renders as one Letter-sized sheet:

```markdown
::: {.page}
# Page 1 Content
Content here...
:::

::: {.page}
# Page 2 Content
Content here...
:::
```

### 3. Footnotes Require Script

Footnotes work through a companion script file (`page-footnotes.html`). Without it, footnotes collect at document end (off the page). The script must be included via `include-after-body` in YAML.

## Custom Classes

### Text Sizing

Use inline span syntax to apply LaTeX-style sizes:

```markdown
[HUGE text]{.HUGE}
[Regular text]{.large}
[small text]{.small}
[tiny text]{.tiny}
```

Available sizes: `.HUGE` (36pt), `.huge` (24pt), `.LARGE` (16pt), `.Large` (14pt), `.large` (12pt), `.small` (9pt), `.footnotesize` (8pt), `.tiny` (6pt).

### Other Utility Classes

- **`.abstract`** — Indented summary block with 1.5" left/right inset (0.5" additional margin beyond page padding)
- **`.center`** — Centered text and blocks with 1.5em top/bottom margin
- **`.compact`** — Dense 9pt text for data, metadata, notes
- **`.page-footnotes`** — Auto-generated container for page-specific footnotes (handled by script)

## Print & PDF Export

### Browser Print to PDF

```bash
# Example: Chrome headless
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --headless=new \
  --disable-gpu \
  --no-pdf-header-footer \
  --print-to-pdf="output.pdf" \
  "file://$(pwd)/document.html"
```

**Tips:**
- Use `--no-pdf-header-footer` to suppress browser chrome
- Disable custom headers/footers in browser print settings
- Set margins to 0 (the stylesheet handles page margins)

### Browser Print Dialog

1. Open HTML in Chrome/Firefox/Safari
2. Print → Save as PDF
3. Set margins to **None** or **0**
4. Disable headers/footers
5. Print

## Screen vs. Print Rendering

- **Screen (@media screen)** — Gray background, drop shadows, visual spacing between sheets, fixed TOC sidebar, page numbers
- **Print (@media print)** — Clean white pages, no shadows/borders, native print engine pagination, TOC hidden

## Dependencies

- **letter.css** — Main stylesheet (this file)
- **page-footnotes.html** — JavaScript for footnote relocation (required for footnote functionality)
- **Quarto** — Version 1.2+

## Browser & PDF Engine Support

- ✅ Chrome (headless and interactive)
- ✅ Firefox
- ✅ Safari
- ✅ Print to PDF (all modern browsers)
- ✅ WeasyPrint (command-line PDF generation)

## Limitations

- Footnotes appear at *end of page content*, not pixel-perfect anchored to physical page bottom (browser print engines don't reliably support CSS footnote floats)
- Page numbering uses CSS `counter(page)` in @page rule (native print engine), visible only in printed output, not on screen preview
- External SVG/image URLs will break self-containment; embed all graphics inline or use base64 data URIs

## Examples

### Minimal Document

```markdown
---
title: "Report"
format:
  html:
    css: "letter.css"
    embed-resources: true
    include-after-body: "page-footnotes.html"
---

::: {.page}

# Report Title
Author Name

::: {.abstract}
This is the abstract/summary.
:::

## Section 1

Body text here with a footnote[^1].

[^1]: Footnote text appears on this same page.

:::
```

### Multi-Page with Sizing

```markdown
::: {.page}

# Title

[Large introduction]{.LARGE}

Body text...

:::

::: {.page}

# Page 2

[Smaller section heading]{.Large}

::: {.center}
[Centered, tiny label]{.tiny}
:::

:::
```

## License

MIT License — see [LICENSE](LICENSE) for the full text.

## Contact

For issues, questions, or improvements, [contact/repo link].
