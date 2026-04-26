# Alfred Documentation

Source for [docs.alfred.black](https://docs.alfred.black) — Alfred's user and API documentation, served by [Mintlify](https://mintlify.com).

If you're looking to **use** Alfred, read the live docs. This repository is for the people who maintain them.

---

## What's in here

```
.
├── docs.json              ← Mintlify config + navigation (single source of truth for nav)
├── openapi.yaml           ← OpenAPI spec consumed by Mintlify's API panel
├── introduction.mdx       ← Documentation tab landing page
├── quickstart.mdx         ← First-run onboarding walkthrough
├── how-it-works.mdx       ← The seven-layer architecture overview
├── architecture/          ← One page per architectural layer (8 pages)
│   ├── infrastructure.mdx
│   ├── agent.mdx
│   ├── data.mdx
│   ├── semantic.mdx
│   ├── kinetic.mdx
│   ├── interface.mdx
│   └── channels.mdx
├── vault/                 ← How the vault is organised (3 pages)
├── guides/                ← User-facing how-tos (19 pages)
├── reference/faq.mdx      ← Frequently asked questions
├── api-reference/         ← One page per ctrl-api endpoint (165 pages, 21 groups)
│   ├── introduction.mdx
│   ├── authentication.mdx
│   ├── vault/
│   ├── streams/
│   ├── integrations/      ← Composio
│   ├── email/             ← AgentMail
│   ├── phone/             ← AgentPhone (SMS + voice)
│   ├── auth-senders/
│   ├── chores/
│   ├── approvals/
│   ├── intuition/         ← Observations, instincts, judgment
│   ├── workers/
│   ├── workflows/
│   ├── schedules/
│   ├── cross-tenant/      ← Alfred Prime federated tools
│   ├── openclaw/
│   ├── agents/
│   ├── models/
│   ├── tools/
│   ├── workspace/
│   ├── notifications/
│   ├── terminal/
│   ├── devices/
│   ├── credentials/
│   ├── logs/
│   └── admin/             ← Container ops on the tenant VPS
├── snippets/              ← Reusable .mdx fragments imported by other pages
└── images/                ← Diagrams, screenshots, icons
```

Total: **198 pages** across **26 navigation groups**, every page verified against current source in [`ssdavidai/alfred-platform`](https://github.com/ssdavidai/alfred-platform) and [`ssdavidai/alfred`](https://github.com/ssdavidai/alfred).

---

## Live preview

Mintlify ships a local dev server. From the repo root:

```bash
npm i -g mintlify
mintlify dev
```

Open <http://localhost:3000>. Edits to `.mdx` files hot-reload.

To check for broken links and orphaned files before committing:

```bash
mintlify broken-links
```

---

## Editing rules

These are non-negotiable. The docs only stay trustworthy if every contributor follows them.

### Voice and brand

- The user is **"Sir"** (capital S). Always.
- Alfred is a **butler service**, not "an AI tool" or "a platform". Premium, quietly competent, never breathless.
- "Your" possessive everywhere — "your vault", "your butler", "your tasks".
- Avoid: *powerful, robust, seamless, intelligent, smart, leverage, ecosystem, cutting-edge, revolutionary*. If you reach for one of those words, you're describing the marketing of the thing instead of the thing itself.
- Concrete > abstract. Name the actual function, the actual file, the actual flow.

### Verification

Every concrete claim in this repo must trace back to source code. Before writing about a feature:

1. Read the route handler in `ssdavidai/alfred-platform/packages/ctrl/src/api/routes/<name>.ts`.
2. Read the workflow in `ssdavidai/alfred-platform/packages/learn/src/workflows/<name>.py`.
3. Read the activity, skill, or template the route invokes.
4. If the source disagrees with what you were going to write, trust the source.

If a feature isn't in the code, it doesn't get a page. No "coming soon", no aspirational documentation, no placeholders.

### Page conventions

Every page starts with this frontmatter:

```yaml
---
title: "Page Title"
description: "One-sentence summary that says exactly what this page covers."
icon: "lucide-icon-name"
---
```

API pages add an `api:` field so Mintlify renders the request/response panel:

```yaml
---
title: "Endpoint name"
description: "What this endpoint does in one sentence."
api: "POST /api/v1/path"
---
```

Use Mintlify's components where they help comprehension — `<Note>`, `<Tip>`, `<Warning>`, `<Steps>`, `<Card>`, `<CardGroup>`, `<Tabs>`, `<ParamField>`, `<ResponseField>`, `<RequestExample>`, `<ResponseExample>`. Don't decorate. A wall of text isn't fixed by sprinkling Cards.

### Navigation

`docs.json` is the single source of truth for what appears in the sidebar. Every `.mdx` file on disk should be in `docs.json`; every entry in `docs.json` should exist on disk. To check:

```bash
python3 -c "
import json, os, glob
d = json.load(open('docs.json'))
nav = {p for tab in d['navigation']['tabs'] for g in tab['groups'] for p in g.get('pages', [])}
disk = {f[:-4] for f in glob.glob('**/*.mdx', recursive=True) if not f.startswith('snippets/')}
print('missing on disk:', nav - disk)
print('orphans (not in nav):', disk - nav)
"
```

Both sets should be empty.

---

## Adding a new endpoint

When a new route lands in ctrl-api, the docs should follow within the same PR.

1. Write `api-reference/<group>/<endpoint-name>.mdx` from the route handler. Use an existing endpoint page as a template.
2. Add the page path to the appropriate group in `docs.json`.
3. Run the nav-vs-disk check above.
4. `mintlify dev` and click the new page to confirm it renders.
5. Commit with `docs(api): add <group>/<endpoint>`.

---

## Adding a new architectural concept

If something material changes in the platform — a new specialist, a new channel, a new layer:

1. Decide whether it's a new section (its own group in nav) or an addition to an existing page.
2. Read the relevant source files end-to-end before writing.
3. If it's a new section, add a `<Card>` link to it from `how-it-works.mdx` and from any related `architecture/*.mdx` page so it's discoverable.
4. Update `introduction.mdx` only if the change is user-visible enough to merit it.

---

## Deployment

Mintlify auto-deploys from `main` on push. There's no separate build step in this repo — Mintlify builds in their cloud.

Open a PR for any non-trivial change. The Mintlify GitHub app posts a preview URL in the PR; review the rendered output before merging.

---

## Source code references

The docs describe these repositories:

- **[ssdavidai/alfred-platform](https://github.com/ssdavidai/alfred-platform)** — private monorepo: SaaS app, fleet control plane (ctrl-api), intelligence layer (learn), Docker images (openclaw), voice bridge.
- **[ssdavidai/alfred](https://github.com/ssdavidai/alfred)** — public Python package: vault workers (curator, janitor, distiller, surveyor) and the `alfred` CLI.

Most API pages reference handlers in `alfred-platform/packages/ctrl/src/api/routes/`. Most architectural pages reference workflows in `alfred-platform/packages/learn/src/workflows/` and activities in `packages/learn/src/activities/`.

---

## License

Documentation © Alfred Black. The underlying `alfred` Python package is open source under its own license; see that repository.
