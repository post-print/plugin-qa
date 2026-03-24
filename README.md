# PostPrint plugin (Cursor + Claude Code)

Bundled **skills**, **rules**, **agents**, **hooks** (placeholder), and **MCP** config for the PostPrint Django server at `apps/backend/postprint/mcp/`.

Source lives under `apps/plugin/`. **Built** output is `dist/dev`, `dist/qa`, and `dist/prod` (gitignored). MCP URL is injected from `.env` / `.env.qa` / `.env.production` into [`.mcp.json`](./.mcp.json) (placeholder `${POSTPRINT_MCP_URL}`) at build time.

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
dist/*/assets/logo.svg             # copied from tspackages/theme at build (manifest icon)
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
