# Coalition Wiki Skeleton Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up `wiki.thecoalition.app` as a MkDocs Material wiki in `UOCompanion/coalition-wiki`, built and deployed by GitHub Pages, with stub pages and a PR-gated contributor workflow.

**Architecture:** Plain Markdown under `docs/` rendered by MkDocs Material; sidebar driven by `docs/.nav.yml` (mkdocs-awesome-nav). A GitHub Actions workflow on GitHub-hosted runners builds with `mkdocs build --strict` and deploys through `actions/deploy-pages`; a second workflow runs the same strict build on pull requests and is a required status check on `main`.

**Tech Stack:** MkDocs 1.6.1, mkdocs-material 9.7.7, mkdocs-awesome-nav 3.3.0, Python 3.12 in CI, GitHub Pages (workflow build type), GitHub CLI `gh`, Cloudflare DNS.

**Spec:** `docs/superpowers/specs/2026-09-05-coalition-wiki-design.md`

## Global Constraints

- Repo: `UOCompanion/coalition-wiki`, public, default branch `main`.
- Local checkout: `~/Development/UOCompanion/coalition-wiki`. Every build/install command runs inside the `dev` distrobox, never on the Bazzite host. Prefix one-off commands with `distrobox enter dev -- bash -lc '<cmd>'`, or open a shell with `distrobox enter dev`.
- Workflows run on `ubuntu-latest` only. No self-hosted runner. No Cloudflare Pages.
- Pinned deps: `mkdocs==1.6.1`, `mkdocs-material==9.7.7`, `mkdocs-awesome-nav==3.3.0`.
- Actions: `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-pages-artifact@v5`, `actions/deploy-pages@v5`.
- `mkdocs build --strict` must pass locally and in CI at the end of every task.
- Domain: `wiki.thecoalition.app`, CNAME to `uocompanion.github.io`, DNS on Cloudflare.
- Design docs live under `docs/superpowers/` and are excluded from the build with `exclude_docs`.
- Commit messages end with:
  ```
  Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01NDrjp16X4541M7nobsX27S
  ```

## File structure

| Path | Responsibility |
|---|---|
| `mkdocs.yml` | Site config: theme, extensions, plugins, `exclude_docs` |
| `requirements.txt` | Pinned Python deps for local and CI builds |
| `.gitignore` | `site/`, `.venv/`, caches |
| `README.md` | What the repo is, how to preview, pointer to contributing |
| `docs/.nav.yml` | Top-level sidebar order |
| `docs/index.md` | Landing page |
| `docs/about.md` | Coalition mission stub |
| `docs/contributing.md` | Editing and PR workflow |
| `docs/guides/.nav.yml` | Guides section title and sort |
| `docs/guides/*.md` | Six guide stubs with headings mirrored from the old wiki |
| `docs/assets/.gitkeep` | Home for future images |
| `docs/CNAME` | Custom domain for Pages |
| `.github/workflows/pages.yml` | Build and deploy on push to `main` |
| `.github/workflows/pr-check.yml` | Strict build on pull requests (required check) |
| `.github/CODEOWNERS` | Route reviews to the maintainer |

---

### Task 1: MkDocs project scaffold that builds strictly

**Files:**
- Create: `mkdocs.yml`, `requirements.txt`, `.gitignore`, `README.md`, `docs/.nav.yml`, `docs/index.md`
- Existing: `docs/superpowers/specs/2026-09-05-coalition-wiki-design.md` (must be excluded from the build)

**Interfaces:**
- Produces: a working `mkdocs build --strict` and a `.venv` in the repo root used by every later task. Nav file format `docs/.nav.yml` consumed by Task 2.

- [ ] **Step 1: Create the virtualenv and confirm the build fails before config exists**

Run (in the dev box):
```bash
cd ~/Development/UOCompanion/coalition-wiki
python3 -m venv .venv && . .venv/bin/activate
pip install mkdocs==1.6.1 mkdocs-material==9.7.7 mkdocs-awesome-nav==3.3.0
mkdocs build --strict
```
Expected: FAIL with `Config file 'mkdocs.yml' does not exist.`

