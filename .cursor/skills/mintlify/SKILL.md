---
name: mintlify
description: Writes and maintains Mintlify documentation for this project. Use when editing .mdx files, adding pages, updating docs.json, or working with Mintlify components.
---

# Mintlify Docs

## Project context

This is a Mintlify documentation site. Full writing rules and component reference live in [.cursor/rules/writing.mdc](.cursor/rules/writing.mdc). Follow that file for language, style, and component usage.

## Running locally

```bash
npx mintlify dev
```

Opens at http://localhost:3000 (or next available port).

## Adding a new page

1. Create the `.mdx` file with YAML frontmatter:
   ```yaml
   ---
   title: "Page Title"
   description: "Brief description for SEO and nav."
   ---
   ```
2. Add the page to `docs.json` under the correct group:
   ```json
   "pages": [
     "path/to/new-page"
   ]
   ```
   Path is relative to docs root, no `.mdx` extension.

## docs.json structure

- **navigation.tabs[].groups[]**: Each group has `group` (sidebar label) and `pages` (array of paths).
- Page paths: Use slashes, no extension. Example: `"feature-guides/backtests"` → `feature-guides/backtests.mdx`.

## Component quick reference

| Use case | Component |
|----------|-----------|
| Procedures | `<Steps>` + `<Step title="…">` |
| Callouts | `<Tip>`, `<Warning>`, `<Info>`, `<Note>`, `<Check>` |
| Platform alternatives | `<Tabs>` + `<Tab title="…">` |
| Images | `<Frame>` around `<img>` |
| Related links | `<Card>` or `<CardGroup cols={2}>` + `<Card … href="…">` |
| API params | `<ParamField path="…" type="…">` |
| API responses | `<ResponseField name="…" type="…">` |
| Collapsible content | `<AccordionGroup>` + `<Accordion title="…">` |

## Removing a page

1. Delete the `.mdx` file.
2. Remove its path from `docs.json` navigation.
3. Search for internal links (`href="/path/to/page"`) and update or remove them.

## Checklist for edits

- [ ] Frontmatter has `title` and `description`
- [ ] Images wrapped in `<Frame>` with `alt` text
- [ ] Links use internal paths (`/quickstart` not `quickstart.mdx`)
- [ ] Steps use `<Steps>` / `<Step>` for procedures
