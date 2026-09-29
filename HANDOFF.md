# Handoff

Sampurna Pal built and maintained this. This file is for whoever picks it up.

## What you just got

A reference site for how SkillCat's work is configured in Jira, styled to look like
Jira itself. It covers four spaces: CDLC (complete), SGP and CIP (basic), QFT (empty
state). See `README.md` for what each page shows and where the structure came from.

**The site is one self-contained file.** `index.html` is 133 KB, has no CSS files, no
JavaScript bundle, no fonts and no CDN links. It makes zero network calls. Double-click
it and it works offline. That is the whole site.

## Running it

You need Node. There are no npm dependencies, no `package.json`, no `npm install`.

```bash
node build.mjs          # regenerates index.html
```

To preview:

```bash
python3 -m http.server 8000     # then open http://localhost:8000
```

Edit `src/page.html` or the JSON files in `data/`. Everything else is generated. The
build is deterministic, so running it twice gives a byte-identical `index.html`. If it
does not, something is wrong.

After changing the Epic model specifically:

```bash
node scripts/build-cdlc.mjs
node build.mjs
```

## Four things that will not work until you set them up

These broke by design when Sampurna's accounts were deactivated. Nothing here is a bug.

**1. `redaction-map.json` is missing.** It is gitignored and deliberately not in this
package, because it maps real colleagues' names to their roles. `scripts/redact.mjs`
will not run without it. Copy `redaction-map.example.json` to `redaction-map.json` and
fill it in, or ask Sampurna for the original. Format is an array of
`["text to find", "text to replace with"]` pairs, longest first so that "Daniel L Riggs"
is matched before "Daniel".

**2. The Jira API token is gone.** `scripts/refresh-usage.mjs` re-measures how many
tickets each work type has. It needs three values:

```bash
JIRA_BASE_URL=https://skillcatapp.atlassian.net \
JIRA_EMAIL=you@skillcatapp.com \
JIRA_API_TOKEN=... \
node scripts/refresh-usage.mjs && node build.mjs
```

Mint your own token at id.atlassian.com/manage-profile/security/api-tokens. You need
Jira read access to the CDLC project. In CI these come from repository secrets named
`JIRA_BASE_URL`, `JIRA_EMAIL` and `JIRA_API_TOKEN`. Until you add them, the
`refresh.yml` workflow fails.

**3. The published URL changed.** It used to be
`palsampurna16-prog.github.io/cdlc-workflow-record`. Once you host it yourself, update
any links in Confluence, Slack pins and onboarding docs, because that old URL only
redirects while Sampurna's GitHub account exists.

**4. `.github/workflows/requests.yml` runs on a daily cron** and writes back to the
repo. After the move it runs under the new owner's account. Check it actually succeeds
rather than finding out in a month.

## Hosting it

Whatever is easiest for your team. In rough order of effort:

- **Send the file.** `index.html` on its own works anywhere: a shared drive, a
  Confluence attachment, an email. No build, no repo. It goes stale unless someone
  re-exports it.
- **An internal web server.** Drop `index.html` on any nginx box or S3 bucket. This is
  the right option if you would rather it not be public.
- **GitHub Pages.** Push this to a repo, then Settings, Pages, deploy from `main`,
  root directory. This is how it was hosted before. Note that private repos need
  GitHub Team or Enterprise for Pages to work.
- **Netlify or Vercel.** Drag the folder onto app.netlify.com/drop for an instant URL,
  or connect the repo for deploy-on-push. Both offer password protection on paid
  plans, which GitHub Pages cannot do at all.

If this package includes a `.git` folder, the full commit history came with it and you
can push it straight to a new home:

```bash
git remote set-url origin https://github.com/YOUR-ORG/cdlc-workflow-record.git
git push -u origin main
```

## One thing to decide early

The old site was **public**. It shows SkillCat's Jira space structure, 69 work types,
ticket counts, workflow graphs, and links directly into `skillcatapp.atlassian.net`.
That is internal process detail on an indexable URL. If that was never a deliberate
choice, host it somewhere access-controlled instead.

## Work that was left unfinished

- **`CDLC-ownership-review.xlsx`** is a blank review template with two sheets, "Who
  fills each field" (53 rows) and "Who owns each work type" (70 rows). Sampurna was
  collecting answers from the team and did not finish. Rows shaded red are dormant:
  work types with no tickets in the last year. Ask Sampurna who had already replied
  before you re-send it.
- **SGP, CIP and QFT contents.** SGP and CIP have workflows and fields but not the
  full tree CDLC has. QFT is an honest empty state: one work type, no hierarchy, so it
  needs its own treatment rather than a copy of the others.
- **A history tab**, from `data/changes.json`, which already holds 40 change tickets
  covering Nov 2025 to Aug 2026.

## Two things the site does not claim

**Usage counts are a snapshot, not a live feed.** A static site cannot authenticate to
Jira, and embedding a token would expose it to every viewer. The numbers are baked in
at build time and refreshed by running the script above.

**The page is a lookalike, not a connection.** It makes no network calls and cannot
reach Jira. Nothing here writes to Jira. The only automated job writes back to this
repository.