- [ ] **Step 2: Write `requirements.txt`**

```
mkdocs==1.6.1
mkdocs-material==9.7.7
mkdocs-awesome-nav==3.3.0
```

- [ ] **Step 3: Write `mkdocs.yml`**

```yaml
site_name: The Coalition Wiki
site_description: Shared reference for the Coalition, the Ultima Online Outlands boating community
site_url: https://wiki.thecoalition.app/
repo_url: https://github.com/UOCompanion/coalition-wiki
repo_name: UOCompanion/coalition-wiki
edit_uri: edit/main/docs/

# Design docs and plans live under docs/superpowers and are never rendered.
exclude_docs: |
  superpowers/

theme:
  name: material
  palette:
    scheme: slate
  features:
    - navigation.top
    - navigation.indexes
    - search.suggest
    - search.highlight
    - content.action.edit
    - content.tabs.link

markdown_extensions:
  - tables
  - attr_list
  - md_in_html
  - admonition
  - toc:
      permalink: true
  - pymdownx.details
  - pymdownx.superfences
  - pymdownx.tabbed:
      alternate_style: true

plugins:
  - search
  - awesome-nav
```

- [ ] **Step 4: Write `docs/.nav.yml` and `docs/index.md`**

`docs/.nav.yml`:
```yaml
nav:
  - Home: index.md
```

`docs/index.md`:
```markdown
# The Coalition Wiki

The Coalition is the Ultima Online Outlands boating community: guilds and crews who farm,
sweep, and hunt ocean bosses together. This wiki is our shared reference.

It is a skeleton right now. Pages carry their intended headings so contributors know where
content belongs. See [Contributing](contributing.md) to help fill it in.

For general Outlands game knowledge (skills, items, monsters) use the
[UO Companion knowledge base](https://uocompanion.pages.dev/).
```

Note: `contributing.md` is created in Task 2. Until then the strict build fails on this link, which is the failing test for Task 2. For this task's green step, temporarily create a one-line `docs/contributing.md` containing `# Contributing` so the build passes; Task 2 replaces it.

- [ ] **Step 5: Write `.gitignore` and `README.md`**

`.gitignore`:
```
site/
.venv/
__pycache__/
.cache/
```

`README.md`:
```markdown
# coalition-wiki

Source for <https://wiki.thecoalition.app>, the Coalition's wiki for Ultima Online Outlands
boating. Plain Markdown under `docs/`, rendered by MkDocs Material and published by GitHub Pages.

## Edit

Open a pull request. See `docs/contributing.md` (rendered at
<https://wiki.thecoalition.app/contributing/>). No local tooling is required.

## Preview locally

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Then open <http://127.0.0.1:8000>. `mkdocs build --strict` is the same check CI runs.
```

- [ ] **Step 6: Run the strict build and confirm it passes and excludes design docs**

Run (dev box, venv active):
```bash
mkdocs build --strict && ls site/ && test ! -e site/superpowers && echo "superpowers excluded"
```
Expected: `INFO - Documentation built in ...`, `site/index.html` present, `superpowers excluded` printed, no warnings.

- [ ] **Step 7: Commit**

```bash
git add mkdocs.yml requirements.txt .gitignore README.md docs/.nav.yml docs/index.md docs/contributing.md
git commit -m "feat: mkdocs material scaffold with strict build"
```

---

### Task 2: Skeleton pages and navigation

**Files:**
- Create: `docs/about.md`, `docs/guides/.nav.yml`, `docs/guides/farming.md`, `docs/guides/loot.md`, `docs/guides/sweeping.md`, `docs/guides/crew-and-upgrades.md`, `docs/guides/ocean-bosses.md`, `docs/guides/map-management.md`, `docs/assets/.gitkeep`, `docs/CNAME`
- Modify: `docs/.nav.yml`, `docs/contributing.md` (replace the placeholder)

