# Git

- Never add a `Co-Authored-By:` trailer to commit messages.
- Put backticks around tags, attributes, filenames, and other inline code in commit messages, so
  that the text needs no editing when it becomes a PR body.
- In WHATWG repos and wpt, prefix branch names with `zcorpan/`.
- In WHATWG spec repos, follow
  [whatwg/meta's COMMITTING.md](https://github.com/whatwg/meta/blob/main/COMMITTING.md):
  - Title line of at most 72 characters, in the imperative mood ("fix", "add", "change"), with no
    trailing period.
  - Then a blank line and a description, which may be omitted for simple fixes. The description
    has no line-length limit, and a blank line separates paragraphs. Reference the relevant issue
    there, using a closing keyword only when the change actually resolves it (prefer "Fixes" over
    "Closes" if it fixes an issue).
  - Most commits have no prefix. The case-sensitive ones that exist are `Editorial: ` (formatting,
    typos, or a refactoring that does not change how the standard is understood), `Meta: ` (does
    not directly affect the text of the standard, but the ecosystem around it, such as spec tooling
    or contributor documentation), and `Review Draft Publication: `.

# Writing

Applies to chat replies as well as to anything drafted for GitHub, Bugzilla, or a spec.

- Write the shortest thing that works. My own comments run one or two sentences; the median is
  about 20 words. A one-line answer stays a one-line answer.
- No preamble and no wrap-up. Don't restate the question, don't recap what was just said, don't
  end with "let me know if". Start at the point and stop at the last fact.
- No headings, bold labels, or bullet scaffolding in a reply. Bullets are for enumerating real
  cases (per-browser behavior, options, scenarios to test), not for structuring an argument.
- When there's some uncertainty, hedge instead of asserting: "I think", "seems", "as far as I can
  tell", "AFAICT", "Maybe", "I believe".
- Prefer a question to a demand: "Is that intended?", "How is this observable and testable?",
  "Do we know if there's a web compat reason for this?"
- Let links carry the evidence. A spec anchor, a searchfox or Chromium source link, a wpt.fyi
  query, or an LDV permalink, with at most a clause of explanation. Write issues and PRs as
  `#11410`, not as full URLs, when they're in the same repo.
- Corrections are one plain sentence, then move on: "I was mistaken, it does still return null."
  No apology, no post-mortem.
- Contractions and plain words. Never "Great question", "Certainly", "You're absolutely right",
  "It's worth noting", "Note that", "In summary".
- Don't overuse em-dashes. Prefer a comma, a colon, parentheses, or a second sentence.
- Abbreviate Firefox as Fx, never FF.
- Use en-US spelling.

# Spec research

- Use the `webspec-index` CLI instead of fetching and scraping spec HTML. It resolves anchors,
  full algorithm text, and the call graph: `webspec-index query HTML#concept-media-load-algorithm`,
  `search "tree order" -s DOM`, `refs HTML#navigate -d incoming`, `anchors "*-tree" -s DOM`,
  `idl Window.open\(\)`, `trace FROM TO`. `--format markdown` reads best.
- For unmerged spec changes: `webspec-index query SPEC#anchor --pr N --diff`. The diff carries a
  lot of link-normalisation noise, so grep it for the terms I care about rather than reading it
  whole.
- `refs ... -d incoming` is the fast way to answer "what invokes this algorithm", which is
  usually the real question behind "when does this run".

# Measuring browser behaviour in wpt

- To find out what browsers do, rather than assert what they should do, write a throwaway
  testharness test in a scratch dir at the repo root and dump the observations through a
  deliberate failure: `assert_true(false, '\n' + log.join('\n'))`. The message shows up in the
  `./wpt run` output. Record several observation points in one run (sync, microtask, task, rAF,
  event handlers) so the whole timeline is visible at once.
- Delete the scratch dir afterwards, and actually re-check `git status` before saying the tree
  is clean.
- `./wpt run --no-pause --yes --binary "<path>" <product> <paths>`. `--yes` skips the
  interactive webdriver-install prompt, `--no-pause` stops it hanging.
- Local browsers:
  `/Applications/Firefox Nightly.app/Contents/MacOS/firefox` (`firefox`),
  `/Applications/Google Chrome Canary.app/Contents/MacOS/Google Chrome Canary` (`chrome`),
  `--webkit-port=safari safari` for release Safari, plus `--channel preview` for Safari TP.
- Before running `./wpt run` for Firefox, check for a staged update:
  `cat "$HOME/Library/Caches/Mozilla/updates/Applications/Firefox Nightly/updates/0/update.status"`.
  If that file exists, ask me to launch Firefox Nightly and let the update apply before proceeding.
  Every launch with a fresh profile otherwise spawns `org.mozilla.updater`, which needs
  authorization to replace the app in `/Applications` and blocks startup behind a password prompt.
  Profile prefs cannot suppress it, since the updater runs before they are read.
- Re-run timing-sensitive results about 3 times before reporting them as stable.
- In a directory that already has many failures, get a baseline before attributing any of them
  to my change: `git stash`, run, `git stash pop`, compare the counts.
- Before reporting a failure as real, re-run it with an added settle delay. If the result is
  identical, it isn't an artifact of the load event or a queued task arriving late.

# Driving a browser at an external URL

- Chrome can dump a page after timers have run:
  `chrome --headless --disable-gpu --virtual-time-budget=8000 --dump-dom "<url>"`.
- Firefox and Safari have no equivalent. Raw W3C WebDriver over HTTP with `urllib` is the
  least troublesome route (wpt's `tools/webdriver` did not expose a usable `Session`): POST
  `/session`, POST `/session/{id}/url`, POST `/session/{id}/execute/sync`, DELETE `/session/{id}`.
  geckodriver is at `_venv3/bin/geckodriver` in a bootstrapped wpt checkout. If STP's
  safaridriver hangs on session creation, fall back to `./wpt run --channel preview` on a local
  equivalent of the page.

# Writing wpt tests

- Don't synchronise a test on a parse-time inline script when the behaviour under test depends
  on an algorithm resuming at a microtask checkpoint. No engine resumes "await a stable state"
  there, so the mutation lands before the algorithm has started and the test quietly measures
  something else, often still failing and looking meaningful. Poll for the state the test
  actually depends on instead:
  `await t.step_wait(() => video.currentSrc == source.src, 'desc', 3000, 5)`.
  `step_wait`'s default 100ms interval is often too coarse for a short window.
- Prefer a sync point tied to the spec step I care about over an event. Events are queued at a
  different step from the state change, and engines disagree about the relative order.
- Sanity-check the sync point with an assertion that the expected work is still pending, so the
  test fails loudly if it ever lands in the wrong state.
- When a shared `resources/` handler needs a new knob, add an optional query parameter behind a
  presence check so existing callers are byte-for-byte unaffected.
- A test that fails identically in every browser for a reason unrelated to its subject is worse
  than no test. Check what the test would report if the feature were correct before keeping it.

# Spec issues

- One issue per issue. Cut tangents, even closely related ones. If I notice a second problem
  while writing up the first, offer it as a separate issue instead of appending a "Related"
  paragraph. Also cut asides that only re-support a point already made.
- State the problem, show the evidence, stop. One hedged sentence proposing a fix at the end
  is welcome, e.g. "I think moving the step to after the stable state would fix this.", but
  nothing longer. Keep incidental references terse: "(Found while reviewing #123.)", not a
  clause explaining the connection.
- When suggesting a fix, check carefully what the impact is of that fix to avoid introducing
  unintended regressions.
- Quote spec text as its own block quote, introduced by a colon and picked up afterwards with
  "...and ". Don't splice quotations inline into my own sentence.
- Hedge claims about implementations: "appear to run it in a task" rather than "run it in a
  task". I am reporting observed behaviour, not what the code does.
- When the problem is in the spec text and no browser distinguishes the cases, leave browsers
  out of the issue entirely. Don't add a paragraph reporting that I could not reproduce it.
  Tell me in the conversation instead.
- Build demos as Live DOM Viewer permalinks instead of inline code blocks. The URL is
  `https://software.hixie.ch/utilities/js/live-dom-viewer/?` followed by
  `encodeURIComponent(markup)`. Don't use the `?saved=N` form, which needs a server-side POST.
  LDV injects `w(thingToLog)` as the logging function, so use that rather than
  `console.log`, and omit optional tags and `<title>` and anything unnecessary in the markup.
- LDV serves test files relative to its own directory, so reference them unqualified:
  `delayed-image` (a GIF after a 2s pause, good for parking a load), `delayed-script`,
  `image`, `null`, `script`, `style`, `document`, `alertdoc`, `svg`, `xml`, `xml-broken`,
  `xhtml`, `download`. Full list is on the LDV page.
- Link them as `[demo](<permalink>)`, using "demo" as the link text, not "live" or the bare
  URL. "Demo" is fine when it starts a sentence. Percent-encode `(` and `)` as well, since
  `encodeURIComponent` leaves them alone and the first `)` in the markup would otherwise
  terminate the Markdown link early.
- Verify every permalink actually reproduces before including it. Loading it headless and
  reading the log pane works:
  `chrome --headless --virtual-time-budget=8000 --dump-dom "<permalink>"`.
- Never write "all engines", "every engine", or "all three". Testing three browsers is not
  testing every engine. Name what I actually tested, and pick one naming scheme and stick to
  it: either browser names (Firefox Nightly, Chrome Canary, Safari TP) or engine names
  (Gecko, Chromium, WebKit), never both together. Confirm the channel before labelling it:
  `./wpt run safari` drives release Safari, and only picks Safari TP's safaridriver with
  `--channel preview`. Check the "Starting WebDriver:" line under `--log-mach=-` if unsure.
- Don't hand me issue text to copy-paste. Open the form pre-filled instead:
  `gh issue create --repo <org>/<repo> --web --title "..." --body-file <file>`.
  Repos with `blank_issues_enabled: false` still accept `?title=`/`?body=` this way. To
  prefill a YAML issue *form* field-by-field instead, the query key must match the field's
  `id`, so check `.github/ISSUE_TEMPLATE/*.yml` first. Fields without an `id` can't be
  prefilled, and `--body` on the blank form is the fallback.
- To file a Chromium bug, open <https://issues.chromium.org/issues/new?noWizard=true>. Without
  `noWizard=true` the wizard takes over and nothing is prefilled.

# Comments and replies

- Reply by quoting the sentence I'm answering as a block quote, then answering underneath.
  Several quote-and-answer pairs in one comment, with no connecting prose, is the normal shape
  and is better than one flowing essay.
- Procedural comments are one line: "Suggest positive.", "Closing per #1896 (comment)",
  "Filed <url>", "Tests here <url>", "Typo", "cc @foo".
- When closing or deciding something, give the reason and a link, not a justification
  paragraph.
- In a code review, for easy fixes, prefer a GitHub ```suggestion block with the exact
  replacement over prose describing the change. Add a sentence of rationale only when the
  change isn't self-evident.
- When suggesting a fix, check carefully what the impact is of that fix to avoid introducing
  unintended regressions.
- When reporting review findings back to me in chat, start each one with the file and line
  number or range it anchors to, e.g. `source:126832-126843`, against the current head commit,
  so I know where to leave the review comment. Say which commit the line numbers are against.
