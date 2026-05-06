Push the project to GitHub, generate a polished README with a live screenshot, deploy via GitHub Pages, and update the repo About with the live URL.

Usage: /github-push <GitHub repo URL>

GitHub repo URL: $ARGUMENTS

---

## Step 1 — Parse inputs

Extract `OWNER` and `REPO` from the URL argument (e.g. `https://github.com/alice/my-site` → owner=`alice`, repo=`my-site`).

If no URL was provided, stop and ask: **"Please run /github-push <GitHub repo URL>"** — do not proceed.

---

## Step 2 — Require a GitHub token

A Personal Access Token is needed to enable GitHub Pages and update the repo About via the API.

Check for a token in this order:
1. `$env:GH_TOKEN` or `$env:GITHUB_TOKEN` environment variable
2. Ask the user: "Please provide a GitHub Personal Access Token with **repo** scope. You can create one at https://github.com/settings/tokens/new — then run: `! $env:GH_TOKEN = 'ghp_...'`"

Do not proceed until a token is available. Store it as `$TOKEN` for all API calls below.

All GitHub API calls use these PowerShell headers:
```powershell
$headers = @{
    Authorization = "token $TOKEN"
    "User-Agent"  = "PowerShell"
    Accept        = "application/vnd.github+json"
}
```

---

## Step 3 — Take a screenshot with Playwright MCP

The local dev server should already be running on http://localhost:8080. If not, start it:
```
node -e "const h=require('http'),fs=require('fs');h.createServer((_,r)=>{r.writeHead(200,{'Content-Type':'text/html;charset=utf-8'});fs.createReadStream('index.html').pipe(r)}).listen(8080)"
```

Use the Playwright MCP browser tool to:
1. Navigate to `http://localhost:8080`
2. Wait for network idle (page fully loaded)
3. Take a full-page screenshot and save it as `screenshot.png` in the project root

If Playwright MCP is unavailable, skip the screenshot and in the README replace the screenshot line with:
```
> Screenshot not available — visit the [live site](https://<OWNER>.github.io/<REPO>) to preview.
```

---

## Step 4 — Scan the project and gather facts

Read the main source files to extract:
- **Site / app title** — from `<title>`, hero heading, or `package.json` `name`
- **One-sentence description** of what the project does and who it is for
- **Tech stack** — detect from file extensions and imports (HTML, CSS, JS, React, Vue, Tailwind, Node, etc.)
- **Sections / pages** — top-level nav links or routes
- **External dependencies** — fonts, APIs, form endpoints, CDNs
- **How to run locally** — check for `package.json` scripts, CLAUDE.md instructions, or plain Node/static server

Build a directory tree of the project excluding `.git`, `node_modules`, and build output folders.

---

## Step 5 — Create README.md

Write `README.md` in the project root. Use only real values — no generic placeholders.

```markdown
# <Site Title>

<One-sentence description>

![Site Screenshot](screenshot.png)

---

## Tech Stack

<!-- Include only the badges that match the actual tech detected in Step 4 -->
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Vue](https://img.shields.io/badge/Vue.js-35495E?style=for-the-badge&logo=vue.js&logoColor=4FC08D)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-43853D?style=for-the-badge&logo=node.js&logoColor=white)
![Google Fonts](https://img.shields.io/badge/Google_Fonts-4285F4?style=for-the-badge&logo=google&logoColor=white)

## Live Demo

🌐 **[View Live Site](https://<OWNER>.github.io/<REPO>)**

---

## About the Project

<2–3 paragraphs: what the project is, who it is for, the design aesthetic or technical approach, and notable features.>

---

## File Structure

\`\`\`
<REPO>/
<directory tree from Step 4>
\`\`\`

---

## How to Use

### Run locally

\`\`\`bash
git clone https://github.com/<OWNER>/<REPO>.git
cd <REPO>
<install and run commands detected in Step 4>
\`\`\`

### Customise

| What to change | Where |
|----------------|-------|
<rows relevant to this specific project>

---

## Deployment

Automatically deployed to **GitHub Pages** on every push to `main` via `.github/workflows/deploy.yml`.

Live URL: **https://<OWNER>.github.io/<REPO>**
```

---

## Step 6 — Create the GitHub Actions workflow

If `.github/workflows/deploy.yml` does not already exist, create it:

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

If the file already exists and is correct, skip this step.

---

## Step 7 — Commit and push to GitHub

Run in sequence — stop and report any error before continuing:

```powershell
# 1. Init git if needed
git init

# 2. Ensure on main branch
git branch -M main

# 3. Set remote (add or update)
git remote add origin https://github.com/<OWNER>/<REPO>.git
# if remote already exists:
git remote set-url origin https://github.com/<OWNER>/<REPO>.git

# 4. Stage and commit
git add .
git commit -m "Add README, screenshot, and GitHub Pages deployment"

# 5. Push
git push -u origin main
```

---

## Step 8 — Make repo public (if needed) and enable GitHub Pages

### 8a — Check repo visibility
```powershell
$repo = Invoke-RestMethod -Uri "https://api.github.com/repos/<OWNER>/<REPO>" -Headers $headers
$repo.visibility   # "public" or "private"
```

If the repo is **private**, make it public (GitHub Pages requires a public repo on free accounts):
```powershell
$body = '{"private":false,"visibility":"public"}'
Invoke-RestMethod -Uri "https://api.github.com/repos/<OWNER>/<REPO>" -Method PATCH -Headers $headers -Body $body -ContentType "application/json"
```

If the user does not want the repo to be public, stop and explain they need GitHub Pro for Pages on private repos.

### 8b — Enable GitHub Pages with GitHub Actions source
```powershell
$body = '{"build_type":"workflow"}'
try {
    Invoke-RestMethod -Uri "https://api.github.com/repos/<OWNER>/<REPO>/pages" -Method POST -Headers $headers -Body $body -ContentType "application/json"
} catch {
    # 409 = already exists, update it
    if ($_.Exception.Response.StatusCode.value__ -eq 409) {
        Invoke-RestMethod -Uri "https://api.github.com/repos/<OWNER>/<REPO>/pages" -Method PUT -Headers $headers -Body $body -ContentType "application/json"
    }
}
```

### 8c — Trigger deployment
Push an empty commit to fire the workflow:
```powershell
git commit --allow-empty -m "Trigger GitHub Pages deployment"
git push origin main
```

---

## Step 9 — Update repo About

Set the description, homepage URL, and topics:

```powershell
# Description and homepage
$body = '{
  "description": "<one-sentence description from Step 4>",
  "homepage": "https://<OWNER>.github.io/<REPO>"
}'
Invoke-RestMethod -Uri "https://api.github.com/repos/<OWNER>/<REPO>" -Method PATCH -Headers $headers -Body $body -ContentType "application/json"

# Topics (replace with tags relevant to this project's actual tech stack)
$body = '{"names":["github-pages","html-css-js","single-page"]}'
Invoke-RestMethod -Uri "https://api.github.com/repos/<OWNER>/<REPO>/topics" -Method PUT -Headers $headers -Body $body -ContentType "application/json"
```

---

## Step 10 — Report back

Print this summary (fill in real values):

```
✅ Pushed to:   https://github.com/<OWNER>/<REPO>
🌐 Live site:   https://<OWNER>.github.io/<REPO>  (GitHub Actions deploys in ~1 min)
📄 README:      Created with badges, file structure, how-to, and screenshot
⚙️  CI/CD:       .github/workflows/deploy.yml — auto-deploys on every push to main
📝 Repo About:  Description and homepage updated
```

Watch the deployment at: `https://github.com/<OWNER>/<REPO>/actions`
