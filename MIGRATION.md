# al-folio v1 migration

## Baselines

- Personal site snapshot: `3a7e46f9c234a2254f25a0f252082d62776dca54`.
- Official al-folio `main`: `929fbf71c5a45b13e9b5a20dfa425ce9e9de7935` (2026-10-26).
- Migration branch: `codex/al-folio-v1-upgrade`. Publication authorized by the site owner on 2026-10-28.
- Runtime: `al_folio_core 1.0.15`, with the official starter's pinned plugin versions and lockfile.

## Preserved site behavior

The homepage, research directions, internships, education, scholarship links and information icons, publication-page link, reviewer service and four teaching entries are retained. The 15 publications retain their order, authors, titles, venue text, abstracts, TL;DR text, coauthor URLs, arXiv/blog/WeChat links, and submission confidentiality. Name order remains Wang Jinbo 王锦波, and the initial theme remains light.

Configuration retains the `/me` project path, personal metadata, disabled search and blog feed, full author lists, and the existing OpenPanel identifier under the new `analytics.openpanel` key. Existing disabled demo navigation stays disabled. Original personalized images and data are retained.

## Runtime migration

The old copied `_plugins`, Sass, icon fonts, and general feature JavaScript are replaced by the official v1 gem runtime. Eight intentional overrides start from the current core gem, with only the necessary site differences:

- `_layouts/about.liquid`: name, education, publication link, service and teaching.
- `_layouts/bib.liquid`: title-brace cleanup, wider content column, Markdown abstracts, TL;DR and WeChat buttons.
- `_includes/header.liquid`, `_includes/footer.liquid`, `_includes/metadata.liquid`: name order and Chinese name.
- `_includes/head.liquid`: load the small site stylesheet.
- `assets/js/common.js`: mutually exclusive TL;DR/abstract panels and keyboard activation.
- `assets/js/theme.js`: default to light mode.

`_includes/education.liquid` and `assets/css/jinbo.css` are site-owned additions. `.al-folio-overrides.yml` records the reviewed upstream checksums so later plugin updates can identify drift.

The new tooltip implementation renders plain text safely. The two existing HTML-formatted biography/Scholar hints therefore keep their wording with plain text and a newline instead of embedded markup. Scholarship explanations work with focus and hover. Bootstrap compatibility remains disabled; the native v1 interactions are used.

## Build environment

The Docker base and Ruby workflows use the 3.3 series. The validated local image uses Ruby 3.3.13. This also fixes the unavailable 3.3.5 patch on GitHub's newer Ubuntu runners. The container entry point preserves `Gemfile.lock` even when linked-worktree Git metadata is outside the mount. Preview mode sets `BUNDLE_FROZEN=true`.

The original site directory and its port 8080 preview are retained. Start this isolated preview from this worktree:

```sh
docker compose -p jinbo-v1 -f docker-compose.preview.yml up -d --build
```

Open <http://localhost:8081/me/>. Stop only this preview with:

```sh
docker compose -p jinbo-v1 -f docker-compose.preview.yml down
```

## Validation

Run the following from this worktree (the preview service must be running):

```sh
npm ci
npm run lint:prettier
docker compose -p jinbo-v1 -f docker-compose.preview.yml exec -T jekyll sh -c 'JEKYLL_ENV=production bundle exec jekyll build --destination /tmp/jinbo-v1-production'
docker compose -p jinbo-v1 -f docker-compose.preview.yml exec -T jekyll bash test/integration_css_minify.sh
docker compose -p jinbo-v1 -f docker-compose.preview.yml exec -T jekyll bash test/integration_upgrade_cli.sh
docker compose -p jinbo-v1 -f docker-compose.preview.yml exec -T jekyll bundle exec al-folio upgrade audit
docker compose -p jinbo-v1 -f docker-compose.preview.yml exec -T jekyll bundle exec al-folio upgrade overrides audit
```

A rendered HTML comparison against the personal-site snapshot passed for all 15 publication entries (order, text and links), homepage text/links, and referenced local assets. Browser checks cover desktop and narrow layouts, navigation, light/dark themes, scholarship hints and publication panels.

Upgrade audit: zero blocking findings and all eight overrides acknowledged. Nine advisory findings remain: three `data-toggle="popover"` markers still handled by the native core UI, four table markers in an upstream demo post, and two false-positive jQuery detections in demo math expressions. These are retained, reviewed compatibility markers rather than a requirement to load legacy jQuery. The starter-only style-contract test intentionally rejects local overrides and is not the appropriate check for this personalized site; upstream already guards that CI job to its own repository.

No intentional content or functional differences remain. Typography and some button casing follow the new core. The owner reviewed the local preview and authorized publication on 2026-10-28.

## Publication-year clarification after review

Fast Catch-Up and GradPower retain `year = {2025}` for grouping. Their separate `conference_year = {2026}` drives the venue line, so ICLR and ICML correctly display 2026. A note on the publications page explains the distinction; the homepage keeps only the publication-page link. Token-Adaptive Mixing now uses the `NeurIPS` badge; the separate `NeurIPS Spotlight` badge is preserved. No 2027 claim was added. Production HTML checks verified the 2025 groups and 2026 venue years.

All conference badges now omit years (including ICLR, ICML, NeurIPS and ICONIP). The publications-page note explains the general first-public-release grouping rule rather than listing exceptions; venue years remain explicit. All 15 rendered badges were checked for leftover years.

The final biography uses “My experience spans research and internships at …” to cover both current and past roles, and “I expect to graduate in 2027 and am currently on the job market.”

The research introduction reads “My current research focuses on two directions:”.

Ziniu Li is linked to https://www.liziniu.org/, whose homepage lists the Larger Batches collaboration. The publication branch `codex/al-folio-v1-release` records the reviewed v1 tree on top of the existing personal-site main history; upstream source history remains available locally on the migration branch.
