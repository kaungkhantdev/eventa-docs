# AGENTS.md

Instructions for AI coding agents in this repo. `CLAUDE.md` imports it; only Claude-specific lines go there.

- **Trust order.** The code repos and the live database say what is built, so they win over every
  document here that describes it (such as `erd.md`, `flowcharts/`, `how-services-connect.md`);
  this repo's backlog, NFRs and DevOps docs say what should be built. Both outrank this file:
  when they disagree with it, follow them and fix this file in the same change.
- **Don't guess.** Look up a name before you use it: file, script, env var, endpoint, table, story
  ID. If you can't find it, say so instead of inventing it.
- **Counts and lists go stale.** This file has none on purpose; run the command it gives instead.

## What this is

The SDLC documentation for Eventa, an event registration and management platform: one folder per
lifecycle stage, `01-requirements-and-features/` to `08-maintenance/`. Plain Markdown and Mermaid;
no code, build, lint, test or CI.

## Commands

From the repo root. Measure a count; never copy it from another document.

```bash
grep -cE '^### US-' 01-requirements-and-features/functional-requirements.md      # stories
grep -cE '^### NFR-' 01-requirements-and-features/non-functional-requirements.md # NFRs
grep -cE '^### TC-' 06-testing/test-cases.md                                      # test cases
grep -cE '^#### `[a-z_]+`' 04-architecture/entities.md                            # catalog tables
# Live tables; `docker compose up` in ../eventa migrates first (../eventa-api's doesn't)
docker exec eventa-postgres-1 psql -U eventa -d eventa -At -c "SELECT table_name FROM
  information_schema.tables WHERE table_schema='public' AND table_type='BASE TABLE' ORDER BY 1;"
# Every story has a test case (no output means yes)
diff <(grep -oE '^### US-[A-Z]+-[0-9]+' 01-requirements-and-features/functional-requirements.md |
  cut -c5- | sort) <(grep '^- \*\*Traces' 06-testing/test-cases.md |
  grep -oE 'US-[A-Z]+-[0-9]+' | sort -u)
find . -name '*.md' -not -path './.git/*'      # the doc inventory; README lists are hand-kept
```

## Rules

1. **Never renumber an id.** The code repos cite `US-*`, `TC-*`, `NFR-*`, `ADR-*` and epic numbers.
   Insert a case with the next free suffix: `TC-<AREA>-<n>b`, then `c`. Resolve an epic by AREA
   code (`E6` holds `US-DISC-*`).
2. **The live schema is the data model.** `erd.md` draws it; `entities.md` lists every column and
   key and says why. Neither derives from the other; a shipped table needs both. Unbuilt design
   goes in a "Designed, not built" note, never in a column table.
3. **Flowcharts draw what the code does today,** checked against `../eventa-api`, `../eventa-web`
   and `../eventa-worker`. Mark a gap with a `Not built:` or `API-only:` node.
4. **Keep the backlog business-observable:** no endpoints, tables or code names in stories or NFRs.
   Write as the header table's **Author** role, if the doc has one.
5. **Change a story and its trace together:** its `TC-*` cases (the `- **Traces:**` line), the
   story map (run its §5 check) and, for a release exception, `project-plan.md` §4.1.
6. **Update every copy of a count** (doc headers, stage and root `README.md`s, `entities.md` and
   `erd.md` prose): `grep -rn '<old number>' --include='*.md' .`
7. **Moving a doc or renumbering a section:** fix inbound links and both `README.md` indexes, then
   grep `../eventa*`; siblings link here by path, section number and GitHub URL.
8. **Commit subject:** `docs(<scope>): <change> (US-…)`, naming every story touched.

## Gotchas

- **Readers see a PNG, not the Mermaid.** `flowcharts/*.md` and `erd.md` show a PNG and fold the
  Mermaid in `<details>`. After editing a fence, re-render the PNG with a setup that first
  reproduces the committed one byte for byte. Renderer (Chrome, mermaid-cli), Mermaid version,
  width and scale differ by image; `git log -- <png>` may name them.
  `08-maintenance/dead-letter-handling/README.md` has a headless-Chrome harness.
- **Never hand-patch `erd/eventa-full.drawio`** (`erd.md` says why); no repo can regenerate it.
  Edit `.tldr` canvases in tldraw, or only their labels in the JSON. In the commit body, name each
  derived file you left stale.
- **No Mermaid checker.** Check the diagram header and that `erDiagram` braces balance.
- **Id regexes undercount:** `TC-*` ids take letter suffixes; `NFR-L10N-*` has digits.
- **`entities.md` defeats naive greps:** bullets come singular and plural (`**INDEX**` /
  `**INDEXES**`), headings can carry a suffix (`` #### `memberships` — JUNCTION ``), and partial
  unique indexes are in `pg_index`, not `pg_constraint`. Read the entry before reporting a gap.
- **A story id in code is a pointer, not proof.** Code can serve a story it does not cite. Follow
  the code before writing "not built".
- **"Prototype" in older docs means `../eventa-web`,** now the shipped app wired to the API.

## Where things are

- `README.md`: stage table and index. Read it first.
- `04-architecture/how-services-connect.md`: how the services meet and who writes which tables;
  `software-architecture.md` is the formal SAD.
- `04-architecture/erd.md` "How to read this ERD": crow's-foot conventions.
- `04-architecture/flowcharts/README.md`: journey index and the journeys not built.
- `05-development/development-guide.md`: code-repo layout, setup and house rules (Appendix A).
- DevOps spans `07-deployment/` (build and ship) and `08-maintenance/` (operate).

## Sibling repos

Checked out beside this one (`ls ..`). `../eventa` holds the compose stack; its README says what
every repo but `../eventa-infra` is.

- The code repos build and test to a story's acceptance criteria; rewording one changes their scope.
- `../eventa-api` owns the schema (`src/db/schema/`) and every migration (`src/db/migrations/`).
- Take screens, routes and UI copy from `../eventa-web`, not its design kit `../eventa-ui-kit`;
  `design-reference.md` only indexes them.

## Editing this file

Add a line only for something an agent got wrong that the code could not have told it. No counts,
inventories, version lists or history of past mistakes: point to the file or command instead. Keep
it under 900 words.