**Interfaces:**
- Consumes: the `.nav.yml` format and venv from Task 1.
- Produces: the page set the deploy in Task 3 publishes.

- [ ] **Step 1: Prove the strict build catches a broken link (failing test)**

Run (dev box, venv active):
```bash
printf '# Sweeping\n\nSee [loot](loot.md).\n' > docs/guides/sweeping.md
mkdocs build --strict
```
Expected: FAIL with `Doc file 'guides/sweeping.md' contains a link 'loot.md', but the target is not found among documentation files.` and `Aborted with 1 warnings in strict mode`.

- [ ] **Step 2: Write `docs/.nav.yml` and `docs/guides/.nav.yml`**

`docs/.nav.yml`:
```yaml
nav:
  - Home: index.md
  - About: about.md
  - Guides: guides
  - Contributing: contributing.md
```

`docs/guides/.nav.yml`:
```yaml
title: Guides
sort:
  type: natural
```

- [ ] **Step 3: Write `docs/about.md`**

```markdown
# About the Coalition

The Coalition is the Ultima Online Outlands boating community. This page describes who we
are and how we operate. It is a stub: headings are in place, content is being migrated.

## Who we are

## How we operate

## Conduct

## Member guilds

## Contact
```

- [ ] **Step 4: Write the six guide stubs**

`docs/guides/farming.md`:
```markdown
# Farming

How Coalition crews farm the ocean, solo and in groups. Stub: headings mirror the previous
wiki; content is being migrated.

## Solo farming

## Group farming
```

`docs/guides/loot.md`:
```markdown
# Loot

How loot is split on Coalition boats. Stub: headings mirror the previous wiki; content is
being migrated.

## Loot distribution

## High end crew

## Links
```

`docs/guides/sweeping.md`:
```markdown
# Sweeping

Sweeping the ocean for targets. Stub: content is being migrated.

## Overview
```

`docs/guides/crew-and-upgrades.md`:
```markdown
# Crew and Upgrades

Which crew and ship upgrades matter, and what they are worth. Stub: headings mirror the
previous wiki; content is being migrated.

## Most valuable crew types

## Most valuable stats

### Lottery crew

### Valuable crew

### Junk crew

## Average crew and upgrade value database

### Crew pricing

### Upgrades pricing
```

`docs/guides/ocean-bosses.md`:
```markdown
# Ocean Bosses

How to run ocean bosses as a fleet. Stub: headings mirror the previous wiki; content is
being migrated.

## How to boss

### Build

### Mastery chain setup

### Aspect

## Mini boss tactics

### Strafe

### Adds

### Looting

### Distributing loot

## Main boss tactics
```

`docs/guides/map-management.md`:
```markdown
# Map Management

Using the shared map and sharing locations. Stub: headings mirror the previous wiki;
content is being migrated.

## The map

## Using the map

## Implementing in game

### Ocean grid

### Files

## Location sharing
```

- [ ] **Step 5: Write `docs/contributing.md`, `docs/CNAME`, `docs/assets/.gitkeep`**

`docs/contributing.md`:
```markdown
# Contributing

Anyone in the wiki-editors team can change this wiki. Changes go through a pull request and
are published automatically once merged.

## Edit a page on GitHub

1. Open the page on the wiki and click the pencil icon, or browse to the file under `docs/`
   in the [repository](https://github.com/UOCompanion/coalition-wiki).
2. Edit the Markdown and choose "Create a new branch and start a pull request".
3. Wait for the "strict-build" check. It fails on broken links, missing nav entries, or bad
   YAML. Fix and push again.
4. A maintainer reviews and merges. The site updates within a couple of minutes.

## Add a page

Create `docs/guides/<name>.md` with a single `#` heading at the top. It appears in the Guides
section automatically. To place a page elsewhere, add it to `docs/.nav.yml`.

