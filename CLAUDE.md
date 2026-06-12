# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A parody clickbait headline generator ("ISBROKEN.COM"). The user types a subject and picks a tone; the page renders a fake YouTube thumbnail, video meta, and a list of related panic-headlines.

The whole site is one static file: [index.html](index.html). No build step, no backend, no env vars, no dependencies. Drop it on any static host (Vercel, Netlify, GitHub Pages, S3) and it runs.

## Architecture

Everything lives in [index.html](index.html):

- **Phrase bank** (`PHRASES` in the script block) — per-tone arrays of templated strings. Four tones (`outrage`, `conspiracy`, `health`, `society`), each with four arrays: `thumb`, `video`, `breaking`, `sub`. A `CHANNELS` list provides the on-screen network name.
- **Templating** — two tokens: `{S}` is replaced with the subject in caps, `{s}` with the subject as the user typed it. Use `{S}` for thumbnail headlines and breaking-news scrolls (all-caps contexts), `{s}` for sentence-style video titles and subheadlines.
- **`generate()`** — picks one random template from each pool, picks 4 unique subheadlines via `pickN`, fills the tokens, and calls `renderCard`. Wrapped in a ~280ms `setTimeout` so the loading spinner is visible — the "machine generating outrage" feel is part of the joke, not latency.
- **`renderCard()`** — paints the thumbnail. Uses a regex (~line 1182) that matches the uppercased subject plus the literals `IS BROKEN` / `BROKEN`, wrapping the subject in `.broken-word` (yellow) and the broken-text in `.is-broken` (red box). It assumes the thumbnail template contains the subject in caps and the word "BROKEN" — every entry in `PHRASES[*].thumb` must satisfy that, or the highlight breaks. It also sets the per-tone background via `bgPalettes` and a random duration badge from `durations`.

## Adding or editing phrases

Just edit the relevant array in `PHRASES`. Constraints:
- `thumb` entries: must contain `{S}` and the word `BROKEN` somewhere (so the highlight regex has something to wrap).
- `breaking` entries: all-caps, short.
- `video` and `sub` entries: full sentences; use `{s}` so capitalization matches the user's input.
- Each tone needs at least 4 `sub` entries (the renderer picks 4 unique ones).

Avoid apostrophe escaping headaches by keeping the arrays in backtick strings.

## Running locally

`open index.html` — no server required. Any static-file server works too (`python3 -m http.server`, `npx serve`, etc.).
