# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file bridal photography booking website (`index.html`). All HTML, CSS, and JavaScript live inline in that one file — no build step, no package manager, no framework.

## Running Locally

Node.js is available; serve with:

```
node -e "const h=require('http'),fs=require('fs');h.createServer((_,r)=>{r.writeHead(200,{'Content-Type':'text/html;charset=utf-8'});fs.createReadStream('index.html').pipe(r)}).listen(8080,()=>console.log('http://localhost:8080'))"
```

Python is **not** installed on this machine.

## Architecture

`index.html` is structured as six sequential sections, each targeted by nav anchor links:

| ID | Purpose |
|----|---------|
| `#hero` | Full-viewport background image with CTA |
| `#about` | Two-column photographer bio |
| `#packages` | Three pricing cards |
| `#gallery` | CSS-columns masonry grid (6 Unsplash images) |
| `#booking` | Nine-field enquiry form |
| `#contact` | Contact details, map iframe, footer |

**CSS** uses CSS custom properties (`--blush`, `--gold`, `--dark`, `--bg`, etc.) defined on `:root` — edit the palette there.

**Form submission** POSTs JSON to `https://formsubmit.co/ajax/youch@i-mxms.com` via `fetch`. On success the form is hidden and `#successCard` is shown. The submit endpoint email is the only external dependency.

**Images** are all Unsplash URLs with `?w=NNN&auto=format&fit=crop` query params — swap URLs to change photos.

**Fonts** load from Google Fonts: `Cormorant Garamond` (headings) and `Lato` (body).

**JS** (inline `<script>` at bottom of `<body>`):
- Sticky nav toggled via `scrolled` class at 80 px
- Mobile hamburger menu (`openMenu` / `closeMenu`)
- Form validation (HTML5 `required` + phone regex)
- Package card clicks pre-select the matching `<select>` option in the booking form