## Style

- One `#` heading per page, then `##` and `###` for sections.
- Link to other pages with relative paths: `[Loot](loot.md)`, `[About](../about.md)`.
- Put images under `docs/assets/` and reference them as `../assets/<file>`.
- Prefer short sections with tables or lists over long paragraphs.
- Use admonitions for warnings and tips:

    !!! tip
        Bring extra cannon shot for mini bosses.

## Preview locally (optional)

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```
```

`docs/CNAME`:
```
wiki.thecoalition.app
```

`docs/assets/.gitkeep`: empty file.

- [ ] **Step 6: Run the strict build and check the nav**

Run (dev box, venv active):
```bash
mkdocs build --strict && grep -o 'guides/[a-z-]*/' site/index.html | sort -u
```
Expected: build succeeds with no warnings; the grep lists all six guide slugs (`crew-and-upgrades`, `farming`, `loot`, `map-management`, `ocean-bosses`, `sweeping`). `site/CNAME` exists.

- [ ] **Step 7: Commit**

```bash
git add docs
git commit -m "feat: skeleton pages, nav, CNAME"
```

---

### Task 3: Workflows, CODEOWNERS, GitHub repo, first deploy

**Files:**
- Create: `.github/workflows/pages.yml`, `.github/workflows/pr-check.yml`, `.github/CODEOWNERS`

**Interfaces:**
- Consumes: the repo contents from Tasks 1 and 2.
- Produces: GitHub repo `UOCompanion/coalition-wiki` with Pages enabled (workflow build type) and a green deploy at `https://uocompanion.github.io/coalition-wiki/` (or the custom domain once Task 4 completes). The `strict-build` job name is the required check Task 5 references.

- [ ] **Step 1: Write `.github/workflows/pages.yml`**

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-python@v7
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements.txt
      - run: mkdocs build --strict
      - uses: actions/upload-pages-artifact@v5
        with:
          path: site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v5
```

- [ ] **Step 2: Write `.github/workflows/pr-check.yml`**

```yaml
name: PR check

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  strict-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-python@v7
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -r requirements.txt
      - run: mkdocs build --strict
```

- [ ] **Step 3: Write `.github/CODEOWNERS`**

```
* @rjk11111
```

- [ ] **Step 4: Validate the YAML parses (failing-test equivalent for config files)**

Run (dev box, venv active):
```bash
python3 -c "import yaml,sys; [yaml.safe_load(open(f)) for f in sys.argv[1:]]; print('yaml ok')" .github/workflows/pages.yml .github/workflows/pr-check.yml
```
Expected: `yaml ok`. (PyYAML is a mkdocs dependency, so it is in the venv.)

- [ ] **Step 5: Commit**

```bash
git add .github
git commit -m "ci: pages deploy and PR strict-build workflows, CODEOWNERS"
```

- [ ] **Step 6: Create the GitHub repo and push**

Run (host or box, `gh` is authenticated as rjk11111):
```bash
cd ~/Development/UOCompanion/coalition-wiki
gh repo create UOCompanion/coalition-wiki --public --source=. --remote=origin \
  --description "The Coalition wiki: Ultima Online Outlands boating community" \
  --homepage https://wiki.thecoalition.app --push
