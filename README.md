# Wappler Git Extension Manifest Standard

**Status:** Active (schema v1) — community proposal, not an official Wappler format  
**Maintainer:** [Mr Cheese](https://www.mrcheese.co.uk)  
**Audience:** Authors of **Git-born** Wappler extensions and tools that generate install scripts

> Spec and docs for **`wappler-install.json`** — machine-readable install manifests for Git-born Wappler extensions (copy paths, folders, post-install notes).

---

## What is this?

A small JSON file at the **root of an extension Git repository** that describes how to install that extension into a Wappler Node project: which files to copy, which folders to create, and what to do after copying.

| Humans read | Tools read |
|-------------|------------|
| `README.md` | **`wappler-install.json`** |

---

## Live tools (Mr Cheese)

| Tool | URL |
|------|-----|
| Extension catalog | [mrcheese.co.uk/extensions](https://www.mrcheese.co.uk/extensions) |
| **Git Extension Installer** | [mrcheese.co.uk/extensions/install](https://www.mrcheese.co.uk/extensions/install) |
| **npm install assistant** | [mrcheese.co.uk/extensions/install/npm](https://www.mrcheese.co.uk/extensions/install/npm) |
| **Manifest Builder** | [mrcheese.co.uk/extensions/manifest-builder](https://www.mrcheese.co.uk/extensions/manifest-builder) |

The Git installer fetches `wappler-install.json` from GitHub when present, with a bundled fallback for extensions without a manifest. The npm assistant uses the same manifest to generate `cp` commands from `node_modules/wappler-*` (after you verify the package exists).

---

## Canonical schema

| Resource | Link |
|----------|------|
| Field reference | [SCHEMA.md](./SCHEMA.md) |
| JSON Schema | [wappler-install.schema.json](./wappler-install.schema.json) |
| Raw (for tools) | `https://raw.githubusercontent.com/MrCheeseGit/Wappler-Git-Extension-Manifest-Standard/main/wappler-install.schema.json` |

---

## Documentation

| File | Purpose |
|------|---------|
| [PROPOSAL.md](./PROPOSAL.md) | Full proposal — problem, goals, architecture |
| [SCHEMA.md](./SCHEMA.md) | Schema version 1 — field reference |
| [MANIFEST-BUILDER.md](./MANIFEST-BUILDER.md) | Author tool specification |
| [community-post.md](./community-post.md) | Draft Wappler Community / Discourse announcement |
| [wappler-install.example.json](./wappler-install.example.json) | Minimal valid example |
| [examples/](./examples/) | PuSH-IT and Redirect-IT samples |

---

## Quick example

Place at the **root** of your extension repo as **`wappler-install.json`**:

```json
{
  "schema": 1,
  "id": "redirect-it",
  "name": "Redirect-IT",
  "version": "1.0.1",
  "cloneDir": "Wappler-Redirect-IT-Extension",
  "serverConnect": {
    "directories": [
      "extensions/server_connect/modules",
      "extensions/server_connect/routes",
      "lib/modules"
    ],
    "copy": [
      { "src": "redirectit.hjson", "dest": "extensions/server_connect/modules/redirectit.hjson" },
      { "src": "redirectit.js", "dest": "lib/modules/redirectit.js" },
      { "src": "redirectit.js", "dest": "extensions/server_connect/modules/redirectit.js" },
      { "src": "redirectit_nav.js", "dest": "extensions/server_connect/routes/redirectit_nav.js" }
    ],
    "notes": [
      "Add the App Connect Browser component in Wappler and set ID to browser.",
      "Quit Wappler completely and restart."
    ]
  }
}
```

Fetch URL pattern:

```text
https://raw.githubusercontent.com/{owner}/{repo}/main/wappler-install.json
```

---

## Adopted by (Mr Cheese extensions)

| Extension | Repository |
|-----------|------------|
| PuSH-IT | [Wappler-PuSH-IT-Extension](https://github.com/MrCheeseGit/Wappler-PuSH-IT-Extension) |
| Redirect-IT | [Wappler-Redirect-IT-Extension](https://github.com/MrCheeseGit/Wappler-Redirect-IT-Extension) |
| ClickSend SMS | [Wappler-ClickSend-SMS-Extension](https://github.com/MrCheeseGit/Wappler-ClickSend-SMS-Extension) |
| Cache Machine | [Wappler-Cache-Machine-Extension](https://github.com/MrCheeseGit/Wappler-Cache-Machine-Extension) |
| Wap-Lastic | [Wap-Lastic-Wappler-Elastic-Search-Extension](https://github.com/MrCheeseGit/Wap-Lastic-Wappler-Elastic-Search-Extension) |
| Generate Auth Code | [Wappler-Generate-Auth-Code-Extension](https://github.com/MrCheeseGit/Wappler-Generate-Auth-Code-Extension) |

---

## What this is not

- **Not** a Wappler official format (unless Wappler adopt it later)
- **Not** a replacement for `package.json` or npm / Project Updater flows
- **Not** runtime config — install-time only

---

## License

[Mr Cheese Extension License v1.0](https://www.mrcheese.co.uk/extension-license) — see [LICENSE](./LICENSE).
