# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

Single-file static web app — no build, no package manager, no tests. The entire application (HTML, CSS, JS) lives in `multiplication_quiz_app.html`. Deployed via GitHub Pages from `main`; the URL in `README.md` is the canonical preview.

To preview locally, open the HTML directly in a browser or serve it with any static server (e.g. `python3 -m http.server`). There is nothing to build or install.

## Application architecture

A Japanese 九九 (multiplication tables, 2–9) quiz with adaptive question selection and persistent per-cell stats.

- **State model (`stats`)**: 8×8 grid keyed `stats[a][b] = { attempts, wrong, times[] }`, persisted to `localStorage` under `quiz_stats_v1`. `migrateStats()` backfills missing cells/fields — preserve this on schema changes rather than breaking old saves. On schema-breaking changes, bump the key (`quiz_stats_v2`, …) and add a migration path.
- **Round structure**: Each round fixes a left operand (段) for `ROUND_SIZE` (20) questions. `pickLeftOp()` chooses it randomly; `pickRightOp()` weights the right operand by error rate (`w = 1 + errorRate*5`), considers both `stats[a][b]` and `stats[b][a]`, and avoids immediate repeats via `lastB`. This is the adaptive-difficulty core — changes here affect the learning loop.
- **Timer**: Tracks both session elapsed (displayed) and per-question elapsed (recorded into `times[]` on submit). `timer.resetQuestion()` must be called whenever a new question is shown.
- **Screens**: Single DOM tree with `#quiz-section`, `#heatmap-section`, and `#start-modal` toggled via the `.hidden` class — no router. `showHeatmap()` stops the timer; `restartGame()` re-opens the start modal. Cumulative stats survive `restartGame()`; only `resetAllRecords()` clears `localStorage`.
- **Heatmap**: Colors cells via `lerpColor(errorRate)` between green (`#1D9E75`) and red (`#D85A30`). Summary cards (累積正解率, 累積問題数, 平均解答時間, 最も苦手, 最も得意) are rebuilt from `stats` each time — no separate aggregate state.

## Commits

作業の区切りごとに適宜コミットしてください。1コミットは1つの論理的な変更にまとめ、コミットメッセージは既存の履歴のスタイル（短い英語の命令形、例: "Add timer and start modal, relocate cumulative counter"）に合わせます。

## Conventions

- UI text is Japanese; keep new strings consistent with the existing tone.
- Everything is wrapped in a single IIFE — no globals, no modules. Keep it that way unless splitting the file.
- CSS uses custom properties under `:root` with a `prefers-color-scheme: dark` override; use the existing tokens (`--accent`, `--correct`, `--wrong`, `--bg`/`--bg2`/`--bg3`, etc.) rather than hardcoding colors.
