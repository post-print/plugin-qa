---
name: postprint
description: Use PostPrint MCP tools for projects, personal library, literature discovery, search, citations, and bibliographies. Use when the user works with papers, DOIs, projects, finding or adding literature, or PostPrint data.
---

# PostPrint MCP

Complete OAuth when the client prompts. All tools run as the signed-in user.

## MCP server id

Possible server ids: **postprint-dev**, **postprint-qa**, **postprint** (production). Use whichever PostPrint tools are actually available in your session—those tool names reflect the active server. If several appear, prefer **postprint** for production unless the user said otherwise. **list_projects** is a simple connectivity check.

## Identifiers

- **project_id** / **organization_id**: UUID or slug (URL segment).
- **source_id**: PostPrint internal id for a paper (UUID). **Not** a DOI, arXiv id, or title.

To get a **source_id** before graph or detail calls: use **search_sources** (public search), **list_project_sources**, or **list_user_library** and read **source_id** from the returned row. Never pass a DOI or arXiv id into **list_source_references**, **list_source_cited_by**, or **list_source_related**—those tools need **source_id** only.

## Workspace and library

- **list_projects**: optional **organization_id**, **q** (matches name or description).
- **get_project**: one project by id or slug.
- **list_project_sources**: papers on a project — identifiers (doi, openalex, arxiv), venue metadata, ingest status, URLs, **project_link** (nickname, who linked, when). Optional **q**, **limit** (1–500, default 50), **offset**, **tag_ids** (AND semantics, user tag UUIDs), **year_from**, **year_to**. Paginate with **limit** / **offset** when needed.
- **get_source**: full detail — identifiers, abstract, links, files, **tags** (tags are here, not on the list endpoints above).
- **list_user_library**: saved library; same optional filters as list semantics — **q**, **limit** (1–500), **offset**, **tag_ids**, **year_from**, **year_to**.
- **list_recently_viewed_papers**: extension-tracked visits; returns **`{ history, total }`**; optional **limit** (1–200), **offset**. Often **empty** without the browser extension—then use **list_user_library** or ask the user which papers to use.
- **list_connected_sources**: related papers from recent views or optional **project_id**; returns **`{ sources: [{ source, relevance_score }], total }`**; optional **limit** (1–500), **offset**.

## Graph

- **list_source_references** / **list_source_cited_by** / **list_source_related**: citation / related graph for a **source_id**; optional **include_sources**. Responses include **total** and tool-specific top-level keys (e.g. citations vs related list shape—inspect the tool result). Resolve **source_id** first (see Identifiers).

## Project management

- **create_project**: **name**; optional **description**, **organization_id** (org id/slug; requires org admin), **visibility** `private` | `link` | `public` (default private). Without **organization_id**, creates a **personal** project.
- **update_project**: **project_id**; optional **name**, **description**, **visibility**. Project admin only.
- **delete_project**: **project_id**. **Permanent** (hard delete). Project admin only. Confirm with the user before calling.

## Search and ingest

- **search_sources**: external literature search; **limit** clamped **1–50** (default 10). Use before ingest when the paper is not already in PostPrint.
- **add_sources_to_project**: **project_id** required.

  **Path A — new ingest (paper not yet in PostPrint / from search):** pass **`sources`** — a list of objects. Each object needs enough signal for ingest (not a bare catalog landing URL alone). Valid patterns include at least one of: **doi**, **arxiv_id**, **pmid**, **abstract_id**, **title**, **pdf_url** (direct PDF), OpenAlex **work_id** or URL under `https://openalex.org/W...`, plus optional **url**, or **pdf_base64** + **pdf_filename**. Items that are only a generic publisher page with no ids/title/pdf/OpenAlex work link are **rejected**.

  **Path B — already in your library:** pass **`source_ids`** (PostPrint UUID strings). Each id must **already** be in the user's saved library (`not_in_library` / similar in **failed** if not). Do **not** pass **`source_ids`** alone for rows straight from **search_sources** until those papers exist in the library—use Path A with **`sources`**, or **add_sources_to_library** first, then link with **`source_ids`**.

  You may combine **`sources`** and **`source_ids`** in one call. Returns **`succeeded`** and **`failed`** — surface **failed** reasons to the user.

- **remove_sources_from_project**: unlink from one project; returns **`removed`** count.
- **add_sources_to_library** / **remove_sources_from_library**: personal library; same batch shapes and **`succeeded`** / **`failed`** patterns as project add where applicable.

## Bibliography and invites

- **get_bibliography**: exactly one of non-empty **`source_ids`** or **`project_id`** (mutually exclusive). **`styles`**: optional list including `apa`, `mla`, `bibtex`; omitted or **[]** behaves like **`["bibtex"]`**. Returns **`citations`** and may include **`failed`** entries — report failures to the user.
- **invite_users**: **`emails`**; exactly one of **`project_id`** or **`organization_id`**. **`role`**: `admin` | `member` | `guest`. Optional **`message`**. Response includes **`invites`** and **`errors`** (partial success possible). On **PermissionError**, tell the user they may need admin on the project or organization.

## Workflows

1. **Find and add a new paper to a project:** **search_sources** → pick work → **add_sources_to_project** with **`sources`** (structured fields from search, e.g. OpenAlex work id/url or DOI). If the user needs confirmation, follow with **list_project_sources** and check ingest status.
2. **Add an existing library paper to a project:** ensure the paper is in the library (**list_user_library** / **get_source**) → **add_sources_to_project** with **`source_ids`** only (or ingest via **`sources`** if it is not saved yet).
3. **Citation neighborhood for a named paper:** **search_sources** or **list_project_sources** / **list_user_library** → read **`source_id`** → **list_source_references** / **list_source_cited_by** / **list_source_related** as needed.

## Conventions

Prefer **list_projects**, **list_project_sources**, **list_user_library**, and **search_sources** before guessing ids. For human project titles, resolve with **list_projects** (**q**) or **get_project**.
