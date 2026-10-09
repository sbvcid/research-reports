# AGENTS.md — Report Publication SOP

Operating rules for publishing a research report to this site. Any agent (or
human) asked to "publish this report" must follow this file end to end. If this
file conflicts with `README.md`, `AGENTS.md` is authoritative and `README.md`
should be corrected as part of the same change.

Scope: this repository is a **publication-only** Jekyll site. It stores rendered
research articles and nothing else. It has no build tooling, no npm, no
dependencies beyond Jekyll, and no tests. Everything here is about moving a
finished article from "a file someone has" to "a live URL", without changing the
article.

---

## 0. Hard rules

These are not negotiable. Violating any of them means stopping and reporting,
not improvising.

1. **The article body is immutable.** Only YAML front matter may be added. Do
   not fix typos, reformat tables, renumber sections, translate, shorten,
   "clean up" wording, or resolve an argument the author left open. The site's
   value proposition is that published reports are reproduced in full, including
   their errors and unresolved questions.
2. **Nothing private ships.** No API keys, tokens, passwords, internal URLs,
   customer or counterparty identifiers, unpublished drafts, raw personal data,
   private research repository paths, or proprietary data files. If the source
   document contains any of these, stop and report — do not redact silently.
   Redaction is an edit, and edits are the author's decision, not the
   publisher's.
3. **One change set per publication.** Stage only the new report file. Any other
   modification in the working tree belongs to someone else and must not be
   swept into the commit.
4. **Do not invent content.** If the source is missing, truncated, or ambiguous,
   stop and report (section 9).
5. **No automation infrastructure.** No watchdogs, no scheduled posts, no
   auto-publish scripts. Publication is a manual, supervised action.

---

## 1. Read before you touch anything

Read these, in this order, every time. They are small; there is no reason to
publish from memory.

| # | File | What you learn from it |
|---|---|---|
| 1 | `AGENTS.md` | This procedure. |
| 2 | `README.md` | Site purpose and the short version of the flow. |
| 3 | `_config.yml` | `site.url`, `site.baseurl`, `exclude`, kramdown settings. If `url`/`baseurl` differ from what you assume, every URL you report is wrong. |
| 4 | `_layouts/default.html` | Which front matter keys the shell actually renders: `title`, `description`, `subtitle`, `date`, `report_id`, `show_title`. Anything else you add is inert. |
| 5 | `index.md` | How the index enumerates reports. It filters on `published: true` and sorts by `date` — no manual index edit is needed. |
| 6 | `feed.xml` | Same filtering, plus that the feed embeds full article HTML. This is why a half-written article would leak into RSS. |
| 7 | One existing report, e.g. `reports/8306-mufg.md` | The house conventions for front matter, heading style, and how a body is formatted. |

Confirm the ground truth before proceeding:

```
git status --porcelain=v1 --branch
git log --oneline -5
git remote -v
```

Expected: on `main`, tracking `origin/main`, remote `sbvcid/research-reports`,
clean tree. Investigate any deviation before continuing.

---

## 2. Locate and confirm the source article

The user names an article, not a file path. Resolve it in this order and stop
at the first hit:

1. If the user gave a path, use it. Confirm it exists and is readable.
2. If the user named the report by `report_id`, `title`, or slug, search the
   repository and the working tree:
   - `reports/` for an already-published copy (if found, this is a re-publish
     or a duplicate — report it, do not overwrite).
   - the working tree and any sibling/adjacent directory the user referenced,
     for the source file.
3. If nothing matches, ask the user for the path. Do not guess among several
   similar files and do not pick the newest by timestamp.

Confirm before writing:

- The file is complete. Check the tail. A truncated file (ends mid-sentence,
  mid-table, mid-code-block) is a source problem, not something to guess at.
- It is the right edition. Translated or bilingual editions exist in this
  repository (`report_id` suffix `-zh` marks the sealed original, no suffix
  marks the English edition). Publishing the wrong edition is unrecoverable in
  public.
- The content is appropriate for public release under rule 0.2.
- Its encoding is UTF-8. Check for a BOM; a BOM becomes visible garbage in the
  page title.

Record the source path and its exact byte length in your report. Byte length is
how the next agent can tell whether anything was lost in transit.

---

## 3. Choose the slug and write front matter

