---
name: conference-report
description: Research tech conferences/keynotes and generate visual HTML reports. Parallel web research → architecture diagram download → single-page or multi-page dark-theme HTML → source links → zip package. Use when user says "整理/调研/报告 某会议", "conference report", "keynote summary", or mentions specific conferences like Google I/O, AWS re:Invent, Microsoft Build, Apple WWDC.
argument-hint: [conference-name] [--multi-page] [--with-video URL]
---

# Conference Report

Generate a comprehensive, visually rich HTML report for any tech conference or keynote.

## Phase 1: Requirements

Ask the user:

1. **Conference name** — e.g. "Google I/O 2026", "AWS re:Invent 2026"
2. **Focus topics** (optional) — specific tracks, products, or themes to prioritize
3. **Output format**:
   - Single-page HTML (default) — one file, Tab/Section switching, easy to share
   - Multi-page HTML — index.html overview + separate detail pages per major topic
   - Markdown — plain text, no visuals
4. **Video analysis** — if user provides YouTube URLs, extract slides/diagrams via `mcp__zai-mcp-server__analyze_video`
5. **Language** — default to user's language (detect from conversation, default 中文)

## Phase 2: Parallel Research

Launch 2-3 `compound-engineering:ce-web-researcher` agents **in parallel**:

| Agent              | Scope                                      |
| ------------------ | ------------------------------------------ |
| Agent A            | Main keynote + major product announcements |
| Agent B            | Developer tools, SDKs, platform updates    |
| Agent C (optional) | Deep dive on user-specified focus topic    |

**Agent prompt template** (adapt per conference):

```
Research [CONFERENCE_NAME] keynote/announcements. I need:

1. Core announcements — what was launched, updated, deprecated
2. Developer impact — new APIs, SDKs, tools, breaking changes
3. Key numbers — stats, benchmarks, adoption figures
4. Architecture diagrams — any official architecture images (grab image URLs)

Search queries to try:
- "[CONFERENCE] [YEAR] keynote announcements"
- "[CONFERENCE] [YEAR] developer highlights"
- site:blog.[company].com [CONFERENCE] [YEAR]
- "[CONFERENCE] [YEAR] recap"

Return results in [LANGUAGE]. Be thorough — check official blogs, tech press, YouTube recaps.
For each finding, capture: what was announced, developer impact, any diagram/slide URLs.
```

## Phase 3: Video Analysis (Optional)

If user provided YouTube URLs:

1. Use `mcp__zai-mcp-server__analyze_video` with prompt: "Extract all key slides, architecture diagrams, product screenshots shown in this keynote. Describe each visual in detail."
2. Save extracted descriptions for report inclusion
3. Note: this step adds ~2-5 min per video

## Phase 4: Image Download

From research results, collect all official architecture diagram URLs. Download to `images/` directory:

```bash
mkdir -p "[output-dir]/images"
curl -sL -o "[output-dir]/images/[descriptive-name].jpg" "[URL]"
```

## Phase 5: Report Generation

### Output Directory

Create at `[working-dir]/[conference-slug]/` (e.g., `google-io-2026/`).

### Single-Page HTML (Default)

One `report.html` file with:

- **Dark theme** (`--bg: #090b12`, `--surface: #111520`, etc.)
- **Hero section** — conference name, date, key tags
- **Stats row** — key numbers in gradient-text cards
- **Tab-based sections** — one tab per major topic
- **Cards grid** — announcements in hover-animated cards with colored badges
- **Architecture figures** — images with Lightbox zoom (click to enlarge, ESC to close)
- **Comparison tables** — for product/model comparisons
- **Timeline** — for chronological announcements
- **Terminal blocks** — for CLI/code examples
- **Sources section** — all references grouped by category
- **All CSS embedded** — zero external dependencies

### Multi-Page HTML

When user specifies `--multi-page` or when content is very large (8+ major topics):

- `index.html` — overview, key stats, module cards linking to detail pages
- `[topic-slug].html` — one detail page per major topic
- `sources.html` — all source links
- All pages share same CSS template, cross-linked with `<a>` navigation
- Use sub-agents (Agent tool) to create detail pages **in parallel**

### Markdown

Simple structured Markdown with:

- Headers for sections
- Tables for comparisons
- Image references as `![desc](images/file.jpg)`
- Source links as bulleted list

## Phase 6: Source Links

Append a Sources section with ALL references, grouped:

| Group          | Content                                       |
| -------------- | --------------------------------------------- |
| Official blogs | blog.google, developers.googleblog.com, etc.  |
| Tech press     | Engadget, TechCrunch, Verge, 9to5Google, etc. |
| Video          | YouTube keynote links                         |

Format: `<a href="URL" target="_blank">Title</a>` + domain label.

## Phase 7: Package

```powershell
Compress-Archive -Path "[output-dir]/*" -DestinationPath "[conference-slug]-Report.zip" -Force
```

## Phase 8: Deliver

Report to user:

1. Output directory path
2. Zip file path and size
3. File structure listing
4. Top 3-5 key findings summary

## HTML Template (Embedded CSS)

Every HTML file must embed this CSS base (adjust as needed for single vs multi-page):

```css
:root{--bg:#090b12;--surface:#111520;--surface2:#1a1f30;--border:#2a2f48;--text:#e6e8f0;--text2:#8a90a8;--accent:#6ea8fe;--accent2:#b197fc;--blue:#4285F4;--green:#34A853;--red:#EA4335;--yellow:#FBBC05}
```

Full template includes: hero with floating orbs animation, sticky TOC nav, card grid with hover glow, stats row with gradient numbers, timeline with colored dots, terminal block with traffic-light dots, figure with Lightbox zoom, fadeUp scroll animation, IntersectionObserver for section reveal.

## Quality Checklist

Before delivering, verify:

- [ ] All major announcements covered
- [ ] Architecture diagrams downloaded and embedded
- [ ] Source links all `target="_blank"`
- [ ] HTML renders correctly offline (no external CDN deps)
- [ ] Zip contains all files including images/
- [ ] Stats/numbers cross-verified from 2+ sources
