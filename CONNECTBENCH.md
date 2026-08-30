# ConnectBench install client — tool behaviour

**Status:** Normative for ConnectBench Phase 7a+  
**Schema:** Uses `wappler-install.json` **schema 1** unchanged — no new manifest fields required

ConnectBench is a compatible install client alongside the [Mr Cheese Extension Installer](https://www.mrcheese.co.uk/extensions/install). It installs **`.cbext`** bundles (zip of an extension repo root) into an opened Wappler Node consumer project.

---

## What ConnectBench adds beyond manifest copy

Schema v1 defines **file copy** (`directories`, `copy`, `notes`). ConnectBench also performs the **npm companion lane** required for Wappler App Connect IDE integration:

| Step | Source | Target |
|------|--------|--------|
| Manifest copy | `serverConnect` / `appConnect` `copy[]` | Project `extensions/`, `lib/modules/`, `public/` |
| Source sync | Bundle root | `{extensionsLibrary}/{cloneDir}` (from manifest `cloneDir`) |
| npm registration | Extension `package.json` `name` | Project `package.json` `devDependencies` as `file:…/{cloneDir}` |
| Wappler registry | Extension `package.json` `name` | `.wappler/project.json` → `extensions[]` |
| node_modules link | `npm install --save-dev wappler-*@file:…` | `node_modules/wappler-*/` (IDE reads `app_connect/components.hjson` here) |
| Install record | — | `.connectbench/installed-extensions.json` |

**ConnectBench does not run Wappler Project Updater** — manifest copy already places runtime files in `public/` and `extensions/`.

---

## Why npm registration matters (App Connect)

Wappler loads the **Elements picker** and property panel from:

```text
node_modules/wappler-your-extension/app_connect/components.hjson
```

Files copied to `extensions/app_connect/components/` are the **runtime project copy**. Copying manifest files alone does **not** show the component group (e.g. **Mr Cheese**) in the IDE until `node_modules` is linked via successful `npm install`.

Authors should include these bullets in manifest `appConnect.notes[]`:

- Register `wappler-your-extension` in Project Settings → Extensions (or ensure it is in `package.json` devDependencies).
- Run `npm install` in the project root.
- Quit Wappler completely and reopen.

ConnectBench performs steps 2–3 automatically when the consumer project's other `file:` deps are valid.

---

## Extensions library path (`file:` layout)

ConnectBench resolves the npm `file:` path as:

```text
file:{extensionsLibrary}/{cloneDir}
```

| Resolution order | Value |
|------------------|--------|
| 1 | `.connectbench/project.json` → `extensionsLibrary` |
| 2 | Inferred from existing `package.json` `file:…/Extensions/…` entries |
| 3 | Default `../Server-Connect/Extensions` |

**`cloneDir`** in the manifest (e.g. `greatRangePicker`) becomes the folder name under the library.

**Anti-patterns:**

- `file:` pointing at ConnectBench app cache or `.connectbench/extensions/`
- Absolute paths in consumer `package.json`
- `cloneDir` that does not match the synced folder name

---

## Uninstall behaviour

ConnectBench uninstall removes:

1. Files listed in `installedPaths` (created at install), or legacy fallbacks:
   - `overwrittenPaths` from `.connectbench/installed-extensions.json`
   - `copy[].dest` from `wappler-install.json` in the extension source folder
2. `devDependencies` entry and `.wappler/project.json` registration
3. `node_modules` link via `npm uninstall wappler-*`

**Does not delete:** extension source in the shared library (`{extensionsLibrary}/{cloneDir}`).

**Does not revert:** files that were overwritten at install — use git.

---

## Author recommendations (no schema change)

### `appConnect.notes[]` template (npm + ConnectBench)

```json
"notes": [
  "npm: add wappler-your-extension to package.json devDependencies (file:../Server-Connect/Extensions/yourCloneDir) and run npm install.",
  "Register in .wappler/project.json extensions if the Elements picker stays empty.",
  "Quit Wappler completely (including tray) and reopen.",
  "ConnectBench .cbext installs perform manifest copy + npm registration automatically."
]
```

### `package.json` (extension repo)

Required for ConnectBench npm registration:

```json
{
  "name": "wappler-your-extension",
  "keywords": ["wappler", "wappler-extension"]
}
```

### `cloneDir`

Set explicitly when the repo folder name differs from `id`:

```json
{
  "id": "great-range-picker",
  "cloneDir": "greatRangePicker"
}
```

---

## Consumer project troubleshooting

| Symptom | Likely cause |
|---------|----------------|
| No **Mr Cheese** (or publisher) group in Elements | `node_modules/wappler-*` missing |
| Manifest files on disk, picker still empty | npm lane incomplete |
| `npm install` fails | Another `wappler-*` `file:` path points at a missing folder |
| `npm install` ignores `package.json` changes | Stale `node_modules/.package-lock.json` |

---

## Future schema v2 (not required today)

A future `schema: 2` might add structured `npm` fields (`packageName`, `extensionsLibrary`). **v1 manifests work unchanged** — ConnectBench infers npm metadata from the extension repo `package.json` and consumer project layout.

---

## Related

- [SCHEMA.md](./SCHEMA.md) — field reference
- [README.md](./README.md) — standard overview
- ConnectBench: `docs/14-premium-extensions.md`, `knowledge/experts/extension/knowledge.md`
