# Git

- Never add a `Co-Authored-By:` trailer.
- Backtick code in commit messages so they need no editing as PR body.
- Keep commit messages short. The title usually says it; add a body only when it needs
  explaining, and then 1-2 sentences at most. No bullet lists, no measurements, no
  rationale essays.
- In WHATWG repos and wpt, prefix branches with `zcorpan/`. Nowhere else: in my own repos, and
  in others where I have push access, branch names take no prefix.
- In WHATWG spec repos, follow
  [whatwg/meta's COMMITTING.md](https://github.com/whatwg/meta/blob/main/COMMITTING.md):
  title ≤72 chars, imperative mood, no period; blank line then description (omit for simple fixes). Reference issues with closing keywords only when actually resolved (prefer "Fixes"). Prefixes: `Editorial: ` (formatting/typos), `Meta: ` (ecosystem), `Review Draft Publication: `.

# Writing

Applies to chat replies and GitHub/Bugzilla/spec drafts.

- Shortest thing that works, ~20 words median, one-line stays one-line.
- No preamble/wrap-up: don't restate, recap, or "let me know if". Start at the point, stop at the fact.
- No headings or bullet scaffolding. Bullets enumerate real cases, not argument structure.
- Hedge uncertainty: "I think", "seems", "as far as I can tell", "AFAICT", "Maybe".
- Prefer questions to demands.
- Let links carry evidence; use `#11410` for same-repo issues/PRs, not full URLs.
- Corrections: one plain sentence, no apology. Contractions. Avoid "Great question", "Certainly", "Note that".
- Limit em-dashes; use comma, colon, parentheses, or new sentence. Firefox = Fx. en-US spelling.

# Spec research

- Use `webspec-index` instead of scraping: `query HTML#concept-media-load-algorithm`, `search "tree order" -s DOM`, `refs HTML#navigate -d incoming`, `anchors "*-tree" -s DOM`, `idl Window.open\(\)`, `trace FROM TO`. Add `--format markdown`. For unmerged specs: `--pr N --diff` (grep the noise).
- `refs ... -d incoming` quickly answers "what invokes this algorithm" (the real question behind "when does this run").

# Measuring browser behaviour in wpt

- Throwaway test in scratch dir: `assert_true(false, '\n' + log.join('\n'))` to dump observations. Record sync, microtask, task, rAF, event handlers in one run. Delete scratch dir and verify with `git status`.
- `./wpt run --no-pause --yes --binary "<path>" <product> <paths>` (`--yes` skips webdriver prompt, `--no-pause` stops hanging).
- Local browsers: `/Applications/Firefox Nightly.app/Contents/MacOS/firefox` (`firefox`), `/Applications/Google Chrome Canary.app/Contents/MacOS/Google Chrome Canary` (`chrome`), `--webkit-port=safari safari` (release), `--channel preview` (Safari TP).
- Before Firefox wpt run, check for staged update: `cat "$HOME/Library/Caches/Mozilla/updates/Applications/Firefox Nightly/updates/0/update.status"`. If it exists, wait for update to apply (fresh profile otherwise spawns `org.mozilla.updater` which needs authorization and blocks on password prompt).
- Re-run timing-sensitive results ~3 times before reporting stable.
- When many failures exist, baseline before attributing to my change: `git stash`, run, `git stash pop`.
- Re-run with added settle delay to rule out artifacts from load event or queued tasks.

# Driving a browser at an external URL

- Chrome: `chrome --headless --disable-gpu --virtual-time-budget=8000 --dump-dom "<url>"`.
- Firefox/Safari: raw W3C WebDriver over HTTP (POST `/session`, `/session/{id}/url`, `/session/{id}/execute/sync`, DELETE `/session/{id}`). geckodriver at `_venv3/bin/geckodriver` in bootstrapped wpt. If safaridriver hangs, fall back to `./wpt run --channel preview` on local equivalent.

# Writing wpt tests

- Don't sync on parse-time inline script when behavior depends on algorithm resuming at microtask checkpoint (engines don't resume there). Poll instead: `await t.step_wait(() => video.currentSrc == source.src, 'desc', 3000, 5)` (adjust 100ms default as needed).
- Prefer sync point tied to spec step over events (events queue at different step, engines disagree on order).
- Sanity-check sync point: assert expected work is still pending, so test fails loudly if state is wrong.
- New `resources/` handler knobs: add optional query param behind presence check (existing callers unaffected).
- Test failing identically in all browsers for unrelated reason is worse than no test. Verify it would pass if feature were correct.

# Spec issues

- One issue per issue, cut tangents. If a second problem appears, file separately instead of "Related" paragraph.
- Problem + evidence, stop. One hedged sentence for fix is OK. Keep refs terse: "(Found in #123.)" not a clause.
- Verify fix impact carefully to avoid regressions.
- Quote spec as block quote (colon intro, "...and " continuation), not inline.
- Hedge implementation claims: "appear to run" not "runs". Report observed behavior, not code.
- If problem is spec-only and no browser distinguishes, omit browsers entirely. Don't report "couldn't reproduce".
- Live DOM Viewer demos: `https://software.hixie.ch/utilities/js/live-dom-viewer/?` + `encodeURIComponent(markup)`. Don't use `?saved=N`. Use `w()` not `console.log`. Link as `[demo](<permalink>)` (percent-encode `(` and `)`). Verify with `chrome --headless --virtual-time-budget=8000 --dump-dom "<url>"`.
- LDV test files (unqualified): `delayed-image`, `delayed-script`, `image`, `null`, `script`, `style`, `document`, `alertdoc`, `svg`, `xml`, `xml-broken`, `xhtml`, `download`.
- Never "all engines" or "all three". Name tested browsers/engines, pick one scheme, confirm channel: `./wpt run safari` (release), `--channel preview` (TP), check "Starting WebDriver:" if unsure.
- Chromium: <https://issues.chromium.org/issues/new?noWizard=true>. WebKit: `https://bugs.webkit.org/enter_bug.cgi?product=WebKit&component=<component>&short_desc=...&comment=...` (WAF blocks `<input`, `<iframe`, `<script`, `<body`, `<form`, `<svg`, `<textarea`, `<button`, `<object`, `<embed`, `<frame>`; `<a`, `<div`, `<p`, `<span`, `<math` pass; no Markdown). Pre-fill with `gh issue create --repo <org>/<repo> --web --title "..." --body-file <file>` (or `?title=`/`?body=` for YAML forms; check `.github/ISSUE_TEMPLATE/*.yml` for field ids).
- Don't put LDV permalinks in impl bugs. Link wpt test or describe repro + attach test file.

# Comments and replies

- Reply via block quote + answer pairs. Multiple pairs in one comment, no connecting prose.
- Short comments like "Typo" are fine.
- Code review: prefer suggestion block over prose for easy fixes; add rationale only if non-obvious.
- When suggesting fix, verify impact carefully to avoid regressions.
- Chat review findings: start with file:line/range (e.g., `source:126832-126843` at current HEAD), so you know where to leave comment. Say which commit.