```
Expected: repo created, `main` pushed. The push triggers `Deploy to GitHub Pages`, which will fail at `deploy-pages` until Pages is enabled in the next step; that is expected.

- [ ] **Step 7: Enable Pages with the workflow build type and rerun**

```bash
gh api -X POST repos/UOCompanion/coalition-wiki/pages -f build_type=workflow
gh workflow run "Deploy to GitHub Pages" --repo UOCompanion/coalition-wiki
sleep 90
gh run list --repo UOCompanion/coalition-wiki --workflow "Deploy to GitHub Pages" --limit 1 --json conclusion,url
```
Expected: `"conclusion":"success"`. If the POST returns 409 the site already exists; run `gh api -X PUT repos/UOCompanion/coalition-wiki/pages -f build_type=workflow` instead.

- [ ] **Step 8: Verify the deployed site**

```bash
curl -sI https://uocompanion.github.io/coalition-wiki/ | head -1
curl -s https://uocompanion.github.io/coalition-wiki/ | grep -c "The Coalition Wiki"
```
Expected: `HTTP/2 200` (a 301 to the custom domain is also fine if Task 4 has already run) and a count of at least 1.

Note: because `docs/CNAME` is present, GitHub sets the custom domain immediately on deploy and may redirect the `github.io` URL to `wiki.thecoalition.app` before DNS exists. If `curl` shows a 301 to the custom domain, proceed to Task 4.

---

### Task 4: Custom domain and HTTPS

**Files:** none in the repo (`docs/CNAME` already exists). Changes are in Cloudflare DNS and GitHub settings.

**Interfaces:**
- Consumes: the deployed Pages site from Task 3.
- Produces: `https://wiki.thecoalition.app/` serving the wiki with a valid certificate.

- [ ] **Step 1: Confirm the domain is not yet served (failing test)**

```bash
curl -sI --max-time 10 https://wiki.thecoalition.app/ | head -1 || echo "not resolving"
dig +short wiki.thecoalition.app CNAME
```
Expected: no answer from `dig`, curl fails.

- [ ] **Step 2: Add the DNS record in Cloudflare (maintainer action)**

In the Cloudflare dashboard for `thecoalition.app`, DNS > Records > Add:
- Type `CNAME`, Name `wiki`, Target `uocompanion.github.io`, Proxy status **DNS only**, TTL Auto.

Checkpoint: this step requires the maintainer's Cloudflare access. Stop and ask if it cannot be done by the executor.

Verify:
```bash
dig +short wiki.thecoalition.app CNAME
```
Expected: `uocompanion.github.io.`

- [ ] **Step 3: Set the custom domain on the repo and wait for the certificate**

```bash
gh api -X PUT repos/UOCompanion/coalition-wiki/pages -f cname=wiki.thecoalition.app
for i in $(seq 1 20); do
  s=$(gh api repos/UOCompanion/coalition-wiki/pages --jq '.https_certificate.state')
  echo "cert: $s"; [ "$s" = "approved" ] && break; sleep 30
done
```
Expected: state progresses `new` -> `authorization_created` -> `authorized` -> `approved` within about ten minutes. If it stalls at `errored`, confirm the DNS record is DNS-only (grey cloud) and retry the PUT.

- [ ] **Step 4: Enforce HTTPS and verify**

```bash
gh api -X PUT repos/UOCompanion/coalition-wiki/pages -F https_enforced=true
curl -sI https://wiki.thecoalition.app/ | head -1
curl -sI http://wiki.thecoalition.app/ | grep -i '^location'
curl -s https://wiki.thecoalition.app/ | grep -c "The Coalition Wiki"
```
Expected: `HTTP/2 200`, an HTTP to HTTPS redirect `location: https://wiki.thecoalition.app/`, count at least 1.

- [ ] **Step 5: Verify the domain on the org (maintainer action, web UI)**

GitHub > UOCompanion org > Settings > Pages > Verified domains > Add `thecoalition.app`.
GitHub shows a TXT record name (`_github-pages-challenge-UOCompanion.thecoalition.app`) and value. Add that TXT record in Cloudflare DNS, then click Verify.

Verify:
```bash
dig +short TXT _github-pages-challenge-UOCompanion.thecoalition.app
```
Expected: the challenge value. The org settings page shows the domain as verified.

- [ ] **Step 6: Optional, proxy through Cloudflare**

Only after the certificate is `approved`: flip the `wiki` record to Proxied and set SSL/TLS mode to Full. Re-run the Step 4 curls. If anything breaks, switch back to DNS only. Skip this step if unsure; DNS-only is fully functional.

- [ ] **Step 7: Record the result**

