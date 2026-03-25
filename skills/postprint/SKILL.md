---
name: postprint
description: Use PostPrint MCP tools for projects, personal library, literature search, citations, and bibliographies. Use when the user works with papers, DOIs, projects, or PostPrint data.
---

# PostPrint MCP

Enable the PostPrint MCP server for your build (**postprint-dev**, **postprint-qa**, or **postprint**) and complete OAuth when prompted. All tools act as the signed-in user.

## Identifiers

- **project_id** / **organization_id**: record id or slug (URL segment).
- **source_id**: PostPrint source id for a paper you can access (library or visible project).

## Workspace and library

- **list_projects** / **get_project**: pick a project.
- **list_project_sources**: papers on a project with identifiers (doi, openalex, arxiv), venue metadata, ingest status, URLs, tags, **project_link** (nickname, who linked, when).
- **get_source**: full detail — identifiers, abstract, links, files, tags.
- **list_user_library**: papers the user saved personally (**saved_at** when added).
- **list_recently_viewed_papers**: extension-tracked research page visits (not the same as library; often empty if no extension).
- **list_connected_sources**: papers linked via citations/related edges from recent views or optional **project_id**.

## Graph and history

- **list_source_references** / **list_source_cited_by** / **list_source_related**: citation graph for a **source_id**; optional **include_sources** returns deduplicated sources.
- Use history tools from the server listing when you need chronological activity (see tool descriptions in the client).

## Search and ingest

- **search_sources**: find public literature to add.
- **add_sources_to_project**: pass **sources[]** with structured fields (doi, arxiv_id, pdf_url, OpenAlex work URL, etc.—like extension ingest, not bare landing pages only) and/or **source_id** for papers already in the library.
- **remove_sources_from_project**: removes from one project only.
- **add_sources_to_library** / **remove_sources_from_library**: personal library only.

## Bibliography and invites

- **get_bibliography**: pass exactly one of non-empty **source_ids** or **project_id**; optional **styles**: `apa`, `mla`, `bibtex` (omit or empty list for bibtex only).
- **invite_users**: **emails** and exactly one of **project_id** or **organization_id**; optional **role** `admin` | `member` | `guest` and short **message**; caller must be allowed to invite.

## Conventions

Prefer existing project/library tools before guessing ids. When the user names a project by human title, resolve via **list_projects** or **search_sources** as appropriate.
