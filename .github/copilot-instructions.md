# Cosmic Cadet

## Run and validation

Open `index.html` directly in a browser; the app must work from a `file://` URL without a server or dependencies. On Windows, `Start-Process .\index.html` opens it in the default browser. Use the integrated browser when available.

Validate changes in the browser:

- For a single correct-answer check, reload and press `2` on question 1 (Yuri Gagarin). Expect `0100` points, locked answers, a green correct state, explanatory feedback, and the center alien disappearing after the shot.
- For a single wrong-answer check, reload and press `1` on question 1. Expect `0000` points, a red wrong state, the correct answer revealed, explanatory feedback, and a missed shot with the alien still visible.
- Complete ten questions to check results, then use "Launch another mission" to check reset. Five correct answers must produce 500 points, 5/10, 50% accuracy, and five missed flight-log segments.
- Check number shortcuts, native Tab/Enter/Space button activation, focus transitions, light/dark themes, reduced motion, and narrow viewports. Synthetic `KeyboardEvent` dispatch tests shortcut handlers but does not verify native Tab traversal or default button activation.

## Architecture

The entire app is in `index.html`: inline CSS, HTML screens, reusable SVG symbols, and a strict-mode script. Keep runtime assets and logic self-contained, including fonts and artwork.

The canonical `questionBank` holds twenty entries. `curatedIds` defines the original ten-question mission. `mission` is a fresh ten-question copy: curated mode preserves order; shuffled mode samples without replacement and remaps shuffled choices to the correct index. `startMission(mode)` owns all replay resets.

The script owns an in-memory quiz state: `current` is the zero-based mission index, `score` accumulates points, `answered` locks each question, and `outcomes` records `{ questionId, selectedIndex, correct }` per answered question. The selected index belongs to the active mission's displayed choices, not the canonical bank. Reloading starts a curated session.

The UI flow is `renderQuestion()` -> `choose(index)` -> Next -> either another question or `showResults()`. `updateProgress()` derives progress from answered questions (`outcomes.length`), not the displayed question number. Restart resets all state and reuses the same rendering functions.

Answer buttons are generated with DOM APIs. `$()` resolves element IDs; preserve the relationship between HTML IDs and script lookups. Quiz/results visibility uses the `hidden` property and the `[hidden]` CSS rule.

## Quiz and interaction conventions

- Keep ten-question missions drawn from twenty bank entries with four answers each. Bank objects have stable `id`, `category`, `question`, `answers`, zero-based `correct`, and `explanation`. Never mutate bank choices when shuffling. Ten-question assumptions remain in markup and result rankings; changing mission length requires updating those together.
- Award 100 points per correct answer, with no penalty for wrong answers. Results derive correct count from `score / 100`; the dashboard score is padded to four digits.
- Both click and number-key paths go through `choose()`. Its guard prevents multiple scores or answers after results. Selecting an answer disables all choices and enables Next; wrong selections also reveal the correct choice.
- Number keys 1-4 choose answers. The Enter shortcut advances from non-button targets; focused buttons retain native keyboard activation. Repeated and Alt/Ctrl/Meta-modified key events are ignored.
- After answering, focus moves to Next without scrolling; the viewport stays stable while battle animations run in place. New questions and results focus their `tabindex="-1"` headings.
- Battle state is CSS-driven: `.battle.hit` fires at and eliminates `.target`; `.battle.miss` fires past it. `renderQuestion()` clears both classes, restoring the target. Do not make scoring or navigation depend on animation completion.
- Results use native `details`/`summary` for missed-answer review. Each entry resolves the selected and correct answers from the active mission. Review must not mutate outcomes or scoring; either replay clears and collapses it.
- Personal bests use `cosmic-cadet.bests.v1` in localStorage: `{ version: 1, curated, shuffled }`. Each mode's value is null or an integer from 0 to 1000 in 100-point steps. Update only on first completion or improvement. Storage failures must produce visible status and retain in-session records; never imply an unsaved record persisted.
- Validate shuffled replay against the canonical answer text, not memorized answer positions. Check mode-specific bests across reload, ties, zero scores, malformed data, and blocked storage. Browser-local file storage can differ from the integrated preview origin.

## Visual and accessibility conventions

- Preserve the Space Invaders arcade styling: inline pixel SVG symbols, sans-serif body text at 15px, monospace status labels, and a centered column capped at 540px.
- Theme colors are CSS custom properties in `:root`, overridden by `prefers-color-scheme: light`. Update both palettes when adding or changing semantic colors.
- Keep `--answer-border` separate from decorative `--border`. Answer outlines must contrast at least 3:1 against both the card and answer fill; normal text must meet 4.5:1. Let CSS transitions settle before measuring computed colors after a theme change.
- Correct/wrong feedback combines color with check/cross marks, explicit text, answer `aria-label`s, and the atomic polite status region. Preserve these non-color cues.
- Progress exposes its value and answered-question description through ARIA. Decorative battle artwork, emoji reaction, and flight-log graphics are hidden from assistive technology; textual results convey the outcome.
- `prefers-reduced-motion: reduce` disables animations and transitions. Its explicit `.battle.hit .target` opacity rule still shows the eliminated state without animation.
