# Git

- Commit title ≤72 chars, imperative mood, no period. Applies in every repo.
- Backtick code in commit messages so they need no editing as PR body.
- Keep commit messages short. The title usually says it; add a body only when it needs
  explaining, and then 1-2 sentences at most. No bullet lists, no measurements, no
  rationale essays.
- In WHATWG repos and wpt, prefix branches with `zcorpan/`. Nowhere else: in my own repos, and
  in others where I have push access, branch names take no prefix.
- In WHATWG spec repos, follow
  [whatwg/meta's COMMITTING.md](https://github.com/whatwg/meta/blob/main/COMMITTING.md):
  blank line then description (omit for simple fixes). Reference issues with closing keywords only when actually resolved (prefer "Fixes"). Prefixes: `Editorial: ` (formatting/typos), `Meta: ` (ecosystem), `Review Draft Publication: `.

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

- Use `webspec-index` (skill) instead of scraping. Add `--format markdown`. For unmerged specs: `--pr N --diff` (grep the noise).
- `refs ... -d incoming --kind step` quickly answers "what invokes this algorithm" (the real question behind "when does this run"). For `trace`, use `--max-depth 9`.

# Measuring browser behaviour in wpt

- Throwaway test in scratch dir: `assert_true(false, '\n' + log.join('\n'))` to dump observations. Record sync, microtask, task, rAF, event handlers in one run. Delete scratch dir and verify with `git status`.
- `./wpt run --no-pause --yes --binary "<path>" <product> <paths>` (`--yes` skips webdriver prompt, `--no-pause` stops hanging).
- Local browsers: `/Applications/Firefox Nightly.app/Contents/MacOS/firefox` (`firefox`), `/Applications/Google Chrome Canary.app/Contents/MacOS/Google Chrome Canary` (`chrome`), `--webkit-port=safari safari` (release), `--channel preview` (Safari TP).
- Before Firefox wpt run, check for staged update in its own command, before launching anything:
  `cat "$HOME/Library/Caches/Mozilla/updates/Applications/Firefox Nightly/updates/0/update.status"`.
  If the file exists at all (any content, including `applied`), stop and ask me to restart Fx Nightly;
  don't run wpt until the file is gone. A fresh profile otherwise spawns `org.mozilla.updater`, which
  needs authorization and blocks on a password prompt.
- Re-run timing-sensitive results ~3 times before reporting stable.
- When many failures exist, baseline before attributing to my change: `git stash`, run, `git stash pop`.
- Re-run with added settle delay to rule out artifacts from load event or queued tasks.

# Driving a browser at an external URL

- Chrome: `chrome --headless --disable-gpu --virtual-time-budget=8000 --dump-dom "<url>"`.
- Firefox/Safari: raw W3C WebDriver over HTTP (POST `/session`, `/session/{id}/url`, `/session/{id}/execute/sync`, DELETE `/session/{id}`). geckodriver at `_venv3/bin/geckodriver` in bootstrapped wpt. If safaridriver hangs, fall back to `./wpt run --channel preview` on local equivalent.
- Interactively via MCP servers (exploration only; report results from `./wpt run`):
  - `safari-tp-mcp`: `create_tab`, `navigate_to_url`, `evaluate_javascript`, `browser_console_messages`, `screenshot`. `evaluate_javascript` takes a function body: use explicit `return` or get `null`.
  - `chrome-canary-mcp` (isolated profile): `new_page`, `evaluate_script`, `list_console_messages`, `take_screenshot`. Every page tool needs `pageId` (from `new_page`/`list_pages`); `evaluate_script` takes a function (`() => ...`).
  - `firefox-nightly-mcp` (temp profile): `new_page`, `evaluate_script`, `screenshot_page`, `close_firefox_session`. `evaluate_script` takes a function (`() => ...`). No console tool by default.

# Writing wpt tests

- Await events/callbacks (wrap in a promise) rather than polling. Poll with `await t.step_wait(() => video.currentSrc == source.src, 'desc', 3000, 5)` (adjust 100ms default as needed) only for spec steps with no observable event, e.g. algorithm resuming at microtask checkpoint; don't sync those on parse-time inline script (engines don't resume there).
- Prefer sync point tied to spec step over events (events queue at different step, engines disagree on order).
- Sanity-check sync point: assert expected work is still pending, so test fails loudly if state is wrong.
- New `resources/` handler knobs: add optional query param behind presence check (existing callers unaffected).
- Test failing identically in all browsers for unrelated reason is worse than no test. Verify it would pass if feature were correct.
- Link the single-page HTML spec (`https://html.spec.whatwg.org/#anchor`), not `/multipage/...`, in `<link rel=help>` and elsewhere.
- Give each media/image resource in a test a distinct URL (add a query string) so a cached copy can't make a lazy resource load eagerly.
- Don't listen for `load` on a parser-inserted iframe/img from a later script; it may already have fired. Await the window `load` event instead (such elements delay it).
- Don't rely on named access on the global (`iframe` for `id=iframe`); use `document.querySelector()`/`getElementById()`.
- Don't add code comments unless really necessary for understanding.

