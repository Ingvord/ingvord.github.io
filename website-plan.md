# ingvord.dev — Site Plan

Personal site for Igor Khokhriakov, Staff Software Engineer.
Single job of the site: convert a recruiter or hiring manager who arrives from LinkedIn
into a message, by proving the headline claims with public artifacts within 60 seconds.

---

## 1. Hosting & Stack

| Decision | Choice | Why |
|---|---|---|
| Host | GitHub Pages | Free, reliable, and itself a credibility signal (repo is public work sample) |
| Generator | **Astro** (alternative: Hugo) | Static output, zero client JS by default, Markdown content, trivially deployed via GitHub Actions |
| Repo | `ingvord/ingvord.github.io` or `ingvord/site` + Pages workflow | Standard setup, custom domain supported |
| Domain | `ingvord.dev` (register), CNAME in repo | `.dev` forces HTTPS, reads as engineer-native, removes the `.ru` friction for EU employers |
| Old site | Keep `ingvord.ru` alive "for memories", add a banner + `<link rel="canonical">` pointing to the new domain; ideally 301 the homepage | Preserves old backlinks, redirects search traffic |
| Analytics | None, or Plausible/GoatCounter | No Yandex Metrica, no cookie banner needed |
| CI | GitHub Actions: build + deploy on push to `main` | The pipeline itself is on-brand ("no manual handoffs") |

Quality floor: Lighthouse ≥ 95 on all categories, responsive to 360px,
respects `prefers-reduced-motion`, visible keyboard focus, OG/meta tags per page.

---

## 2. Site Map

```
/                  Home (the argument — 90% of the value)
/talks/            Curated talks, each with one-line context
/writing/          Notes archive (migrated selectively from ingvord.ru)
/trading-platform/ Case study (written for systematic trading firms)
/publications/     One page: 3 highlighted papers + link to Google Scholar
404                One-liner in the site's voice + link home
```

No blog commitment, no "Uncategorized", no bitcoin page.
Old Java/C/ZeroMQ notes migrate only if individually still accurate; otherwise they stay
on ingvord.ru as the archive.

---

## 3. Home Page — Content (full draft)

### Hero

> **Igor Khokhriakov**
> Staff Software Engineer · Java / JVM · Distributed Systems · Kubernetes · Elasticsearch
>
> # Distributed systems don't fail loudly.
>
> They degrade quietly, accumulate debt silently, and block the next phase of work
> long before anyone names the blocker. I've spent 17 years finding those blockers
> and removing them.
>
> [Read the proof ↓]   [Message me — I respond to everything]

The headline doubles as the signature visual element (see §5): behind or beneath it,
a stylized APM trace waterfall — the sequential-vs-parallel Elasticsearch query
pattern that produced the P99 win. The hero literally shows "infrastructure
explaining itself."

### Section: The work takes three forms

Three blocks, each: claim → 2–3 sentences → **linked proof artifacts**.
This linking is the whole point of the site; LinkedIn can't do it.

**1 · Build the platform that makes the next thing possible**

- RCSB Protein Data Bank (5B requests/yr, 14.5M users): integrated Elastic APM,
  surfaced sequential Elasticsearch execution no application metric had caught,
  cut P99 latency 33% with zero caller-code changes.
  → proof: [SCaLE 20x talk — RCSB.org → Kubernetes migration (video)],
  [Observability of SCADA systems w/ Elastic APM (slides)]
- DESY: ISPyB refactor that let the beamline team bring one of the facility's most
  commercially significant beamlines live.
- OpenStack → Kubernetes migration, $200K direct savings.
  → proof: [WeAreDevelopers World Congress 2024 talk (video)]
- Now building: a fully automated trading-strategy platform — concept to live
  production with no manual handoffs. → [case study]

**2 · Generate the evidence that prevents the wrong thing from being built**

- Code-level assessment of a distributed storage engine: modeled 5×–10× growth,
  located where metadata coordination, request fan-out, and object-storage latency
  combine into production constraints — before any appeared.
- 72-scenario load study of SciCat's metadata API; showed 20+ European and US
  facilities where their scaling ceiling sits.
  → proof: [NOBUGS 2024 talk (video)], [Zenodo DOI: 10.5281/zenodo.15056189]

**3 · Extract the method and leave it behind**

- Tango REST specification → industry standard across 100+ facilities.
- Published, reusable benchmarking framework and load-testing methodology.
  → proof: [Zenodo records], [GitHub: waltz-controls]

### Section: Still shipping

> Production Java is where I'm sharpest — including low-level `Unsafe`-based work —
> and I intend to keep shipping it.
> Stack: Java/JVM · Node/TypeScript · Rust · C++ · ELK · Kubernetes.
> Python for tooling and analysis, not production.

### Section: Selected talks (4, curated — full list on /talks/)

1. How I Saved $200K with 0 Code in K8s — WeAreDevelopers World Congress 2024
2. Migrating RCSB.org to Kubernetes — SCaLE 20x, 2023
3. Performance Benchmarking of Node.js, Rust, Python and Go — NOBUGS 2024
4. Reactive Programming for Tango-Controls — 40th Tango Users Meeting, 2026

Each entry: title, venue, year, one line of "why this matters", links to video + slides.

### Footer / Contact

> Currently exploring Staff and Principal Engineer roles at infrastructure product
> companies and systematic trading firms. Fully remote, async-first.
> **If you have a system that's blocking what comes next — [email] · [LinkedIn] ·
> [GitHub]. I respond to everything.**

© year · Built with Astro, deployed by GitHub Actions — [source]

