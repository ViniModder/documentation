# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About this project

This is a documentation site built on [Mintlify](https://mintlify.com) — a documentation-as-code platform. Pages are MDX files with YAML frontmatter. All site structure, styling, and navigation are controlled by [docs.json](docs.json). There is no build pipeline in this repo; Mintlify handles compilation and deployment.

For deep Mintlify product knowledge (components, configuration, writing standards), install the Mintlify skill:

```bash
npx skills add https://mintlify.com/docs
```

## Commands

```bash
npm i -g mint        # Install Mintlify CLI (requires Node.js v19+)
mint dev             # Start local preview at http://localhost:3000
mint broken-links    # Validate all links across the docs
mint update          # Update the CLI to the latest version
```

Deployment is automatic: push to `main` and the Mintlify GitHub app redeploys the site.

## Architecture

### Navigation & configuration

All navigation lives in [docs.json](docs.json) — tabs, groups, page order, colors, logo, and API settings. Adding a new page requires registering it here under the correct tab/group as well as creating the `.mdx` file.

Two top-level tabs:
- **Guides** — grouped into Getting started, Customization, Writing content, AI tools
- **API reference** — driven by [api-reference/openapi.json](api-reference/openapi.json) (OpenAPI 3.1.0)

### Content

Each page is an `.mdx` file with a frontmatter block:

```mdx
---
title: "Page title"
description: "Short description"
icon: "icon-name"
---
```

Reusable content lives in [snippets/](snippets/) and is imported into pages with `<Snippet>` components.

Images and static assets go in [images/](images/) and [logo/](logo/).

Files listed in [.mintignore](.mintignore) (e.g., `drafts/`, `*.draft.mdx`) are excluded from the build.

### API reference

The sample API spec is at [api-reference/openapi.json](api-reference/openapi.json). Endpoint pages in [api-reference/endpoint/](api-reference/endpoint/) reference operations from that spec.

## Style preferences

- Use active voice and second person ("you")
- Keep sentences concise — one idea per sentence
- Use sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references
