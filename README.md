# PostPrint plugin (Cursor + Claude Code)

Bundled **skills**, **rules**, **agents**, **hooks** (placeholder), and **MCP** config for the PostPrint Django server at `apps/backend/postprint/mcp/`.

Source lives under `apps/plugin/`. **Built** output is `dist/dev`, `dist/qa`, and `dist/prod` (gitignored). MCP URL is injected from `.env` / `.env.qa` / `.env.production` into [`.mcp.json`](./.mcp.json) (placeholder `${POSTPRINT_MCP_URL}`) at build time.

### Cursor marketplace (name / icon)

Cursor resolves the plugin from the **git repo root**. In this monorepo the manifest lives under `apps/plugin/`, so pasting only the monorepo URL does not find `.cursor-plugin/plugin.json` unless a root catalog exists. Use **[`.cursor-plugin/marketplace.json`](../../.cursor-plugin/marketplace.json)** at the repo root: it points `source` at `apps/plugin`. The manifest still references `assets/logo.svg`; that path exists only **after a build** (`dist/*/assets/logo.svg`, copied from [`tspackages/theme`](../../tspackages/theme/assets/logo.svg)), so the **monorepo URL may not show an icon** until you commit a logo under `apps/plugin/assets/` or point `logo` at an absolute URL—optional.

For **[`{org}/plugin`](./DISTRIBUTION.md)** and **`{org}/plugin-qa`**, the published tree is the flat `dist/` output: the build copies the theme logo to `assets/logo.svg` at the repo root—no root `marketplace.json` required there.

### Claude Code (marketplace)

Claude Code expects **`.claude-plugin/marketplace.json` at the repository root** of whatever you pass to `/plugin marketplace add` ([marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces)). This monorepo adds **[`.claude-plugin/marketplace.json`](../../.claude-plugin/marketplace.json)** so the same git URL can register the catalog; the plugin files are under `apps/plugin`. Distribution builds **generate** `.claude-plugin/marketplace.json` into `dist/*` so **`plugin`** and **`plugin-qa`** each expose a catalog (marketplace ids **`postprint`** vs **`postprint-qa`** so you can add both).

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
/plugin install postprint@postprint-qa
/reload-plugins
```

**This monorepo** (`YOUR_ORG/applications` or full `https://github.com/YOUR_ORG/applications.git`):

```
/plugin marketplace add YOUR_ORG/applications
/plugin install postprint@postprint-apps
/reload-plugins
```

From a shell (same CLI as [Claude Code plugin commands](https://code.claude.com/docs/en/discover-plugins)):

```bash
claude plugin marketplace add YOUR_ORG/plugin
claude plugin install postprint@postprint

claude plugin marketplace add YOUR_ORG/plugin-qa
claude plugin install postprint@postprint-qa

claude plugin marketplace add YOUR_ORG/applications
claude plugin install postprint@postprint-apps
```

Refresh listings after upstream changes: `/plugin marketplace update postprint` (or `postprint-qa` / `postprint-apps`). Remove a catalog: `/plugin marketplace remove <name>`.

## Local development

From `apps/plugin/`:

```bash
bun run dev
bun run build
bun run build:qa
```

From monorepo root:

```bash
bun run dev:plugin
```

This builds `dist/dev/`, symlinks `~/.cursor/plugins/local/postprint` → `apps/plugin/dist/dev`, and registers `postprint@local` in `~/.claude/plugins/installed_plugins.json` + `~/.claude/settings.json`.

Restart Cursor (or **Developer: Reload Window**) and Claude Code. If rules/skills are missing, enable third-party plugins under **Settings → Features**.

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
dist/*/assets/logo.svg             # build copies from tspackages/theme (manifest icon in distribution repos)
dist/*/.claude-plugin/marketplace.json   # generated at build (postprint vs postprint-qa)
../.cursor-plugin/marketplace.json # monorepo root — Cursor catalog → apps/plugin
../.claude-plugin/marketplace.json # monorepo root — Claude catalog → ./apps/plugin
.env / .env.qa / .env.production   # POSTPRINT_MCP_URL per environment
.mcp.json                          # placeholder URL → dist/*/.mcp.json (build)
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
