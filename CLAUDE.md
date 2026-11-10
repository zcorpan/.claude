# Scripting

- Avoid nontrivial bash/zsh (loops, variable expansion, word splitting); it doesn't always do what
  you expect. Prefer Python when possible.

# Git

- Use SSH remotes (`git@github.com:org/repo.git`), not HTTPS; e.g. after `gh repo create`, `git remote set-url`.
- Prefer linear history: fast-forward or rebase, not merge commits.
- Commit title ≤72 chars, imperative mood, no period. Applies in every repo except mozilla/firefox.
- Backtick code in commit messages so they need no editing as PR body (except in WHATWG spec
  repos, below).
- Keep commit messages short. The title usually says it; add a body only when it needs
  explaining, usually 1-2 sentences. Go longer only when the change genuinely needs more
  explanation (e.g. a non-obvious why); bullets are fine there if they list real cases.
  No measurements, no padding.
- In WHATWG repos and wpt, prefix branches with `zcorpan/`. Nowhere else: in my own repos, and
  in others where I have push access, branch names take no prefix.
- In WHATWG spec repos, follow
  [whatwg/meta's COMMITTING.md](https://github.com/whatwg/meta/blob/main/COMMITTING.md):
  blank line then description (omit for simple fixes). Reference issues with closing keywords only when actually resolved (prefer "Fixes"). Prefixes: `Editorial: ` (formatting/typos), `Meta: ` (ecosystem), `Review Draft Publication: `.
  No backticks in commit messages unless needed for disambiguation; add them in the PR
  description where needed. End the description with `Tests: <wpt PR URL>.`, then `Fixes #N.`

# Local checkouts

- Repos live at `~/git/<github-org>/<repo>`; check there before fetching remotely, and clone new
  ones there. E.g. `whatwg/*` specs and `html-build`, `w3c/*` specs, `web-platform-tests/wpt`,
  `mdn/content`, `mozilla/standards-positions`, `speced/bikeshed`.
- Engines, as full clones (history and all files) for `git grep`, `git log -S`, and pinned
  permalinks: `mozilla/firefox` (origin is the `mozilla-firefox/firefox` GitHub mirror),
  `chromium/chromium`, `webkit/WebKit`, `LadybirdBrowser/ladybird`.
- In `chromium/chromium`, third_party deps are gclient-managed: `git fetch` may print "Could not
  access submodule" and exit 1 after succeeding, so pass `-c submodule.recurse=false`.

# Claude Code plugins

- Bump `version` in `.claude-plugin/plugin.json` once per push that changes what the plugin
  ships (skills, hooks, commands), in a separate `Bump version to X.Y.Z` commit. Skip it for
  README-only or repo-tooling changes.
- Semver: minor for a new skill/hook/command or new behavior, patch for wording fixes and
  clarifications, major for removing or renaming a skill or command. While below 1.0, use minor
  for those too.

# Writing

Applies to chat replies and GitHub/Bugzilla/spec drafts.

- Shortest thing that works, ~20 words median, one-line stays one-line.
- No preamble/wrap-up: don't restate, recap, or "let me know if". Start at the point, stop at the fact.
- No headings or bullet scaffolding. Bullets enumerate real cases, not argument structure.
- Hedge uncertainty: "I think", "seems", "as far as I can tell", "AFAICT", "Maybe". When 100% certain
  (e.g. verified in source or spec), state it plainly without hedging.
- Prefer questions to demands.
- Let links carry evidence. On GitHub, use `#11410` for same-repo issues/PRs and `owner/repo#123`
  for other repos, not full URLs. For commits, a short SHA autolinks only in the same repo; link
  other-repo commits with the full `https://github.com/<org>/<repo>/commit/<full sha>` URL.
  Bugzilla and Phabricator don't autolink GitHub refs, so use full URLs there (`bug NNNNNN` and
  `DNNNNNN` are fine).
- Corrections: one plain sentence, no apology. Contractions. Avoid "Great question", "Certainly", "Note that".
- Limit em-dashes; use comma, colon, parentheses, or new sentence. en-US spelling.
- When abbreviating Firefox, use Fx (not FF). Say "Firefox" unless there's a reason to abbreviate.
- Never "all engines" or "all three". Name tested browsers/engines, pick one scheme, and state the channel.
- Say "Safari TP" when the tested browser was Safari Technology Preview; plain "Safari" means release.
- When listing several browsers, give versions for all or none (none is usually fine). Safari TP's
  version is its release number ("Safari TP 254"); the UA and app bundle lack it, so take the latest
  "Safari Technology Preview" entry's `displayVersion` in `/Library/Receipts/InstallHistory.plist`
  (read with Python `plistlib`).
- When suggesting a fix, verify impact carefully to avoid regressions.

# Spec research

- Use `webspec-index` (skill) instead of scraping.
- When reviewing spec changes, use `web-archeologist` (skill) to find out why the spec is the way it is before
  suggesting changes to it.

# Skills, tools and memories

- When editing a skill or writing a memory, state the general rule, not the incident: no case
  narrative and no specifics from the case that prompted it. Keep it compact and cut anything
  that doesn't change behavior.
- For creating, editing, or improving a skill, plugin, agent, hook, or a script a skill drives,
  use the `build-tooling` skill (zcorpan/agent-tooling).

# Comments and replies

- Reply via block quote + answer pairs. Multiple pairs in one comment, no connecting prose.
- Formatting block quotes: blank line, then the entire body as ONE unbroken block quote. Every line gets `>`, including the blank separators between paragraphs (a bare `>`, never an empty line, or the quote ends and the rest renders as my own words). Nested block quotes go to `>>`.
- Short comments like "Typo" are fine.
- Don't use backticks around <short sha>.
