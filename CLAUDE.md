# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-file static site: `index.html` (~3,450 lines) is a probability & combinatorics cheat sheet for quant interview prep — 10 sections, 83 concept cards, every worked example numerically verified. There is no build system, package manager, or test suite. All CSS lives in the one `<style>` block in the head and all JavaScript in the one `<script>` block before `</body>` (plus a tiny theme-bootstrap script in `<head>`); external dependencies (KaTeX 0.16.9, Google Fonts) load from CDNs with `defer`.

## Development

Open the file directly in a browser to view changes (`open index.html` on macOS), or serve it with `python3 -m http.server`. Verify math rendering visually: KaTeX is configured with `throwOnError: false`, so malformed LaTeX shows up as raw/broken text rather than a console error.

## Architecture

**Math rendering** — KaTeX auto-render runs on DOMContentLoaded with delimiters `$$…$$` (display) and `\(…\)` (inline). Single-dollar `$…$` is not registered and renders as literal text — bare `$` is therefore safe (and used) for currency in prose; `\$` is only valid *inside* math. Long display math scrolls horizontally inside `.formula-box`/`.cdf-box` (`overflow-x:auto` on `.katex-display`). The render call is guarded so a failed CDN load cannot break page interactivity. In math, keep spaces around `<`, `>`, and `&` so the HTML parser can't misread them.

**Theming** — `data-theme="dark"|"light"` on `<html>`, set before first paint by an inline `<head>` script (localStorage key `prob-theme`, falling back to `prefers-color-scheme`), toggled by the topbar button. Every color is a CSS custom property defined once per theme block; all foreground/surface pairs are verified ≥ 4.5:1 (WCAG AA) — if you add or change a color, re-verify contrast in BOTH themes computationally, not by eye. An `@media print` block redefines the tokens to an ink-friendly light palette (so dark-theme users get legible printouts), hides the chrome, and force-shows all solutions.

**Content structure** — 10 sections, each linked from the sticky `.topbar` nav: `#counting`, `#probability`, `#conditional`, `#ev`, `#joint`, `#distributions`, `#limit`, `#markov`, `#classics`, `#guide` (a decision table mapping question phrasings to tools). Sections hold a `.card-grid` of `<article class="card" id="c-SLUG">` cards, optionally grouped by `.subhead` dividers. Card anatomy:

- `.card-top`: `.card-head` (a `.card-label` colored by a per-section `label-*` class + a `.know-btn`), `<h3 class="card-name">`, `<p class="card-intuition">`, then `.formula-box` boxes (accent class `blue|teal|amber|violet|rose|green` paints the left border; extra boxes take class `stack`, side-by-side pairs sit in `.dual-formula`/`.triple-formula` with class `flush` and `.mini-label` captions; multi-line identity lists use classes `list` + `.identity-list`), optional `.cdf-box` and `.tip-box`.
- `.example-block`: `.example-tag`, `.example-q`, a `.reveal-btn` (with `aria-expanded` + `aria-controls`) containing `<span class="rb-label">Show solution</span>`, and a hidden `.solution` (id `sol-SLUG`) holding `.example-steps` → `.step` rows (each opening with a `.step-label` like `SET UP →`) plus a final `.example-answer`.

**Interactivity is convention-driven** — the script wires every `.reveal-btn`, `.know-btn`, and card generically off these classes and ids; a new card following the markup pattern (unique `SLUG` in `id="c-SLUG"`/`id="sol-SLUG"`/`aria-controls`) gets reveal, search, known-tracking, and progress behavior with no JS changes. Features: Reveal-all (state derived from the DOM, never a stored boolean), search filter over card names/labels/intuitions/questions, per-card "✓ Known" marking persisted in localStorage (`prob-known`) with per-section progress chips, Focus mode (hides known cards), active-section TOC highlight, back-to-top, and keyboard shortcuts (`/` search, `a` reveal all, `f` focus, `t` theme). All localStorage access is try/catch-guarded.

The hero's section/card counts are computed by JS at load — no need to update them when adding cards. The static text in the HTML is only a no-JS fallback.

**Content conventions** — E[X] (not `\mathbb{E}`), `\text{Var}`/`\text{Cov}`, ρ for correlation. Geometric and Negative Binomial use the *total-trials* convention (each card's note states the failures-convention alternative); Gamma uses shape α + *rate* λ. If you add a card, state the convention when more than one exists, keep intuitions ≤ 2 sentences, and numerically verify the example's answer (e.g. with Python fractions) before committing it. Consider whether the `#guide` decision table needs a matching row.
