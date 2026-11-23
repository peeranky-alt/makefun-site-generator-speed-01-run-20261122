[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/peeranky-alt/makefun-site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/peeranky-alt/makefun-site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

# site-generator-speed-01 — fast drop-in for makefun-site 1.0.0

`solve.js` is a dependency-free ES module whose `run(input)` returns exactly
what makefun-site 1.0.0's `build` returns for the same input: the output tree
of the case's site, byte for byte. `main.ts` is the harness driver: it reads
every case from `/input/cases.json` (or `$MAKEFUN_INPUT_DIR`), answers each
with `run`, and writes `/output/answers.json` (or `$MAKEFUN_OUTPUT_DIR`).

## How it is faster than the reference

Same semantics, cheaper computation only:

- **Slug cache + ASCII fast path.** `slugify` results are cached (titles,
  tags and headings repeat across a site), and pure-ASCII inputs skip the
  NFKD normalisation entirely — NFKD is the identity on ASCII and ASCII has
  no combining marks, so the result is identical.
- **Buffered inline Markdown.** The reference escapes HTML one character at
  a time; this buffers literal runs and escapes them in bulk (escaping is
  per-character and context-free, so the output is unchanged).
- **Prefix-dispatched Markdown lexer.** Block classification dispatches on a
  line's first characters instead of running six regexes per line.
- **Accumulator template rendering.** Rendering appends into one array
  instead of building a fresh string per node.
- **Link cache.** Link resolutions are cached per (source directory, target);
  broken links are still reported once per (source, target), as the
  reference does.
- **One card per source.** Each source's card is built once and shared
  between lists and its page (templates only read cards).

No I/O, no clock, no randomness, no environment reads: the output is a
function of the input alone, exactly as the reference.

## Files

- `solve.js` — the entry (`run`).
- `main.ts` — the harness driver (same driver as the house baseline,
  including `--no-incremental-marking`).
- `STRATEGY.md` — what was optimised and what was deliberately not changed.
- `LICENSE` — MIT.
- `.makefun/owner.json` — payout wallet.
