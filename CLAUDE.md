# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal portfolio site for 황해승 (game server/client developer), served as a GitHub Pages **user site** at `https://dakgg.github.io` from the `dakgg/dakgg.github.io` repo. Content is written in Korean (`<html lang="ko">`).

The entire site is two files — `index.html` and `style.css`. There is no build step, package manager, framework, test suite, or CI config. All JavaScript is a single inline `<script>` at the bottom of `index.html`.

## Commands

```bash
# Preview locally (no build required)
open index.html
# or, if you need a real origin (iframe embeds, etc.)
python3 -m http.server 8000

# Deploy — GitHub Pages publishes whatever is on master
git push origin master
```

## Architecture

### Project cards → hidden templates → modal

The Projects section works by cloning, not by fetching or templating. Three pieces must stay in sync:

1. `<button class="proj-card" data-detail="detail-pN">` — the visible card in `.proj-board`.
2. `<div class="proj-detail" id="detail-pN">` — the full detail markup, inside the `hidden` `.proj-details` container. This block is never displayed in place; it exists only as a template.
3. `#proj-modal` — a single reusable dialog. On click, `openModal()` deep-clones the matching `.proj-detail`, strips its `id` (to avoid duplicate IDs in the document), unsets `hidden`, and injects it into `#modal-body`.

Adding a project means adding both a card and a matching `.proj-detail` with an `id` equal to the card's `data-detail`. A mismatch silently does nothing — `openModal()` returns early.

There are **two** `.proj-board`s — `#projects` (work projects, `detail-pN`) and `#oss` (personal GitHub repos, `detail-gN`) — but only one `.proj-details` container and one `#proj-modal`. Both boards' detail blocks live in that single container, and the wiring needs no change to add a third board: `openModal()` resolves by `getElementById` and the click handler binds every `.proj-card` in the document. Each board numbers its own cards from `01`.

Ordering matters in `openModal()`: `modal.hidden` must be cleared **before** the clone is appended. An `<iframe>` inserted into a `display:none` subtree never starts loading, so injecting first leaves the YouTube embeds blank. For the same reason the detail iframes deliberately carry no `loading="lazy"` — they are already load-on-demand, since the element only exists once a card is clicked.

Accessibility wiring that is easy to break when editing: the clone's `.detail-title` is assigned `id="modal-title"` at runtime to satisfy the modal's `aria-labelledby`, so every `.proj-detail` needs a `.detail-title`. The modal also scrolls-locks `document.body`, focuses `.modal-close` on open, and restores focus to the triggering card on close; anything with `data-close` inside the modal acts as a close button, and Escape closes.

### Detail block vocabulary

Detail blocks are composed from a fixed set of optional pieces, in this order: `.result-row` (stat cards with `.big`/`.cap`), `.video-embed` (16:9 YouTube iframe — use `youtube-nocookie.com/embed/`), `.media-grid`, `.work` (the work-performed list), `.proj-body` (prose alternative to `.work`), `.tags`, `.detail-links`. Use `.work` for substantial projects and `.proj-body` for short ones — see `#detail-p5` and `#detail-p6` for the short form.

Inside `.work`, items live in a `ul.work-list` (custom accent bullets; `<strong>` promotes a lead-in phrase to full-contrast text). Wrap lists in `.work-group` — each with a `.work-label` heading — when a project splits its work into areas; `#detail-p4` shows the ungrouped form, a bare `.work-list`. Content mirrors the "수행 업무" sections of the Notion portfolio at `faceted-candle-946.notion.site/0d3a681ea9df4320af3876482892f732`, so edit there first and port across.

A prior revision used `.story` (three `.story-cell`s labelled PROBLEM / SOLUTION / IMPACT). That vocabulary is gone from both files — don't reintroduce it.

### Styling

`style.css` is plain CSS driven by custom properties on `:root`, with a full dark-mode override under `@media (prefers-color-scheme: dark)`. Change colors through the tokens (`--accent`, `--surface`, `--border`, `--text-muted`, …) rather than hardcoding values, or dark mode will break. `--maxw: 960px` governs the `.wrap` content width.

Responsive breakpoints are hand-placed next to the rules they modify: `.proj-board` steps 3 → 2 → 1 columns at 860px / 560px, `.story` and `.media-grid` collapse at 720px, and the career timeline collapses at 520px. There are `@media (prefers-reduced-motion: reduce)` blocks that disable smooth scrolling, button lift, and modal animation — extend them when adding animation.

Note: `.project-grid` / `.project-card` rules exist in `style.css` but the markup now uses `.proj-board` / `.proj-card`. Do not treat the old classes as live.
