# Coalition Wiki: design

Date: 2026-09-05
Status: approved in discussion, pending written review

## Purpose

A Markdown wiki for the Coalition, the Ultima Online Outlands boating community, served at
`wiki.thecoalition.app`. It replaces the retired "Not a Pirate" Starlight wiki
(`f8169730.nap-wiki.pages.dev`) as the community's shared reference. Version 1 is a skeleton:
repo, build, domain, contributor workflow and page stubs. Content migration happens afterwards
by the group.

## Decisions

| Question | Decision |
|---|---|
| Editors | Maintainer plus a few trusted people, via GitHub pull requests |
| Relationship to UO Companion knowledge base | Separate repo and site; cross-link for general game info |
| Scope of v1 | Skeleton only: structure, stubs, workflow |
| Generator | MkDocs with the Material theme |
| Hosting | GitHub Pages, built by a workflow on GitHub-hosted runners. No self-hosted runner, no Cloudflare Pages |
| Domain | `wiki.thecoalition.app`, DNS on Cloudflare |
| Repo visibility | Public |

## Repository

Name: `UOCompanion/coalition-wiki`. Local checkout: `~/Development/UOCompanion/coalition-wiki`,
worked from the `dev` distrobox.

```
mkdocs.yml                     site config
requirements.txt               pinned Python deps
docs/                          all content
docs/.nav.yml                  sidebar order (mkdocs-awesome-nav)
docs/CNAME                     wiki.thecoalition.app
docs/index.md                  landing page
docs/about.md                  what the Coalition is (stub)
docs/contributing.md           how to edit and open a PR
docs/guides/farming.md
docs/guides/loot.md
docs/guides/sweeping.md
docs/guides/crew-and-upgrades.md
docs/guides/ocean-bosses.md
docs/guides/map-management.md
.github/workflows/pages.yml    build and deploy on push to main
.github/workflows/pr-check.yml strict build on pull requests
.github/CODEOWNERS             maintainer reviews everything
README.md                      purpose, local preview, contributing pointer
docs/superpowers/specs/        design docs (this file)
```

`mkdocs.yml` is a trimmed copy of the UO Companion config: Material theme, slate palette,
`navigation.tabs` off (single-level sidebar is enough), `navigation.top`, `search.suggest`,
`search.highlight`, admonition, details, superfences, tabbed, tables, attr_list, md_in_html,
emoji. Plugins: `search`, `awesome-nav`. `site_url: https://wiki.thecoalition.app/`.
`exclude_docs: superpowers/` in `mkdocs.yml` keeps design docs out of the build entirely.

`requirements.txt`: `mkdocs`, `mkdocs-material`, `mkdocs-awesome-nav`, pinned to exact
versions at creation time.

### Access and review

- Org team `wiki-editors` with write permission on the repo. Members branch and open PRs; they
  do not need org owner rights.
- Branch protection on `main`: pull request required, the `pr-check` status check required,
  linear history, no force pushes, no deletions. Maintainer may bypass for emergencies.
- `CODEOWNERS`: `* @rjk11111`. Review requests route to the maintainer automatically.
- Contributors need no local toolchain. GitHub's web editor plus a PR is the expected path;
  local preview is optional.

## Build and deploy

`pages.yml`, on push to `main` and manual dispatch:

1. `actions/checkout`
2. `actions/setup-python` 3.12 with pip cache keyed on `requirements.txt`
3. `pip install -r requirements.txt`
4. `mkdocs build --strict`
5. `actions/upload-pages-artifact` from `site/`
6. `actions/deploy-pages` in a separate `deploy` job with `pages: write` and `id-token: write`

Runs on `ubuntu-latest` only. Concurrency group `pages`, cancel in progress.

`pr-check.yml`, on pull requests to `main`: steps 1 through 4 only. A broken link, missing nav
entry, or bad YAML fails the check and blocks merge.

Repo Pages setting: build type "GitHub Actions". No `gh-pages` branch exists.

GitHub Pages has a single environment, so there are no per-PR preview URLs. Reviewers read the
rendered Markdown in the PR or run `mkdocs serve` locally.

## Domain

1. Repo Pages settings: custom domain `wiki.thecoalition.app`, Enforce HTTPS on. `docs/CNAME`
   keeps the setting from being lost on redeploy.
2. Cloudflare DNS: `CNAME wiki -> uocompanion.github.io`, DNS-only (grey cloud) until GitHub
   reports the certificate issued. Proxying afterwards is optional; if enabled, SSL mode Full.
3. Org settings: add `thecoalition.app` as a verified domain so no other GitHub account can bind
   its subdomains.

The old NAP site is left untouched.

## Skeleton content

Every page carries a one-paragraph purpose statement and section headings mirrored from the
corresponding old wiki page, so migrators have a map. No prose is copied in v1.

- `index.md`: what the Coalition is in two sentences, cards linking to About, Guides,
  Contributing, and to the UO Companion knowledge base for general game reference.
- `about.md`: mission and conduct, written for the whole coalition rather than one guild.
  Stub headings: Who we are, How we operate, Conduct, Member guilds, Contact.
- `guides/*.md`: headings taken from the old pages of the same name.
- `contributing.md`: fork or branch, edit on GitHub, open a PR, what the check enforces,
  style notes (one H1 per page, relative links, images under `docs/assets/`).

Sidebar (`docs/.nav.yml`): Home, About, Guides (folder, alphabetical), Contributing.

## Verification

Local: `mkdocs build --strict` and `mkdocs serve` from the dev box succeed.
CI: first push to `main` deploys; `curl -I https://wiki.thecoalition.app/` returns 200 over
HTTPS; search returns results; a PR containing a deliberately broken relative link fails
`pr-check` and cannot be merged.

Rollback: `git revert` on `main`; the workflow redeploys the previous site.

## Out of scope for v1

Content migration, per-PR previews, a CMS, analytics, custom theming beyond the palette,
private pages, and any change to the UO Companion site.
