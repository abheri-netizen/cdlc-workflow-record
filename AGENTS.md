# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this project is

A static reference site showing how SkillCat's work is configured in Jira, styled to
look like Jira itself. It is a lookalike, not a connection: the page makes no network
calls and cannot read or write Jira. Read `README.md` for the content model and
`HANDOFF.md` for operational setup.

## The one rule that matters

**Never edit `index.html` directly.** It is generated output. Any change you make there
is destroyed the next time anyone runs the build.

Edit these instead:

- `data/*.json` for content. This is the single source of truth
- `src/page.html` for layout and markup. It is a body fragment with a data placeholder

Then regenerate:

```bash
node build.mjs
```

After changing the Epic model specifically, run `node scripts/build-cdlc.mjs` first,
because `data/cdlc-epic.json` is itself generated from `worktypes.json` and
`transitions.json`.

## Verifying your work

The build is deterministic. Running it twice must produce a byte-identical
`index.html`. This is the fastest correctness check available:

```bash
node build.mjs && cp index.html /tmp/a && node build.mjs && cmp /tmp/a index.html
```

If those differ, the build has acquired a nondeterministic input and that is a bug
worth fixing before anything else.

To preview: `python3 -m http.server 8000`

## Environment

Node only. There are no npm dependencies, no `package.json` and no lockfile. Every
script uses Node builtins. Do not add a dependency without a strong reason, because
zero-install is a deliberate property of this project and the main reason it is cheap
to maintain.

## Never commit these

- **`redaction-map.json`** maps real employees' names to their roles. It is gitignored
  and has never been committed. Keep it that way. `redaction-map.example.json` is the
  public template
- **Jira credentials.** `scripts/refresh-usage.mjs` reads `JIRA_BASE_URL`,
  `JIRA_EMAIL` and `JIRA_API_TOKEN` from the environment. They belong in repository
  secrets or a local shell, never in a file
- Verify with `git status` before committing. The repo has no secret scanning

## Data integrity

The content came from live Jira, not from the SOP document. Where the two disagreed,
live Jira won. If you are adding or correcting content, derive it from a Jira export
rather than from samples, screenshots or memory, and say in the commit message which
export you used.

Do not invent ticket counts, status names or transitions. An unknown is better recorded
as an honest empty state, which is what the QFT space already does, than guessed at.

## Commit style

`<type>: <description>` using feat, fix, refactor, docs, chore. Lower case, present
tense. Match the existing log.
