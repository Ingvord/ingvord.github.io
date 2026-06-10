# Content conventions

## File locations

All pages live in `src/pages/`. Each route is a directory with `index.astro`.

## Headings

Sentence case. `#` reserved for the page H1. Sections start at `##`.

## Metrics and numbers

Wrap in `<span class="metric">` — renders in IBM Plex Mono at `--p99` amber:

```html
<span class="metric">33%</span>
<span class="metric">$200K</span>
```

## Proof links

Use the `.proof-link` class everywhere. Format: `[label]` in square brackets, mono face, cyan border.

```html
<a href="..." class="proof-link">[video]</a>
<a href="..." class="proof-link">[DOI: 10.5281/zenodo.xxx]</a>
<a href="..." class="proof-link">[slides]</a>
<a href="..." class="proof-link">[source]</a>
```

Keep the vocabulary consistent: `video`, `slides`, `DOI`, `source`. No invented labels.

## Images

SVG preferred for diagrams (draw in site palette). Raster as WebP in `public/`. Always include meaningful `alt`.

## Design tokens (in `src/styles/global.css`)

| Token | Value | Use |
|---|---|---|
| `--ink` | `#0E1116` | Background |
| `--panel` | `#161B22` | Cards, code blocks |
| `--text` | `#D6DEE7` | Body text |
| `--muted` | `#8B98A5` | Captions, metadata |
| `--trace` | `#5CC8FF` | Links, accents |
| `--p99` | `#FFB454` | Metrics, numbers only |

One accent per screen. `--p99` is for numbers, `--trace` is for everything else.

## Restraint rules

- One animation on the site (hero trace). Nothing else moves.
- No card grids, no skill bars, no avatar sections.
- No decorative code blocks. Code appears only when code genuinely appears.
