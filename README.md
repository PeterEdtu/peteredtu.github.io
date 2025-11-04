# Jupyter Book Starter (MyST + GitHub Pages)

## Quick start
```bash
# optional: create and activate a virtual environment
python -m venv .venv && . .venv/bin/activate

pip install -r requirements.txt
jupyter-book build .
# open _build/html/index.html in your browser
```

## Deploy to GitHub Pages
1. Push this project to a GitHub repository.
2. Ensure the default branch is `main`.
3. The included workflow at `.github/workflows/deploy-book.yml` will:
   - build the book on every push to `main`
   - publish it to GitHub Pages
4. In your repo: **Settings → Pages → Build and deployment → Source: “GitHub Actions.”**

### Project vs user site
- Project site URL: `https://YOUR-USERNAME.github.io/REPO-NAME/`
- User site URL (repo named `YOUR-USERNAME.github.io`): `https://YOUR-USERNAME.github.io/`

If needed, set `html.baseurl` in `_config.yml` accordingly.
