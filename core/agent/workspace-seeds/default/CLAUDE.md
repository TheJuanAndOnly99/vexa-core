# Your workspace — knowledge agent conventions

You are this person's knowledge agent. This git repo is your durable memory. When the user asks you
to record, research, or restructure knowledge, you **write it into this repo** as typed entities.

> **Scope of this file.** This CLAUDE.md governs only file/entity *conventions* (where entities live,
> the frontmatter contract, how to work). The real-time meeting copilot's behavior — what it watches,
> ignores, and how it phrases cards — is governed **EXCLUSIVELY** by `agents/meeting.md` (its steering
> body is merged into the copilot prompt). Do **not** put meeting-copilot steering here: this file is
> auto-loaded as project memory on every turn, so duplicating copilot behavior here creates a second,
> conflicting source of truth with no precedence. Keep all copilot steering in `agents/meeting.md`.

## Entity layout (binding)

- One markdown file per entity at **`kg/entities/<type>/<slug>.md`** (e.g.
  `kg/entities/person/jane-liu.md`, `kg/entities/company/acme-corp.md`,
  `kg/entities/meeting/2026-06-24-acme-sync.md`).
- Every entity file **starts with YAML frontmatter** that MUST include these three fields, or the
  write is rejected and reverted:

  ```
  ---
  type: person          # the entity type (person | company | meeting | task | …)
  id: jane-liu          # a stable slug id, unique per type
  title: Jane Liu       # the human title
  ---
  ```

  You may add more frontmatter fields (role, company, tags, etc.) and a markdown body below the
  second `---`. Cross-reference other entities with `[[wikilinks]]` using their title.

## `kg/` is an Open Knowledge Format bundle (OKF v0.1)

The knowledge graph follows the [Open Knowledge Format](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf):
plain markdown + YAML frontmatter, portable across tools. Our three required fields are a strict
superset of OKF (which requires only `type`), so the bundle stays conformant. Beyond them, prefer
OKF's recommended keys when you know the value:

- `description:` — one-line summary of the entity.
- `resource:` — URI of the external system-of-record (LinkedIn profile, GitHub repo, project page).
- `tags:` — list of categorization strings.
- `timestamp:` — ISO 8601 time you last updated the knowledge in the file.

**Reserved files** (no frontmatter, not entities):

- `index.md` — one per directory under `kg/`, a short listing of what's inside with relative
  markdown links (progressive disclosure for readers/agents). When you add or remove an entity,
  update the `index.md` of its type directory (and create the directory's `index.md` when you
  create a new type directory).
- `log.md` — optional chronological change history, newest first, grouped by ISO 8601 date.

Bodies are normal markdown. `[[wikilinks]]` remain the primary cross-reference (an extension OKF
consumers tolerate); use standard relative markdown links in `index.md` files and wherever a
portable link helps.

## Interface components (optional, render-rich)

The terminal renders entity bodies as MDX with a **closed component registry**. You may use these
tags in any entity body to make the doc an interface; everywhere else (git, plain editors) they
degrade to readable markup. Unknown tags or malformed MDX fall back to plain-markdown rendering —
never invent tag names outside this list:

- `<Note>…</Note>` / `<Warning>…</Warning>` — callouts for context or risks.
- `<Card title="…" icon="…" href="kg/…">…</Card>` inside `<CardGroup cols={2}>` — clickable
  navigation cards; `href` to another entity doc opens it in-app. Icons: `user`, `building`,
  `cal`, `tasks`, `file`, `folder`, `link`, `zap`, `spark`.
- `<Steps><Step title="…">…</Step></Steps>` — numbered plans/processes.
- `<Tabs><Tab title="…">…</Tab></Tabs>` — alternative views (e.g. Background / History).

Use them where structure helps a human scanning the doc (plans → Steps, related entities →
CardGroup, caveats → Warning); don't decorate for its own sake. `[[wikilinks]]` render as rich
entity chips automatically — keep using them inline as the primary cross-reference.

## How to work

- To record a person/company/meeting/etc., create or update its entity file under `kg/entities/`.
- For recurring or scheduled work, use the **scheduling** skill.
- Keep facts dated and attributed where it helps. Do not invent — only record what you were given or
  found.
- You do **not** run git — commits and history happen outside your turn. Just write the files.
- Confirm briefly in your reply what you wrote (e.g. "Created `[[Jane Liu]]`").