# Spec issues

- One issue per issue, cut tangents. If a second problem appears, file separately instead of "Related" paragraph.
- Problem + evidence, stop. One hedged sentence for fix is OK. Keep refs terse: "(Found in #123.)" not a clause.
- Verify fix impact carefully to avoid regressions.
- Issues containing an AI-drafted plan: open with a first-person paragraph (not `<details>`) saying it was drafted with Claude Code (model name), what was explored, and which decisions I made; then put the whole plan in a block quote. Example: validator/validator#2143.
- Quote spec as block quote (colon intro, "...and " continuation), not inline.
- Hedge implementation claims: "appear to run" not "runs". Report observed behavior, not code.
- If problem is spec-only and no browser distinguishes, omit browsers entirely. Don't report "couldn't reproduce".
- Live DOM Viewer demos: `https://software.hixie.ch/utilities/js/live-dom-viewer/?` + `encodeURIComponent(markup)`. Don't use `?saved=N`. Use `w()` not `console.log`. Link as `[demo](<permalink>)` (percent-encode `(` and `)`). Verify with `chrome --headless --virtual-time-budget=8000 --dump-dom "<url>"`.
- LDV test files (unqualified): `delayed-image`, `delayed-script`, `image`, `null`, `script`, `style`, `document`, `alertdoc`, `svg`, `xml`, `xml-broken`, `xhtml`, `download`.
- Never "all engines" or "all three". Name tested browsers/engines, pick one scheme, confirm channel: `./wpt run safari` (release), `--channel preview` (TP), check "Starting WebDriver:" if unsure.
- Say "Safari TP" when the tested browser was Safari Technology Preview; plain "Safari" means release.
- Chromium: <https://issues.chromium.org/issues/new?noWizard=true>. WebKit: `https://bugs.webkit.org/enter_bug.cgi?product=WebKit&component=<component>&short_desc=...&comment=...` (WAF blocks `<input`, `<iframe`, `<script`, `<body`, `<form`, `<svg`, `<textarea`, `<button`, `<object`, `<embed`, `<frame>`; `<a`, `<div`, `<p`, `<span`, `<math` pass; no Markdown). Pre-fill with `gh issue create --repo <org>/<repo> --web --title "..." --body-file <file>` (or `?title=`/`?body=` for YAML forms; check `.github/ISSUE_TEMPLATE/*.yml` for field ids).
- Don't put LDV permalinks in impl bugs. Link wpt test or describe repro + attach test file.

# Comments and replies

- Reply via block quote + answer pairs. Multiple pairs in one comment, no connecting prose.
- Comments you post without me reviewing them first: attribute them. Line `Comment by Claude:`, blank line, then the entire body as ONE unbroken block quote. Every line gets `>`, including the blank separators between paragraphs (a bare `>`, never an empty line, or the quote ends and the rest renders as my own words). Quotes of other people go to `>>`. Only when I've read the draft do you post it unattributed as my own words.
- Short comments like "Typo" are fine.
- Code review: prefer suggestion block over prose for easy fixes; add rationale only if non-obvious.
- When suggesting fix, verify impact carefully to avoid regressions.
- Chat review findings: start with file:line/range (e.g., `source:126832-126843` at current HEAD), so you know where to leave comment. Say which commit.
