Push the project to GitHub, generate a polished README with a live screenshot, deploy via GitHub Pages, and update the repo About section with the live URL.

GitHub repo URL: $ARGUMENTS

---

## Step 1 — Parse the repo URL

Extract `OWNER` and `REPO` from the provided URL (e.g. `https://github.com/alice/my-site` → owner=`alice`, repo=`my-site`).
If no URL was provided, stop and ask: "Please run /github-push <GitHub repo URL>".

---

## Step 2 — Take a screenshot with Playwright MCP

The local dev server should already be running on http://localhost:8080 (started by the Stop hook). If it is not responding, start it now with:
```
node -e "const h=require('http'),fs=require('fs');h.createServer((_,r)=>{r.writeHead(200,{'Content-Type':'text/html;charset=utf-8'});fs.createReadStream('index.html').pipe(r)}).listen(8080)" &
```

Then use the Playwright MCP browser tool to:
1. Navigate to `http://localhost:8080`
2. Wait for the page to fully load (wait for network idle)
3. Take a full-page screenshot and save it as `screenshot.png` in the project root

If Playwright MCP is not available, skip the screenshot and note it in the README.

---

## Step 3 — Gather project facts for the README

Read `index.html` to extract:
- The site title (from `<title>` or the hero heading)
- A one-sentence description of the project
- The list of all six sections (Hero, About, Packages, Gallery, Booking Form, Contact)
- The form submission endpoint email
- External dependencies (Google Fonts, Unsplash, formsubmit.co)

Build a directory tree of the project (excluding `.git`).

---

## Step 4 — Create README.md

Write `README.md` in the project root with the following structure. Use real values from Step 3 — do NOT use generic placeholders.

```markdown
# <Site Title>

<One-sentence description of the project>

![Site Screenshot](screenshot.png)

---

## Tech Stack

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Google Fonts](https://img.shields.io/badge/Google_Fonts-4285F4?style=for-the-badge&logo=google&logoColor=white)

## Live Demo

🌐 **[View Live Site](https://<OWNER>.github.io/<REPO>)**

---

## About the Project

<2–3 paragraph description of the site: what it is, who it is for, and the design aesthetic (elegant, romantic, blush-pink + gold). Mention the photographer persona and the six sections.>

---

## File Structure

\`\`\`
<repo-name>/
├── index.html          # All HTML, CSS, and JavaScript — single file, no build step
├── screenshot.png      # Hero screenshot (auto-generated)
├── .github/
│   └── workflows/
│       └── deploy.yml  # GitHub Pages deployment workflow
└── .claude/
    ├── settings.json   # Claude Code project settings (Stop hook)
    └── commands/
        └── github-push.md  # This slash command
\`\`\`

---

## Sections

| Section | Description |
|---------|-------------|
| **Hero** | Full-viewport background image with CTA button |
| **About** | Two-column photographer bio |
| **Packages** | Three tiered pricing cards (Essentials / Signature / Luxury) |
| **Gallery** | CSS-columns masonry grid with 6 Unsplash images |
| **Booking Form** | Nine-field enquiry form (name, email, phone, date, time, package, location, guests, message) |
| **Contact** | Contact details, embedded Google Map, and footer |

---

## How to Use

### Run locally

Node.js must be installed. No build step or package manager required.

\`\`\`bash
git clone https://github.com/<OWNER>/<REPO>.git
cd <REPO>
node -e "const h=require('http'),fs=require('fs');h.createServer((_,r)=>{r.writeHead(200,{'Content-Type':'text/html;charset=utf-8'});fs.createReadStream('index.html').pipe(r)}).listen(8080,()=>console.log('http://localhost:8080'))"
\`\`\`

Then open [http://localhost:8080](http://localhost:8080).

### Customise

| What to change | Where |
|----------------|-------|
| Colour palette | CSS custom properties on `:root` in `index.html` |
| Photos | Unsplash `src` URLs in the `#gallery` section |
| Packages & pricing | `.package-card` elements in the `#packages` section |
| Form email | `fetch` URL in the inline `<script>` at the bottom |
| Photographer bio | `#about` section |

---

## Deployment

The site is automatically deployed to **GitHub Pages** on every push to `main` via the included GitHub Actions workflow (`.github/workflows/deploy.yml`).

Live URL: **https://<OWNER>.github.io/<REPO>**
```

---

## Step 5 — Create the GitHub Actions workflow

Create `.github/workflows/deploy.yml`:

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
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## Step 6 — Initialize git and push to GitHub

Run the following in sequence. Stop and report any error before proceeding:

```bash
# 1. Initialize git if not already a repo
git init

# 2. Stage everything
git add .

# 3. Commit
git commit -m "Initial commit: Lumière Bridal Photography site + GitHub Pages deployment"

# 4. Rename branch to main (safe even if already on main)
git branch -M main

# 5. Add remote (skip if it already exists)
git remote add origin $ARGUMENTS || git remote set-url origin $ARGUMENTS

# 6. Push
git push -u origin main
```

---

## Step 7 — Enable GitHub Pages and update repo About

After the push succeeds:

1. Use the `gh` CLI to enable GitHub Pages from the `gh-actions` source:
```bash
gh api repos/<OWNER>/<REPO>/pages \
  --method POST \
  -f build_type=workflow \
  -f source='{"branch":"main","path":"/"}' 2>/dev/null || true
```

2. Wait ~5 seconds for the Pages environment to be created, then get the live URL:
```bash
gh api repos/<OWNER>/<REPO>/pages --jq '.html_url'
```

3. Update the repo About (description + website):
```bash
gh repo edit <OWNER>/<REPO> \
  --description "Elegant single-page bridal photography booking site — HTML/CSS/JS, no build step" \
  --homepage "https://<OWNER>.github.io/<REPO>"
```

4. Add the topics:
```bash
gh repo edit <OWNER>/<REPO> --add-topic "bridal-photography" --add-topic "single-page" --add-topic "html-css-js" --add-topic "github-pages"
```

---

## Step 8 — Report back

Print a summary:

```
✅ Pushed to:     https://github.com/<OWNER>/<REPO>
🌐 Live site:     https://<OWNER>.github.io/<REPO>  (GitHub Pages deploys in ~1 min)
📄 README:        Created with screenshot and full documentation
⚙️  CI/CD:         .github/workflows/deploy.yml — auto-deploys on push to main
```
