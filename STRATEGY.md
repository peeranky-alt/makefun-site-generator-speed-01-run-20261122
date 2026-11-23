# Strategy

## Goal

Beat makefun-site 1.0.0 on CPU time while returning byte-for-byte identical
output on every case. Exactness gates speed: any divergence fails the case,
so every optimisation below preserves the reference semantics exactly and
was verified against the practice set (3/3 exact).

## What was optimised

1. **Slugify** — the hottest pure function. Called for every title, every
   tag (twice per tag in the reference), and every heading. Two changes:
   a module-level cache (slugify is pure; titles/tags/headings repeat), and
   an ASCII fast path that skips `normalize('NFKD')` plus the `\p{M}` strip.
   NFKD is the identity on ASCII and ASCII contains no combining marks, so
   this cannot change a result. Non-ASCII inputs take the reference path.
2. **Inline Markdown** — the reference calls `escapeHtml` once per literal
   character. Literal runs are now buffered and escaped in bulk; escaping
   is per-character and context-free, so bulk escaping is identical.
   `findUnescaped` / `findSingleStar` keep the reference's exact scan
   (including skipping backslash-escaped characters).
3. **Markdown lexer** — block classification dispatches on the line's first
   character(s) (fence / heading / hr / quote / list / paragraph) instead of
   testing six regexes on every line. Each classifier re-implements its
   regex by hand with the same acceptance (e.g. ordered lists accept 1–9
   digits, headings 1–6 `#` plus a space/tab, list continuations are
   exactly `/^ {2,}\S/`).
4. **Template rendering** — `renderNodes` builds a string per node via `+=`;
   rendering now appends into a single array joined once at the top. Lookup,
   truthiness and `textOf` are unchanged.
5. **Link resolution** — resolved hrefs are cached per (directory, target).
   A cached broken link still adds its `broken-link <source> -> <target>`
   report line per source, exactly as the reference does.
6. **Cards** — the reference rebuilds a post's card for its page after
   building it for the lists (a second word count and tag map per post).
   Each source's card is built once and shared; templates only read cards.

## What was deliberately NOT changed

- Front-matter parsing, template parsing/tokenising, the build pipeline
  order, the report format, and every error string are the reference's,
  line for line where they are cheap (they run once per file/template).
- No template is compiled to generated code: the risk of a semantic drift
  in scope lookup / include depth / `loop` handling outweighs the gain at
  this size, and the accumulator renderer already removes the main cost.
- No lookup tables or precomputed answers are shipped: the scored set is
  drawn after entries close, so memorising the practice set would be
  useless as well as wrong.

## Measured

- Practice set: 3/3 cases exact vs `practice/reference/expected.json`.
- Baseline (house makefun-site) on this machine: ~1.0 s for the 3 practice
  cases; this entry: see `SUBMIT_STATE.md` for the side-by-side timing.