**Slug** = the filename stem in `reports/<slug>.md`. It must also be the
`report_id` and the last path segment of the `permalink`. One value, three
places, so the URL, the file on disk, and the site metadata can never disagree.

Slug rules:

- Lowercase ASCII letters, digits, and single hyphens. No spaces, no
  underscores, no non-ASCII characters.
- Stable and descriptive of the subject, not of the publication moment. Avoid
  dates, versions, and language markers (`-v2`, `-final`, `-new`) — a
  corrected article is a new report or an edit, not a slug suffix.
- 2–6 words. Shorter than 2 suggests you are naming a chapter, not a report.
- Check `reports/` first. If the slug is taken, it is a collision: stop and
  report (section 9).

Front matter, exactly these keys in this order:

```yaml
---
layout: default
title: "<Report title>"
subtitle: "<Optional one-line qualifier>"
description: "<One or two sentences: what the report covers and what a reader gets.>"
report_id: <slug>
date: <YYYY-MM-DD>
published: true
permalink: /reports/<slug>/
---
```

Per key:

- `layout: default` — required. `feed.xml` uses `layout: null`; a report must not.
- `title` — quoted. Must not contain unescaped `"`. It becomes both the `<h1>`
  and the `<title>` element.
- `subtitle` — optional, quoted. Rendered under the title. Omit the key rather
  than leaving it empty.
- `description` — quoted. It is the `<meta name="description">`, the index
  blurb, and the RSS `<description>`. Write it for a reader who has not opened
  the article. Keep it under roughly 320 characters.
- `report_id: <slug>` — must equal the filename stem, unquoted.
- `date` — unquoted ISO date. This is the sort key for both the index and the
  feed. If it is in the future, the report is listed last; use the real
  publication date, not a placeholder.
- `published: true` — the switch that admits the article to the index and the
  feed. Do not set it to `false` on a file in `reports/`; that publishes a file
  nobody links and confuses the next agent.
- `permalink: /reports/<slug>/` — trailing slash, matching every existing
  report. Omitting it lets Jekyll derive the same value, but write it explicitly
  so a reader of the file can see the URL.

One article = one file. If an article needs both a Chinese and an English
edition, that is two reports with distinct slugs and distinct dates.

---

## 4. Body assembly — preserve everything

Append the source body **verbatim** below the front matter. Do not retype,
re-paste through a formatter, or re-save the source through an editor that may
normalise line endings or trailing whitespace.

What must survive unchanged:

- **Tables.** GFM pipe tables, alignment rows, inline `**bold**` inside cells,
  `｜` and other full-width separators used in CJK documents, and rows whose
  cells contain `\|`. Do not re-align columns to look tidy.
- **Blockquotes and callouts.** `>` prefixes, nested quotes, and admonition-style
  blocks. Do not convert a quote into a paragraph.
- **Code blocks and formulas.** Fenced blocks with their language tag, LaTeX and
  Unicode math, subscript/superscript notation, arrows, Greek letters. Close
  every fence. An unclosed fence swallows the rest of the article.
- **Footnotes** `[^1]` and their definitions, plus reference lists with URLs.
  Footnote definitions must not be moved above the text that references them.
- **Special characters.** Full-width punctuation, CJK brackets, em dashes,
  non-breaking spaces, `%`, `#`, `<`, `>`, `&`, currency symbols. Do not
  normalise them to ASCII; in CJK text that is a content change.
- **Internal cross-references.** "Chapter IV", "see §3.2", "Appendix A". If the
  target is missing in the source, that is an unresolved question in the
  author's document, not a defect for you to fix.
- **HTML blocks.** The existing reports embed raw `<div>` for callouts. Raw HTML
  in a Markdown body passes through kramdown untouched. Keep it balanced.

After assembly, prove the body is unchanged rather than assuming it:

```
# byte-level check: strip the front matter, compare the remainder
python -c "import sys,io;p=sys.argv[1];d=io.open(p,encoding='utf-8').read();b=d.index('---',3)+4;open(sys.argv[2],'w',encoding='utf-8',newline='').write(d[b:])" reports/<slug>.md %TEMP%\body.md
fc /b %TEMP%\body.md "<source path>"
```

`fc /b` must report no differences. Any difference means the body was altered;
fix it before continuing. Keep the line-ending convention of the source — if the
repository normalises to LF for Markdown, confirm with `git diff --stat` that
you have not rewritten every line of the file.

