# MkDocs Setup & Deployment — Full Workflow

## 1. Repo / Directory Layout

**Original**  

```
ShopWIKI/
├── docs/
│   ├── index.md
│   └── ipad-restore.md
├── mkdocs.yml
└── .github/
    └── workflows/
        └── deploy.yml
```

**Issue**  
GitHub Pages 404’d and new pages (like our iPad guide) never appeared.

**Final**  
Either rename `docs/` to the repo root **or** tell MkDocs where your docs live:

```yaml
# mkdocs.yml
site_name: ShopWIKI
docs_dir: '.'        # if you want to store .md files at the repo root
theme:
  name: material
plugins:
  - search
  - awesome-pages
```

> We kept our files in `/docs`, so we left the default and did _not_ set `docs_dir:`.

---

## 2. `mkdocs.yml` Configuration

```yaml
site_name: ShopWIKI

theme:
  name: material

plugins:
  - search
  - awesome-pages   # auto-builds the sidebar from your folder structure
```

- **site_name**: shown in the navbar  
- **plugins**: include `awesome-pages` so new files drop into the nav automatically  

---

## 3. Install Dependencies

```bash
pip install mkdocs-material mkdocs-awesome-pages-plugin
```

- `mkdocs-material`: the Material theme  
- `mkdocs-awesome-pages-plugin`: auto-generates your sidebar  

---

## 4. Local Preview

```bash
mkdocs serve
```

- Open http://127.0.0.1:8000  
- Live-reload as you edit  
- Sidebar updates with every `.md` in `docs/`  

---

## 5. GitHub Actions Workflow

Create or update `.github/workflows/deploy.yml`:

```yaml
name: Deploy MkDocs
on:
  push:
    branches: [ main ]

jobs:
  build:
    permissions:
      contents: write       # allow pushing to gh-pages
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      - run: pip install mkdocs-material mkdocs-awesome-pages-plugin

      - run: mkdocs gh-deploy --force --verbose
```

1. **Checkout** code  
2. **Setup Python**  
3. **Install** MkDocs + plugins  
4. **Deploy** to the `gh-pages` branch  

---

## 6. Troubleshooting & Lessons

1. **404 on Pages**  
   
   - Ensure GitHub Pages is set to serve from the `gh-pages` branch.  

2. **“awesome-pages plugin not installed”**  
   
   - Add `mkdocs-awesome-pages-plugin` to your `pip install` step.  

3. **Local vs GitHub mismatch**  
   
   - Commit & push every `.md` you add.  
   - Confirm `docs_dir` matches where your files live.  

4. **Site Title wrong**  
   
   - Update `site_name` in `mkdocs.yml`.  

5. **Adding new pages**  
   
   - With **awesome-pages**, just drop a `.md` into `docs/`.  
   
   - Or manually define in `mkdocs.yml` under `nav:` if you need custom order:
     
     ```yaml
     nav:
      - Home: index.md
      - iPad Restore: ipad-restore.md
      - MkDocs Guide: mkdocs-setup-guide.md
     ```

---

## 7. Next Steps

- Add front-matter (e.g. `title:`) to customize page titles.  
- Explore other plugins: redirects, include-markdown, etc.  
- When you’re ready, compare with GitBook or other hosted wikis—but this GitHub+MkDocs setup is rock-solid for now.  