(The "source" link to the site repo is deliberate: the site is a work sample.)

---

## 4. /trading-platform/ — Case Study Outline (~600–900 words)

Audience: systematic trading firms. Tone: engineering write-up, not marketing.

1. **Problem** — taking a strategy from idea to live capital usually involves manual
   handoffs (research → backtest → review → deploy → monitor), each one a delay and
   an error source.
2. **Architecture** — one diagram (SVG, drawn in site style): strategy definition →
   validation/backtest → promotion gate → live execution → observability loop back
   to research. Name the stack honestly (JVM core, etc.).
3. **The interesting decisions** — 2–3 short subsections, e.g. how promotion gates
   are encoded, how the observability loop closes, what "no manual handoffs" means
   at the failure boundary.
4. **Status & numbers** — whatever is true and shareable (strategies live, latency,
   uptime). No inflated claims; absence of numbers is better than soft ones.
5. **What this demonstrates** — one paragraph tying back to the three-forms framing.

---

## 5. Design Direction

Subject grounding: the site belongs to the world of traces, dashboards, and latency
percentiles — observability made legible. The design borrows that vernacular
precisely and quietly, instead of a generic "developer portfolio" look.

### Signature element (the one memorable thing)

**The hero trace.** A horizontal APM-style span waterfall rendered as inline SVG:
a row of sequential spans that, on load (or on scroll, reduced-motion users see the
final state), reflows into parallel spans — the actual shape of the P99 fix. Caption
in the utility face: `p99 −33% · 0 caller-code changes`. This is the only animated
element on the site. Everything else stays still.

### Palette (dark, instrument-like — but not the generic black + acid green)

| Token | Hex | Use |
|---|---|---|
| `--ink` | `#0E1116` | Background — deep blue-black, like a dashboard at night |
| `--panel` | `#161B22` | Cards, code blocks |
| `--text` | `#D6DEE7` | Body text |
| `--muted` | `#8B98A5` | Captions, metadata |
| `--trace` | `#5CC8FF` | Links, the trace spans — instrument-panel cyan |
| `--p99` | `#FFB454` | One accent only: the latency numbers and the fixed span |

Light mode optional later; ship dark-first, it matches the subject.

### Typography

| Role | Face | Notes |
|---|---|---|
| Display | **Söhne Breit** fallback **Archivo Expanded / Archivo** | Wide, engineered, confident; used only for H1/H2 |
| Body | **Source Serif 4** | Long-form credibility; the site argues in prose, a serif carries it |
| Utility / data | **IBM Plex Mono** | Span labels, metrics, dates, the `p99 −33%` captions |

Scale: 1.25 ratio. Body 18px/1.65. H1 clamp(2.4rem, 6vw, 4rem).
Metrics and numbers always set in the mono face with the `--p99` accent — numbers
are the site's punctuation.

### Layout

```
┌────────────────────────────────────────────┐
│ name · headline (mono, muted)              │
│                                            │
│ DISTRIBUTED SYSTEMS DON'T FAIL LOUDLY.     │  ← display face
│ prose intro (serif, 2 sentences)           │
│ ▓▓▓▓▓░░░░  ▓▓▓░░  ▓▓▓▓░░   ← hero trace   │
│ [Read the proof]  [Message me]             │
├────────────────────────────────────────────┤
│ 1 · Build ……  claim + proof links          │
│ 2 · Evidence … claim + proof links         │  ← numbered: it IS a sequence
│ 3 · Method …   claim + proof links         │     (build → verify → generalize)
├────────────────────────────────────────────┤
│ Still shipping (stack line, mono)          │
│ Selected talks (4 rows: title·venue·year)  │
│ Footer: roles, contact, source link        │
└────────────────────────────────────────────┘
```

Single column, max-width 68ch for prose, full-bleed only for the hero trace.
Section markers `1 ·` / `2 ·` / `3 ·` in mono — justified because the three forms
are an actual sequence (build → verify → generalize), not decoration.

### Restraint rules

- One animation (hero trace), one accent color in use per screen.
- No card grids, no avatars-with-gradient-rings, no skill bars, no "About Me" selfie
  section; the pie-chart language image from the old site does not migrate.
- Proof links styled identically everywhere: `[video]` `[slides]` `[DOI]` `[source]`
  in mono — a consistent vocabulary, like a well-labeled dashboard.

---

## 6. Markdown / Content Formatting Conventions (for the repo)

- All content as `.md`/`.mdx` in `src/content/`; front-matter: `title`, `description`,
  `date`, `og_image`.
- Headings: sentence case, `#` reserved for the page H1, sections start at `##`.
- Numbers, dates, metrics wrapped in `<span class="metric">` (or an MDX shortcode)
  → renders in mono + accent.
- Proof links as reference-style Markdown at the bottom of each file — keeps prose
  diff-friendly.
- Code blocks only where code genuinely appears (case study); no decorative code.
- Images: SVG preferred (diagrams drawn in site palette), raster as WebP, all with
  meaningful `alt`.
- One `CONTENT.md` in the repo documenting these rules — the site practices what
  the owner preaches about leaving the method behind.

---

## 7. Build Order

1. Register domain, create repo, Astro skeleton + GitHub Actions deploy (half a day).
2. Home page with full §3 content and proof links — ship this alone if needed.
3. Hero trace SVG + design tokens.
4. /talks/ (curated list) and /publications/ (3 + Scholar link).
5. Trading-platform case study.
6. Banner + canonical on ingvord.ru pointing to the new domain.

Ship after step 2. Everything else is iteration on a live site.