If the source itself needs correction, stop and report (section 9).

---

## 5. Validate before committing

Run these checks and record the results. Do not skip to the commit because the
article "looks fine".

**a. Front matter parses and has the right keys.** `python` with PyYAML is
enough; no Jekyll required:

```
python -c "import io,sys,yaml;p=sys.argv[1];d=io.open(p,encoding='utf-8').read();fm=d.split('---',2)[1];m=yaml.safe_load(fm);req=['layout','title','description','report_id','date','published','permalink'];miss=[k for k in req if k not in m];print('MISSING',miss) if miss else print('OK',m['report_id'],m['permalink'])" reports/<slug>.md
```

Also confirm `report_id` equals the filename stem and `permalink` equals
`/reports/<slug>/`.

**b. The body is byte-identical.** The `fc /b` check in section 4. Non-negotiable.

**c. No private material.** Read the diff yourself, in full. Search for the
classes of thing in rule 0.2: `api[_-]?key`, `token`, `secret`, `password`,
`bearer`, internal hostnames, absolute paths from other machines, and any
mention of a private repository path. A grep is a screen, not a substitute for
reading.

**d. Links.** Extract every URL and confirm:
- External links resolve (`https://` only; no bare `http://` leftovers unless
  the source is genuinely http-only).
- Relative links point at paths that exist in the repository.
- Links inside footnote definitions and reference lists are present too — they
  are easy to miss.

**e. Attachments.** If the article references images, PDFs, or data files,
confirm each exists in the repository and is committed, and that it is either
under `assets/` or otherwise inside the published tree. A file only on the
author's disk renders as a broken image. Never attach a file that carries
private data, even to illustrate a point.

**f. Site compatibility.** Confirm the front matter keys match
`_layouts/default.html`, the file is in `reports/`, and `published: true` is
set. Confirm you did **not** need to touch `_config.yml`, `index.md`, or
`feed.xml` — those enumerate `published: true` pages automatically, and editing
them by hand is a bug, not a step.

**g. Site build.** If Ruby/Jekyll is available locally:

```
jekyll build --trace
```

Then confirm the article is listed on `/` and present in `/feed.xml`. If Jekyll
is not available locally (as in the current environment), say so in your report
and rely on checks (a)–(f) plus post-push deployment verification. Do not
install a toolchain or add a `Gemfile` to satisfy this step.

---

## 6. Inspect git state before staging

This repository is shared. Other work is often sitting in the tree.

```
git status --porcelain=v1
git diff
```

Rules:

- **Stage by explicit path.** `git add reports/<slug>.md` — never `git add .`,
  `git add -A`, `git add reports/`, or `git commit -a`.
- Untracked or modified files you did not create are **other people's work**.
  Leave them exactly as they are and mention them in your report.
- An untracked file already sitting in `reports/` is either an unfinished
  article or someone else's in-progress publication. Do not commit it,
  delete it, or `git clean` it.
- If your own file conflicts with an untracked file at the same path, you cannot
  stage cleanly — stop and report.
- Confirm the branch: `git rev-parse --abbrev-ref HEAD` must be `main`. This
  site deploys from `main`. Never publish from a feature branch or a worktree
  that is not `main`.
- Confirm the remote: `git remote -v` must show `sbvcid/research-reports`. Push
  to `main`, never to a fork or a differently named remote.
- Pull first if `origin/main` is ahead: `git pull --ff-only origin main`. Never
  force-push. Never amend or rebase shared history.

---

## 7. Commit, push, verify deployment

Commit, staging only the new file:

```
git add reports/<slug>.md
git status --porcelain=v1     # must show only the staged article as A
git commit -m "publication: <report title or subject>" -m "<2-6 lines: source of the text, what was verified, which edition is authoritative, and the URL this produces>"
git rev-parse HEAD
```

Match the repository's existing commit style: `publication:`, `docs:`,
`feat:`, `chore:`. Publication commits have carried a body explaining provenance
and verification — follow that.

Push:

```
git push origin main
```

Never push more than the commit you made.

Deployment verification, in order of preference:

1. `gh run list --repo sbvcid/research-reports --limit 5` and confirm the
   `pages-build-deployment` workflow succeeded for the pushed commit.
