# PostPrint plugin (Cursor + Claude Code)

Bundled **skills**, **rules**, **agents**, **hooks** (placeholder), and **MCP** config for the PostPrint Django server at `apps/backend/postprint/mcp/`.

Source lives under `apps/plugin/`. **Built** output is `dist/dev`, `dist/qa`, and `dist/prod` (gitignored). Each build sets the **plugin id** and default **MCP server id** to **postprint-dev** / **postprint-qa** / **postprint** (dev / QA / prod). The Cursor manifest includes **`displayName`** **PostPrint Dev** / **PostPrint QA** / **PostPrint** (same pattern as plugins such as Figma in the Cursor plugin cache). [`.mcp.json`](./.mcp.json) is a **valid dev MCP config** (so IDEs parse the repo). **`bun run build`** writes **`dist/*/.mcp.json`** from **`.env`**. **`bun run dev`** can patch repo-root **`.mcp.json`** from **`.env`** when **`MCP_URL`** / **`POSTPRINT_MCP_URL`** is set (optional **`POSTPRINT_MCP_SERVER_NAME`**).

### Cursor marketplace (name / icon)

Cursor resolves the plugin from the **git repo root**. In this monorepo the manifest lives under `apps/plugin/`, so pasting only the monorepo URL does not find `.cursor-plugin/plugin.json` unless a root catalog exists. Use **[`.cursor-plugin/marketplace.json`](../../.cursor-plugin/marketplace.json)** at the repo root: it points `source` at `apps/plugin`. The manifest still references `assets/logo.svg`; that path exists only **after a build** (`dist/*/assets/logo.svg`, copied from [`tspackages/theme`](../../tspackages/theme/assets/logo.svg)), so the **monorepo URL may not show an icon** until you commit a logo under `apps/plugin/assets/` or point `logo` at an absolute URL—optional.

For **[`{org}/plugin`](./DISTRIBUTION.md)** and **`{org}/plugin-qa`**, the published tree is the flat `dist/` output: the build copies the theme logo to `assets/logo.svg` at the repo root—no root `marketplace.json` required there.

### Claude Code (marketplace)

Claude Code expects **`.claude-plugin/marketplace.json` at the repository root** of whatever you pass to `/plugin marketplace add` ([marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces)). This monorepo adds **[`.claude-plugin/marketplace.json`](../../.claude-plugin/marketplace.json)** so the same git URL can register the catalog; the plugin files are under `apps/plugin`. Distribution builds **generate** `.claude-plugin/marketplace.json` into `dist/*` with catalog id matching the plugin id (**`postprint`** on prod, **`postprint-qa`** on QA, **`postprint-dev`** on dev) so you can add prod and QA catalogs side by side.

In Claude Code, replace `YOUR_ORG` with your GitHub org (see [DISTRIBUTION.md](./DISTRIBUTION.md)).

**Production** (`YOUR_ORG/plugin`):

```
/plugin marketplace add YOUR_ORG/plugin
/plugin install postprint@postprint
/reload-plugins
```

**QA** (`YOUR_ORG/plugin-qa`):

```
/plugin marketplace add YOUR_ORG/plugin-qa
/plugin install postprint-qa@postprint-qa
/reload-plugins
```

**This monorepo** (`YOUR_ORG/applications` or full `https://github.com/YOUR_ORG/applications.git`):

```
/plugin marketplace add YOUR_ORG/applications
/plugin install postprint-dev@postprint-apps
/reload-plugins
```

