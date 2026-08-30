# Schema reference — `wappler-install.json` (version 1)

This document defines every field in schema **1**. Tools should reject or warn on unknown `schema` values.

---

## File rules

| Rule | Value |
|------|--------|
| Filename | `wappler-install.json` |
| Location | Repository root |
| Encoding | UTF-8 |
| Format | JSON (no comments — use a `.md` note if you need prose) |
| Path separators | Forward slash `/` only in all paths |
| Path style | Relative — no absolute paths, no `..` segments |

---

## Top-level object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schema` | integer | **yes** | Must be `1` for this document |
| `id` | string | **yes** | Stable slug, lowercase, hyphens (e.g. `push-it`, `redirect-it`) |
| `name` | string | **yes** | Human display name (e.g. `PuSH-IT`) |
| `version` | string | no | Extension version; semver recommended; should match release tag when possible |
| `cloneDir` | string | no | Suggested directory name for `git clone` (default: repo name or `extension-repo`) |
| `hasServerConnect` | boolean | no | Hint for UIs; default `true` if `serverConnect` present |
| `hasAppConnect` | boolean | no | Hint for UIs; default `true` if `appConnect` present |
| `serverConnect` | object | no* | Server Connect install section |
| `appConnect` | object | no* | App Connect install section |
| `readme` | string | no | URL to full human documentation (usually repo README) |

\* At least one of `serverConnect` or `appConnect` must be present.

---

## Install section (`serverConnect` / `appConnect`)

Same shape for both keys.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `directories` | string[] | no | Project-relative folders to create before copy (e.g. `extensions/server_connect/modules`) |
| `copy` | copy entry[] | no** | Files to copy from clone root to project |
| `notes` | string[] | no | Post-copy steps shown to the user (designer actions, env vars, restart) |

\** At least one of `directories` or `copy` should be present for a useful section.

### Copy entry

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `src` | string | **yes** | Path relative to **extension repo root** (clone directory) |
| `dest` | string | **yes** | Path relative to **Wappler project root** |

Example:

```json
{ "src": "pushit.js", "dest": "lib/modules/pushit.js" }
```

Duplicate `src` → different `dest` is allowed (common for `lib/modules/` and `extensions/server_connect/modules/`).

---

## Path conventions (Wappler Node project)

Typical destinations tools should understand:

| Destination | Purpose |
|-------------|---------|
| `extensions/server_connect/modules/` | HJSON step definitions |
| `extensions/server_connect/routes/` | Routes hooks (`*.js`) |
| `lib/modules/` | Server Connect module runtime JS |
| `extensions/app_connect/components/` | App Connect `*.hjson` component defs |
| `public/js/` | Client scripts |
| `public/css/` | Client styles |
| `public/` | Service workers, static assets |

Sources are relative to the **cloned extension repo**, e.g. `app_connect/includes/dmx-pushit-subscribe.js`.

---

## Notes (`notes` array)

Plain text strings. Rendered as bullet lists in install wizards.

**Good:**

- “In Wappler, add the App Connect Browser component and set ID to `browser`.”
- “Set VAPID_* env vars in Wappler — see README.”
- “Quit Wappler completely and restart.”
- “npm: add `wappler-your-extension` to `package.json` devDependencies (`file:../Server-Connect/Extensions/yourCloneDir`) and run `npm install`.”
- “Register the extension in `.wappler/project.json` if the Elements picker stays empty.”

**App Connect extensions — always mention both lanes when publishing npm packages:**

- Manifest copy (Git / ConnectBench `.cbext`) places runtime files in `extensions/` and `public/`.
- npm + `node_modules` is required for the **Wappler IDE** Elements picker and property panel.

**Avoid:**

- Secret values, API keys, connection strings  
- HTML (tools escape text; use plain language)  
- Long tutorials (link via `readme` instead)

---

## Tool behaviour (normative for compatible clients)

### Install type selection

| User selects | Tool uses |
|--------------|-----------|
| Server Connect only | `serverConnect` section |
| App Connect only | `appConnect` section |
| Both | merge `directories`, `copy`, `notes` from both sections (dedupe directories and notes) |

### ConnectBench (`.cbext` client)

ConnectBench implements schema v1 copy **plus** the npm companion lane for App Connect IDE integration. See [CONNECTBENCH.md](./CONNECTBENCH.md).

Summary for authors:

- Manifest `copy[]` alone does **not** populate the Wappler Elements picker — consumers also need `node_modules/wappler-*` linked via `npm install`.
- Set `cloneDir` when the repo folder name differs from `id`.
- Include npm + Wappler restart bullets in `appConnect.notes[]`.
- Extension `package.json` must use a `wappler-*` name and `wappler-extension` keyword for tool registration.

No additional manifest fields are required for ConnectBench in schema v1.

### Directory list

If `directories` is empty or missing, tools may default to:

- `extensions/server_connect/modules`
- `lib/modules`

…when Server Connect is involved.

### Clone root

For each `copy` entry:

```bash
cp "{cloneRoot}/{src}" "{projectRoot}/{dest}"
```

`cloneRoot` = user’s local clone path, or `./{cloneDir}` after `git clone`.

### Validation checklist

Tools should warn or fail softly when:

- `schema` is missing or not `1`  
- `copy[].src` or `copy[].dest` contains `..` or starts with `/`  
- `id` contains spaces or uppercase  
- No `serverConnect` / `appConnect` for the selected install type  

---

## Full minimal example

See [wappler-install.example.json](./wappler-install.example.json).

## Full realistic examples

See [examples/redirect-it.wappler-install.json](./examples/redirect-it.wappler-install.json) and [examples/push-it.wappler-install.json](./examples/push-it.wappler-install.json).

---

## Future schema versions (not in v1)

Possible additions for `schema: 2` if needed:

- `serverConnectPhp` section  
- `env` — list of required env var *names* (not values)  
- `npm` — alternative install via package name  
- `postInstall` — structured steps with `type: "designer" | "shell" | "manual"`  

Do not use these fields in v1 files.
