# Git-born Wappler extensions: install manifests, a free installer & manifest builder on Mr Cheese 🧀

Hi everyone — **Mr Cheese** here.

Something we’ve been building quietly has grown into something we’re genuinely proud of, and we’d love to share it with the Wappler community.

It started as a simple idea: *“Help people install my extensions without hunting through README `cp` blocks.”*  
That became a **browser install wizard**, then a **standard install manifest format**, then a **manifest builder for authors** — all live on **[mrcheese.co.uk](https://www.mrcheese.co.uk/extensions)** today, free, no account required.

We hope this helps anyone who ships extensions via **GitHub + manual copy** (the path many of us use alongside npm / Project Updater). Feedback very welcome.

---

## Index

| Section | For who |
|--------|---------|
| [TL;DR](#tldr) | Everyone |
| [The problem](#the-problem) | Context |
| [For Wappler users — Extension Installer](#for-wappler-users--extension-installer) | Installing extensions |
| [For extension authors — `wappler-install.json`](#for-extension-authors--wappler-installjson) | Publishing extensions |
| [Manifest Builder](#manifest-builder) | Creating manifests |
| [Glossary](#glossary) | Terms |
| [Mr Cheese extensions using this today](#mr-cheese-extensions-using-this-today) | Examples |
| [Note for the Wappler team](#note-for-the-wappler-team) | Platform & product |
| [Links & resources](#links--resources) | Quick reference |

---

## TL;DR

- **`wappler-install.json`** — a small JSON file at the **root of an extension Git repo** that describes *which files to copy where* and *what to do after* (restart Wappler, env vars, designer steps, etc.).
- **[Extension Installer](https://www.mrcheese.co.uk/extensions/install)** — pick Server Connect / App Connect / both, choose an extension (or paste any public GitHub URL) → get `mkdir` + `git clone` + `cp` scripts to run from your **open Wappler project root** in a terminal.
- **[Manifest Builder](https://www.mrcheese.co.uk/extensions/manifest-builder)** — create a valid `wappler-install.json` in your browser and download it for your repo.
- **Mr Cheese extensions** on GitHub already ship this file; the installer **fetches it automatically** (with a fallback if a repo doesn’t have one yet).

Your README stays the human guide. The manifest is for **tools**.

---

## The problem

Many community extensions (including ours) are distributed like this:

1. Clone or download from GitHub  
2. Copy files into `extensions/`, `lib/modules/`, `public/`, etc.  
3. Restart Wappler  

Install steps live in **README prose** — great for reading, awkward for tooling.

Every author writes slightly different `cp` instructions. Install wizards can’t help without duplicating that data somewhere else — and it drifts out of date when files move.

**`wappler-install.json`** is our answer: one machine-readable file, versioned with the extension, that any compatible tool can read.

> This is a **community proposal** (de facto standard), not an official Wappler format — though we’d be delighted if the platform team found it useful.

---

## For Wappler users — Extension Installer

**URL:** [mrcheese.co.uk/extensions/install](https://www.mrcheese.co.uk/extensions/install)

### What it does

Runs entirely in your browser. It does **not** write to your machine — it generates terminal commands **you** review and run.

### Typical flow

1. **Choose layer** — Server Connect, App Connect, or **Both** (e.g. PuSH-IT needs both).
2. **Pick extension** — Mr Cheese catalog, or **Custom GitHub repo** with any public clone URL. Have a terminal open in your **Wappler project root** (`app/`, `lib/`, `extensions/`).
3. **Optional** — path to an already-cloned repo (skips `git clone`).
4. **Enable** *“Use `wappler-install.json` from the repository”* (on by default) — the wizard fetches the manifest from GitHub.
5. **Commands** (step 3):
   - **Create folders** — `mkdir -p` with paths relative to your project root (copyable)
   - **After copy** — designer / env / restart notes from the manifest
   - **Copy files** — `git clone` (alongside your project) + `cp` into relative folders (copyable)

A status line tells you whether install data came from **the repo manifest** or the **bundled catalog** (fallback).

### What you still do manually

- Run the script in your terminal  
- Set env vars in Wappler (VAPID, API keys, etc.) — the manifest only *points* you to the README  
- Designer steps (e.g. add Browser component for Redirect-IT)  
- **Quit Wappler completely and restart**

---

## For extension authors — `wappler-install.json`

### What it is

| | |
|---|---|
| **Filename** | `wappler-install.json` |
| **Location** | Repository root (next to `README.md`) |
| **Schema** | `"schema": 1` |
| **Purpose** | Install automation — not runtime behaviour |

### What goes inside

```json
{
  "schema": 1,
  "id": "redirect-it",
  "name": "Redirect-IT",
  "version": "1.0.1",
  "cloneDir": "Wappler-Redirect-IT-Extension",
  "serverConnect": {
    "directories": ["extensions/server_connect/modules", "lib/modules"],
    "copy": [
      { "src": "redirectit.js", "dest": "lib/modules/redirectit.js" }
    ],
    "notes": [
      "Quit Wappler completely and restart."
    ]
  }
}
```

**Server Connect + App Connect in one file?** Yes. Use both `serverConnect` and `appConnect` sections (see PuSH-IT on GitHub). Install tools merge them when the user picks “Both”.

### Live example (raw JSON on GitHub)

- [Redirect-IT — `wappler-install.json`](https://github.com/MrCheeseGit/Wappler-Redirect-IT-Extension/blob/main/wappler-install.json)
- [PuSH-IT — `wappler-install.json`](https://github.com/MrCheeseGit/Wappler-PuSH-IT-Extension/blob/main/wappler-install.json) *(Server + App Connect)*

Fetch URL pattern tools use:

```text
https://raw.githubusercontent.com/{owner}/{repo}/main/wappler-install.json
```

### What it is **not**

- Not a replacement for `package.json` or **Project Updater** / npm flows  
- Not a Wappler IDE registration format — you still wire App Connect via Updater or `project.json` where needed  
- Not a place for secrets — only env var *names* in notes, never values  

---

## Manifest Builder

**URL:** [mrcheese.co.uk/extensions/manifest-builder](https://www.mrcheese.co.uk/extensions/manifest-builder)

Free browser wizard for authors:

1. Extension identity (name, id slug, version)  
2. Layers (Server / App / both)  
3. Copy map — with shortcuts (e.g. “add Server Connect module files”)  
4. Post-install notes — with snippets (restart Wappler, env vars, Browser component, npm)  
5. **Preview & export** — validate, copy JSON, download `wappler-install.json`  

Commit the file to your repo root. Mention it in your README Installation section. Done.

**JSON Schema** (for validators / IDEs): [mrcheese.co.uk/data/wappler-install.schema.json](https://www.mrcheese.co.uk/data/wappler-install.schema.json)

---

## Glossary

| Term | Meaning |
|------|---------|
| **Git-born extension** | Distributed via public Git repo + manual file copy (vs npm-only or marketplace). |
| **`wappler-install.json`** | Standard install manifest file (schema v1). |
| **Install manifest** | Machine-readable copy map + folders + post-install notes. |
| **Extension Installer** | Mr Cheese wizard that generates `mkdir` / `clone` / `cp` scripts for end users. |
| **Manifest Builder** | Mr Cheese wizard that helps authors create `wappler-install.json`. |
| **`serverConnect` section** | Copy paths for HJSON steps, `lib/modules/`, routes hooks, etc. |
| **`appConnect` section** | Copy paths for components, `public/js`, `public/css`, service workers. |
| **Bundled catalog** | Fallback install data on mrcheese.co.uk when a repo has no manifest or fetch fails. |
| **Schema v1** | Current manifest version (`"schema": 1`). Breaking changes would increment this. |

---

## Mr Cheese extensions using this today

All listed on **[mrcheese.co.uk/extensions](https://www.mrcheese.co.uk/extensions)**. Each repo root includes `wappler-install.json` + README install section.

| Extension | GitHub | Manifest |
|-----------|--------|----------|
| **PuSH-IT** | [Wappler-PuSH-IT-Extension](https://github.com/MrCheeseGit/Wappler-PuSH-IT-Extension) | Server + App Connect |
| **Redirect-IT** | [Wappler-Redirect-IT-Extension](https://github.com/MrCheeseGit/Wappler-Redirect-IT-Extension) | Server Connect |
| **ClickSend SMS** | [Wappler-ClickSend-SMS-Extension](https://github.com/MrCheeseGit/Wappler-ClickSend-SMS-Extension) | Server Connect |
| **Cache Machine** | [Wappler-Cache-Machine-Extension](https://github.com/MrCheeseGit/Wappler-Cache-Machine-Extension) | Server Connect |
| **Wap-Lastic** | [Wap-Lastic-Wappler-Elastic-Search-Extension](https://github.com/MrCheeseGit/Wap-Lastic-Wappler-Elastic-Search-Extension) | Server Connect |
| **Generate Auth Code** | [Wappler-Generate-Auth-Code-Extension](https://github.com/MrCheeseGit/Wappler-Generate-Auth-Code-Extension) | Server Connect |

Try the installer with **PuSH-IT** and **Both** selected — you’ll see both copy maps and notes pulled from one manifest file.

---

## Note for the Wappler team

*A short note for Wappler staff / extensibility folks — feel free to skip if you’re here to install extensions.*

### Context

Wappler’s documented distribution paths include **npm + `wappler-extension` + Project Updater** (especially App Connect). Many community extensions — including Mr Cheese’s — also ship as **flat Git repos** with README `cp` instructions for Server Connect modules, routes hooks, and manual App Connect file drops.

There is no shared, tool-readable install metadata for that Git path today.

### What we’re proposing

A minimal, optional convention:

- **One file:** `wappler-install.json` at repo root  
- **Schema v1:** `copy[]` (src → dest), `directories[]`, `notes[]`, optional `serverConnect` / `appConnect` sections  
- **Tool-agnostic:** any client can fetch from GitHub raw and generate install scripts  

We are **not** asking Wappler to implement this in core — we’ve dogfooded it on mrcheese.co.uk. If the team ever wanted to:

- Document it alongside [custom extensions](https://docs.wappler.io/t/wappler-extensibility-build-custom-wappler-extensions/48499)  
- Add IDE / Updater hooks to read install manifests  
- Or standardise a filename / schema officially  

…we’d be happy to collaborate. The JSON Schema lives in [Wappler-Git-Extension-Manifest-Standard](https://github.com/MrCheeseGit/Wappler-Git-Extension-Manifest-Standard) on GitHub.

### Gaps we’re **not** claiming to solve

- IDE extension registration (`.wappler/project.json`, component picker)  
- PHP Server Connect layouts (schema v1 is Node-oriented; PHP could be a future `schema: 2` section)  
- Replacing npm publish workflows  

### Why we built it

- **Single source of truth** — install paths live in the repo that ships the files  
- **Less README drift** — tools read JSON; README covers env vars, examples, troubleshooting  
- **Community-friendly** — any Git-born author can adopt without Mr Cheese approval  

---

## Links & resources

### Specification (this standard)

| Resource | URL |
|----------|-----|
| **GitHub repo** | [Wappler-Git-Extension-Manifest-Standard](https://github.com/MrCheeseGit/Wappler-Git-Extension-Manifest-Standard) |
| JSON Schema (v1) | [wappler-install.schema.json](https://github.com/MrCheeseGit/Wappler-Git-Extension-Manifest-Standard/blob/main/wappler-install.schema.json) |

### Mr Cheese (live tools)

| Resource | URL |
|----------|-----|
| Extension catalog | [mrcheese.co.uk/extensions](https://www.mrcheese.co.uk/extensions) |
| **Extension Installer** | [mrcheese.co.uk/extensions/install](https://www.mrcheese.co.uk/extensions/install) |
| **Manifest Builder** | [mrcheese.co.uk/extensions/manifest-builder](https://www.mrcheese.co.uk/extensions/manifest-builder) |

### Wappler docs (official)

- [Build custom Wappler extensions](https://docs.wappler.io/t/wappler-extensibility-build-custom-wappler-extensions/48499)  
- [Extensibility forum category](https://community.wappler.io/c/docs/wappler-extensibility/58)  

### Mr Cheese on GitHub

- [MrCheeseGit repositories](https://github.com/MrCheeseGit?tab=repositories)  

---

## We’re proud of this — and we hope it helps

Honestly: watching this go from *“can we generate a bash script?”* to a **reusable convention other authors can adopt** has been one of the most satisfying side-projects we’ve shipped for the Wappler ecosystem.

It’s **free**, **browser-based**, and **doesn’t lock you in** — if you never use the tools, a `wappler-install.json` in your repo still documents install paths clearly for humans and future tooling.

If you try the **Installer** or **Manifest Builder**, or add a manifest to your own extension repo, we’d love to hear how it went — what’s confusing, what’s missing, whether the Wappler team sees value in something like this longer term.

Thanks for reading. 🧀

— **Mr Cheese**  
[mrcheese.co.uk](https://www.mrcheese.co.uk) · [GitHub](https://github.com/MrCheeseGit) · [Buy me a coffee](https://buymeacoffee.com/mrcheese)