From a shell (same CLI as [Claude Code plugin commands](https://code.claude.com/docs/en/discover-plugins)):

```bash
claude plugin marketplace add YOUR_ORG/plugin
claude plugin install postprint@postprint

claude plugin marketplace add YOUR_ORG/plugin-qa
claude plugin install postprint-qa@postprint-qa

claude plugin marketplace add YOUR_ORG/applications
claude plugin install postprint-dev@postprint-apps
```

Refresh listings after upstream changes: `/plugin marketplace update postprint`, `postprint-qa`, or `postprint-apps`. Remove a catalog: `/plugin marketplace remove <name>`.

## Local development

From `apps/plugin/`:

```bash
bun run dev
bun run build
bun run build:qa
```

From monorepo root:

```bash
bunx nx run plugin:dev
```

This **does not** run `dist/dev` build. It symlinks **`~/.cursor/plugins/local/postprint-dev` → `apps/plugin`** (source tree), runs **`claude plugin marketplace add`** on that path, removes stale **`@local`** plugin keys, and registers **`postprint-dev@postprint-dev`** in `~/.claude/plugins/installed_plugins.json` + `~/.claude/settings.json`. Edits to **skills**, **rules**, **agents**, **hooks**, and manifests apply after **`/reload-plugins`** (Claude) or **Reload Window** (Cursor).

If **`.env`** defines **`MCP_URL`** or **`POSTPRINT_MCP_URL`**, the script rewrites repo-root **`.mcp.json`** from env (optional **`POSTPRINT_MCP_SERVER_NAME`**; default **`postprint-dev`**). With no URL in env, **`.mcp.json`** is left as committed.

If **`url.https://github.com/.insteadOf`** is unset globally, the script sets it to **`git@github.com:`** so Claude can clone GitHub-based marketplaces over HTTPS. It does not override an existing value.

Use **`bun run build`** / **`bun run build:dev`** when you need **`dist/dev`** (e.g. distribution or a clean artifact).

**Workspace MCP:** [`.cursor/mcp.json`](../../.cursor/mcp.json) in this repo still points at localhost when you open the workspace (separate from the plugin bundle).

**Manual `--plugin-dir`:**

```bash
bunx nx run plugin:build:qa
claude --plugin-dir ./apps/plugin/dist/qa
```

## Deploy (QA / production)

Tag-driven CI pushes built `dist/` to distribution repos. One-time setup: [DISTRIBUTION.md](./DISTRIBUTION.md).

```bash
bun run deploy:qa plugin patch
bun run deploy:production plugin patch
```

## OAuth

PostPrint MCP uses **PKCE OAuth**. Cursor callback: `cursor://anysphere.cursor-mcp/oauth/callback` (must remain allowed in backend `MCP_ALLOWED_REDIRECT_URIS`).

Details: [apps/backend/postprint/mcp/README.md](../../backend/postprint/mcp/README.md).

## Distribution

- **Prod:** public `{org}/plugin` repo (`main`), built by `plugin-production` workflow — Cursor marketplace points here.
- **QA:** `{org}/plugin-qa` repo (`main`), built by `plugin-qa` workflow.
- **Claude Code:** Team marketplace or `/plugin marketplace add` with the git URL of the distribution repo.

## Layout (source)

```
assets/logo.svg                    # Cursor manifest icon (copy of tspackages/theme; symlinked dev uses this)
.claude-plugin/marketplace.json    # Claude directory marketplace for postprint-dev (symlinked dev; dist overwrites on build)
dist/*/assets/logo.svg             # build copies from tspackages/theme (distribution)
dist/*/.claude-plugin/marketplace.json   # generated at build (catalog id = postprint-dev | postprint-qa | postprint)
../.cursor-plugin/marketplace.json # monorepo root — Cursor catalog → apps/plugin
../.claude-plugin/marketplace.json # monorepo root — Claude catalog → ./apps/plugin
.env / .env.qa / .env.production   # MCP_URL / POSTPRINT_MCP_URL; optional POSTPRINT_MCP_SERVER_NAME
.mcp.json                          # dev-default MCP (valid schema); dist/*/.mcp.json from env (build); bun run dev can patch from .env
.cursor-plugin/plugin.json
.claude-plugin/plugin.json
skills/postprint/SKILL.md
rules/postprint.mdc
agents/research-assistant.md
hooks/hooks.json
package.json
scripts/build.ts
scripts/dev.ts
```
