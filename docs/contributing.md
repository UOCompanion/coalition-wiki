# Contributing

Members of the wiki-editors team branch in this repository; anyone else forks it. Either way,
changes go through a pull request and are published automatically once merged.

## Edit a page on GitHub

1. Open the page on the wiki and click the pencil icon, or browse to the file under `docs/`
   in the [repository](https://github.com/UOCompanion/coalition-wiki).
2. Edit the Markdown and choose "Create a new branch and start a pull request" (on a fork,
   GitHub proposes the fork branch automatically).
3. Wait for the "strict-build" check. It fails on links to pages that do not exist and on bad
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

    ```markdown
    !!! tip
        Bring extra cannon shot for mini bosses.
    ```

## Preview locally (optional)

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```