No repo change. Note in the PR or commit log that the domain is live, then move to Task 5.

---

### Task 5: Contributor access and branch protection

**Files:** none in the repo. Changes are GitHub org and repo settings.

**Interfaces:**
- Consumes: the `strict-build` job name from `.github/workflows/pr-check.yml` (Task 3).
- Produces: `wiki-editors` team with write access; `main` protected so merges require a PR with a passing `strict-build` check.

- [ ] **Step 1: Confirm main is unprotected (failing test)**

```bash
gh api repos/UOCompanion/coalition-wiki/branches/main --jq .protected
```
Expected: `false`.

- [ ] **Step 2: Create the team and grant write access**

```bash
gh api -X POST orgs/UOCompanion/teams -f name=wiki-editors -f description="Trusted editors of wiki.thecoalition.app" -f privacy=closed
gh api -X PUT orgs/UOCompanion/teams/wiki-editors/repos/UOCompanion/coalition-wiki -f permission=push
gh api orgs/UOCompanion/teams/wiki-editors/repos --jq '.[].full_name'
```
Expected: last command prints `UOCompanion/coalition-wiki`. Add members later with
`gh api -X PUT orgs/UOCompanion/teams/wiki-editors/memberships/<github-login>`.

- [ ] **Step 3: Apply branch protection**

```bash
cat > /tmp/protection.json <<'EOF'
{
  "required_status_checks": { "strict": true, "contexts": ["strict-build"] },
  "enforce_admins": false,
  "required_pull_request_reviews": { "required_approving_review_count": 0 },
  "restrictions": null,
  "required_linear_history": true,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF
gh api -X PUT repos/UOCompanion/coalition-wiki/branches/main/protection --input /tmp/protection.json --jq '{checks: .required_status_checks.contexts, pr: .required_pull_request_reviews.required_approving_review_count, linear: .required_linear_history.enabled}'
```
Expected: `{"checks":["strict-build"],"pr":0,"linear":true}`.

Approval count is 0 on purpose: with a single maintainer as the only code owner, requiring one approval would block the maintainer's own PRs. CODEOWNERS still auto-requests the maintainer as reviewer on every PR. `enforce_admins` is false so the maintainer can bypass in an emergency.

- [ ] **Step 4: Prove the gate blocks a broken PR (end-to-end test)**

```bash
cd ~/Development/UOCompanion/coalition-wiki
git switch -c test/broken-link
printf '\nSee [nowhere](does-not-exist.md).\n' >> docs/guides/sweeping.md
git commit -am "test: deliberately broken link"
git push -u origin test/broken-link
gh pr create --title "test: broken link gate" --body "Should fail strict-build and be unmergeable." --base main
sleep 120
gh pr checks --json name,state
gh pr view --json mergeable,mergeStateStatus --jq '{mergeable, mergeStateStatus}'
```
Expected: `strict-build` state `FAILURE`; `mergeStateStatus` is `BLOCKED`.

- [ ] **Step 5: Prove a fixed PR turns green, then clean up**

```bash
git revert --no-edit HEAD
git push
sleep 120
gh pr checks --json name,state
gh pr close --delete-branch
git switch main && git pull
```
Expected: `strict-build` state `SUCCESS` after the revert. PR closed, branch deleted, `main` untouched.

- [ ] **Step 6: Final verification against the spec**

```bash
curl -sI https://wiki.thecoalition.app/ | head -1
curl -s https://wiki.thecoalition.app/search/search_index.json | python3 -c "import sys,json; print(len(json.load(sys.stdin)['docs']), 'search entries')"
gh api repos/UOCompanion/coalition-wiki/branches/main --jq .protected
gh api repos/UOCompanion/coalition-wiki/pages --jq '{build_type, cname, https_enforced}'
```
Expected: `HTTP/2 200`; a positive search entry count; `true`; `{"build_type":"workflow","cname":"wiki.thecoalition.app","https_enforced":true}`.
