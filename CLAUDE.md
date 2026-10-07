# Scripting

- Avoid nontrivial bash/zsh (loops, variable expansion, word splitting); it doesn't always do what
  you expect. Prefer Python when possible.

# Git

- Use SSH remotes (`git@github.com:org/repo.git`), not HTTPS; e.g. after `gh repo create`, `git remote set-url`.
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
- Hedge uncertainty: "I think", "seems", "as far as I can tell", "AFAICT", "Maybe". When 100% certain
  (e.g. verified in source or spec), state it plainly without hedging.
- Prefer questions to demands.
- Let links carry evidence; use `#11410` for same-repo issues/PRs, not full URLs.
- Corrections: one plain sentence, no apology. Contractions. Avoid "Great question", "Certainly", "Note that".
- Limit em-dashes; use comma, colon, parentheses, or new sentence. When abbreviating Firefox, use Fx (not FF). en-US spelling.
- Never "all engines" or "all three". Name tested browsers/engines, pick one scheme, and state the channel.
- Say "Safari TP" when the tested browser was Safari Technology Preview; plain "Safari" means release.
- When suggesting a fix, verify impact carefully to avoid regressions.

# Spec research

- Use `webspec-index` (skill) instead of scraping.
- When reviewing spec changes, use `web-archeologist` (skill) to find out why the spec is the way it is before
  suggesting changes to it.

# Comments and replies

- Reply via block quote + answer pairs. Multiple pairs in one comment, no connecting prose.
- Comments you post without me reviewing them first: attribute them. Line `Comment by Claude:`, blank line, then the entire body as ONE unbroken block quote. Every line gets `>`, including the blank separators between paragraphs (a bare `>`, never an empty line, or the quote ends and the rest renders as my own words). Quotes of other people go to `>>`. Only when I've read the draft do you post it unattributed as my own words.
- Short comments like "Typo" are fine.
- Code review: prefer suggestion block over prose for easy fixes; add rationale only if non-obvious.
- Chat review findings: open with "Reviewed at <short sha>" (plus related PRs reviewed together). Group findings under bold **Blockers** (wrong behavior, regressions), **Suggestions** (should change, but judgment call), **Nits** (wording, style); omit empty groups. One bullet per finding: start with file:line/range (e.g., `source:126832-126843`) or, for spec algorithms, the italicized algorithm name and step, then a colon; state the problem with a concrete case, then one sentence on the fix ("Consider ...", or "Either X, or confirm Y is intended").