2. `gh api repos/sbvcid/research-reports/pages` and read `status` / `html_url`.
3. Fetch the live URLs and confirm the article appears:
   - `https://sbvcid.github.io/research-reports/`
   - `https://sbvcid.github.io/research-reports/reports/<slug>/`
   - `https://sbvcid.github.io/research-reports/feed.xml`

GitHub Pages builds are asynchronous. If the workflow is still queued or
running, say the push succeeded and deployment is pending. If `gh` is not
authenticated, or the network blocks it, say verification of deployment was not
possible and give the URLs for the user to check — do not claim a deployment
result you did not observe.

If deployment fails, fetch the build log, report the error verbatim, and stop.
Do not weaken the article to make a build pass.

---

## 8. Report format

Every publication ends with a report containing all of:

- **Article title and slug.**
- **Source**: the file it was published from, and its byte length.
- **Files added / modified**, with paths.
- **Validation results**: front matter parse, byte-identical body check
  (`fc /b`), links, attachments, site compatibility, Jekyll build or the reason
  it was skipped, secret scan.
- **Commit SHA** (full hash) and the branch pushed.
- **Push result**: remote branch and confirmation that local and remote agree.
- **Deployment result**: workflow status or the explicit statement that it could
  not be verified.
- **Article URL**, and the index + feed URLs.
- **Anything left undone**: unrelated uncommitted files in the tree, follow-up
  work, unresolved source defects.

State failures plainly. A partially verified publication reported honestly is
useful; one reported as successful when it was not is worse than no report.

---

## 9. When to stop and report

Stop, do not improvise, and ask the user in any of these cases:

**Ambiguity about the article**
- The source is not found, or more than one file plausibly matches the request.
- The edition (original vs translation, sealed vs draft) is unclear.
- Two files differ and there is no way to tell which is authoritative.
- The file appears truncated, or ends mid-table, mid-sentence, or inside a
  fenced code block.

**The body would need to change**
- A typo, broken table, wrong cross-reference, or mistranslation is found in
  the source. Report the location and the issue; do not fix it.
- Footnote markers are unmatched, or a code fence or HTML block is unbalanced.
- Footnote order, section numbering, or table column order looks wrong.
- The article states conclusions that are stale or contradicted by later
  material — report it, publish it as written.
- Data in the article is sensitive, embargoed, or market-moving in a way that
  publishing creates risk.

**Path or naming conflicts**
- The intended slug already exists in `reports/`.
- An untracked file already occupies the target path.
- `report_id` or `permalink` would not match the filename stem.
- The same content already exists under a different slug.

**Validation failures**
- The `fc /b` body check shows any difference. This is the hard stop.
- Front matter fails to parse, or a required key is missing.
- A secret, private path, or restricted data appears in the body.
- A referenced link or attachment is missing.
- The Jekyll build fails, or the article is absent from the index or feed
  after a successful build.

**Repository state**
- The working tree has unrelated modifications and you cannot be certain which
  files belong to this publication.
- The current branch is not `main`, or the remote is not
  `sbvcid/research-reports`.
- `origin/main` has diverged and cannot be fast-forwarded.
- The push is rejected, or the deployment fails.

In every case: report what you found, with file paths, line numbers, and the
exact error text. Then state what you would need in order to proceed. A stopped
publication is a correct outcome; a guessed one is not.

---

## 10. Worked reference: what a correct publication looks like

```
1. Read AGENTS.md, README.md, _config.yml, _layouts/default.html, index.md,
   feed.xml, and one existing report.
2. git status --porcelain=v1 --branch        -> main, clean
3. Resolve the source file the user named. Note path + byte length.
4. Derive slug from the subject. Confirm reports/<slug>.md does not exist.
5. Write YAML front matter with the seven required keys, then append the body
   verbatim. No retyping, no reformatting.
6. fc /b body check against the source        -> identical
7. PyYAML front matter check                  -> OK, keys present
8. Read the full diff. Secret scan. Link check. Attachment check.
9. git add reports/<slug>.md                  -> only that file staged
10. Commit "publication: ..." with a provenance body. Capture the SHA.
11. git push origin main
12. Confirm the Pages workflow, then confirm the live index, article URL and
    feed.
13. Report: article URL, commit SHA, push status, deployment status, and any
    unrelated files left untouched in the working tree.
```
