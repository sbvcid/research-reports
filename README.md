# research-reports

Publication site for research reports. GitHub Pages, Jekyll, kramdown. No
framework, no npm, no dependencies beyond Jekyll itself.

**This repository contains published reports only.** The private research
repository is separate and is never made public.

## Layout

```
_config.yml            site config, kramdown + GFM input
_layouts/default.html  page shell
assets/style.css       minimal stylesheet
index.md               report index
feed.xml               RSS 2.0, generated from published reports
reports/<id>.md        one file per report
```

## Publishing a new report

**Before publishing anything, read [`AGENTS.md`](AGENTS.md).** It is the
authoritative publication SOP: what to read, how to locate and confirm the
source article, slug and front-matter rules, how to prove the body is preserved
byte-for-byte, the validation checklist, git staging discipline, push and
deployment verification, and the cases where an agent must stop and report
instead of guessing. `AGENTS.md` wins if the two ever disagree.

The short version:

1. Add `reports/<id>.md` with front matter containing
   `layout: default`, `title`, `description`, `report_id`, `date`,
   `published: true`, `permalink: /reports/<id>/`.
2. Adding the file to a list in `_config.yml` is **not** needed —
   `index.md` and `feed.xml` both enumerate `site.pages | where: "published", true`.
   The index updates itself.
3. Stage only the new report file, commit, and push to `main`. GitHub Pages
   rebuilds automatically.

`AGENTS.md` and this file are excluded in `_config.yml`, so neither is published
as a page.

## URLs

| Page | URL |
|---|---|
| Index | `https://sbvcid.github.io/research-reports/` |
| Report | `https://sbvcid.github.io/research-reports/reports/8306-mufg/` |
| RSS | `https://sbvcid.github.io/research-reports/feed.xml` |

## Notes

- **Report bodies are byte-identical to their source.** Only YAML front matter
  is added on top. No research content, figures, tables, quotes or conclusions
  are altered at publication time.
- **`/feed.xml` is hand-written, not jekyll-feed.** jekyll-feed builds from
  `_posts`; this site has none by design, so its feed would be structurally
  empty. The hand-written template emits only pages marked
  `published: true`, so the feed cannot contain anything unpublished.
- If `site.url` / `site.baseurl` change (repo rename or a custom domain), update
  both `_config.yml` and the front-matter `baseurl` fallback.