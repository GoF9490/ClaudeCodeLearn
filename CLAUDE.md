# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-page Korean mental-arithmetic (암산) quiz. The entire app is one file, `index.html`, with inline CSS and JavaScript. There is no build step, package manager, test suite, or git repository.

`history.txt` is a record of the original request and the outcome (in Korean), not code. It notes that the app was never run in the authoring environment, so behavior has only been checked by reading the code.

## Commands

- Run: open `index.html` directly in a browser. There is no server or build.
- Test, lint, build: none configured. Verify changes by opening the page and playing through a quiz in a browser.

## Architecture (all in `index.html`)

- **Screens**: three `<section>` elements (`#start`, `#quiz`, `#result`). `show(id)` toggles the `on` class so only one is visible. The result screen's confirm button returns to `#start`.
- **Quiz flow**: `startGame()` builds 20 problems (10 level-0, 7 level-1, 3 level-2) via `makeProblem`, starts a 100 ms timer, and steps through them with `next()`. `submit()` compares `Number(input)` to the stored value `qs[i].v`. A correct answer advances; a wrong one adds the `bad` shake class and a message, and the same problem stays up so it can be retried. The timer runs from start to the final correct answer.
- **Problem generation**: the building blocks are chunk generators that return `{s: text, v: value, n: digit-count}`:
  - `N(m)` a single number 1..m
  - `M(a,b)` multiplication
  - `D(b,q)` exact division (the dividend is built as `b*q`, so results are integers)
  - `P(m)` one parenthesized `(x ± y)` with a single `×` or `÷` by a small integer
  - `LV[level]` lists the candidate chunk combinations for each difficulty. Difficulty is the set of chunk combinations and operand sizes, not a numeric score.
  - `makeProblem(lv)` picks a candidate, joins its chunks with random `+`/`−`, and retries until the result is ≥ 1 at every step, ≤ 99999, and the total digit count across chunks is ≤ 4.
- **Constraints the generator enforces**: natural-number answers, a maximum of 4 terms, operands up to 3 digits, results up to 5 digits, and at most one pair of parentheses. Keep these when changing the generator.

## Known gaps

- The answer field is `inputmode="numeric"` but is a text input, so non-digit characters can still be typed. `history.txt` flags this as a known issue that was not fixed.
- Requests in `history.txt` were written in Korean and the UI text is Korean. Keep user-visible strings in Korean unless asked otherwise.
