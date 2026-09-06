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
