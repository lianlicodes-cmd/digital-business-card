# Decap CMS Setup — My Notes

How to wire a hand-coded site to Decap CMS, and the mistakes to avoid.

---

## Before you start: folder structure

Get this right first. Most problems come from here.

```
project-folder/
├── index.html          ← the site page
├── admin/
│   ├── index.html      ← the CMS panel
│   └── config.yml      ← CMS settings
├── content/
│   └── links.json      ← editable content
└── images/
```

**Three rules:**

1. No spaces in any folder or file name
2. Never nest a folder inside another with the same name
3. `admin` and `content` sit next to `index.html`, not inside anything else

---

## Setup steps

1. Build the page and get it working first
2. Push to GitHub
3. Settings → Pages → branch `main`, folder `/ (root)` → Save
4. Check the live site loads before touching Decap
5. Create `admin/index.html` with the Decap script
6. Create `admin/config.yml`
7. Sign up at DecapBridge → Add site → connect the repo
8. Paste the DecapBridge auth block at the top of `config.yml`
9. Visit `yoursite.com/admin/` → Login → check it loads
10. Invite the client as a collaborator — they set their own password

---

## Wiring a page to Decap

Decap saves content into files. The page has to read those files. That's the wiring.

**Step 1 — pick what should be editable**

Start with one thing. Links, or images, or text. Not all of it.

**Step 2 — move that content into a file**

Create `content/links.json` and put the content in it:

```json
[
  {
    "title": "Design",
    "url": "https://example.com/",
    "description": "Short description here.",
    "image": "https://example.com/photo.webp"
  }
]
```

**Step 3 — empty the HTML and give it an id**

Replace the hardcoded blocks with one empty container:

```html
<div id="link-cards"></div>
```

**Step 4 — add the fetch code**

Paste this just above `</body>`. The whole block, including the script tags:

```html
<script>
fetch('content/links.json')
  .then(res => res.json())
  .then(links => {
    const wrap = document.getElementById('link-cards');
    wrap.innerHTML = links.map(link => `
      <a href="${link.url}">
        <div>${link.title}</div>
        <div>${link.description}</div>
      </a>
    `).join('');
  });
</script>
```

**Step 5 — test it**

Change something in the JSON file, save, hard refresh the browser.
If it changes, the wiring works.

**Then repeat for the next page.** Wire each page as you finish it, not all at the end.

---

## Things that went wrong last time

**A space in a folder name**
The folder was `admin ` not `admin`. Every link 404'd.
→ If a URL shows `%20`, that's a space. Fix the name.

**Two folders with the same name**
`Linkboard` inside `Linkboard`. Files split across both, nothing could find anything.
→ One project folder. Flat structure.

**Several `index.html` files**
Editing one, viewing another. Looked like changes did nothing.
→ Check the breadcrumb path at the top of the VS Code editor before editing.

**Live Server showing the wrong page**
`127.0.0.1:5500/` served a different folder than the one being edited.
→ Make the URL match the file. Add the folder path if needed.

**Code pasted outside the script tags**
`fetch` ended up above `<script>` instead of inside it. Silently did nothing.
→ Paste the whole block, tags included.

**Browser cache**
Changes were saved but the old page kept showing.
→ Always hard refresh: `Ctrl + Shift + R`.

---

## Quick checks when something isn't working

- Does the URL match the file being edited?
- Hard refreshed, not just refreshed?
- Any `%20` in the URL?
- Is the fetch path correct relative to `index.html`?
- Is the JSON valid? A missing comma breaks the whole file.

---

## For client handover

- Keep this README in the repo
- Comment anything non-obvious in the code
- Note which JSON file feeds which page
