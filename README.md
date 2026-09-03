# research-tasks

Version-controlled **Claude Cowork tasks** for Syracuse University research administration, maintained by Duncan Brown (VP for Research; Syracuse University OSPO). Licensed under Apache-2.0.

## What this repository is

A Cowork *task* is a saved set of instructions that Claude Cowork runs — often on a schedule (e.g., a weekly refresh) — against connected folders, the Chrome connector, and other tools. This repository treats those instruction files as open-source software: each is versioned, licensed, has a named maintainer and a review cadence, and records its own provenance.

### Tasks vs. skills (why this repo is separate from `research-skills`)

- **Skills** (`duncan-brown/research-skills`) are agent *capabilities*, invoked by description-matching and distributed as packaged `.skill` files. Their source lives in `SKILL.md`.
- **Tasks** (this repository) are Cowork *automations*, pasted into a Cowork scheduled task's Instructions field and run by the agent directly. Their source lives in `TASK.md`.

They are functionally different and distributed differently, so they are kept in separate repositories.

## Repository layout

```
research-tasks/
├── README.md                    # this file
├── LICENSE                      # Apache-2.0 (applies to all tasks)
└── <task-name>/                 # one directory per task (kebab-case)
    ├── TASK.md                  # the versioned task: metadata header + runnable body
    └── (optional) assets/, scripts/, references/
```

## Anatomy of a TASK.md

Every `TASK.md` has two parts, separated by a horizontal rule (`---`):

1. **Metadata header** — YAML frontmatter (`name`, `description`, `version`, `license`, `maintainer`, `created`, `updated`, `repository`) followed by: a **Version Information** table, a **Version History** (newest first), a **Maintenance — parameters that expire** section, **Versioning Instructions**, a **GitHub Repository Update Workflow**, and a **License** section.
2. **Operational body** — everything below the `---`. This is the exact text pasted into the Cowork task's Instructions field, so it must be self-contained and runnable on its own without the metadata above it.

## Conventions for maintainers (human or LLM)

- **Versioning:** `MAJOR.MINOR`. Auto-increment the minor on routine edits; a MAJOR bump (architecture or scope change) requires explicit maintainer instruction. On every change, update the `updated:` field and the "Last Updated" row, and add a Version History entry.
- **Commit messages:** `<task-name> v<X.Y>: <summary>`.
- **Tags:** `<task-name>-v<X.Y>` — the `{name}-v{version}` form prevents tag collisions when multiple tasks share one repository. Create tags on a machine with push access (see "Pushing").
- **Privacy scan before every commit:** grep the diff for personally identifying information (real names beyond the named maintainer, salaries, IDs, third-party contacts). Task files should contain none by design.
- **Encode-expiring-rules discipline:** Tasks that hard-code dates, rates, fiscal years, dashboard paths, or sponsor rules *degrade silently* — they keep producing confident, well-formatted output against stale rules rather than failing loudly. Every such task MUST include a **Maintenance — parameters that expire** section that names exactly which values go stale and when, a **review cadence**, and a **named maintainer**. Review on that cadence, not only when something breaks. (Framing: Lorena A. Barba, "Skills are infrastructure, and a new open-source artifact class," GW OSPO, 2026.)

## Pushing (note for an LLM working inside Cowork)

The Cowork sandbox can run local `git` in a cloned working tree but **cannot reach GitHub over the network** — there is no DNS/egress to `github.com` and the maintainer's SSH key is not present. Use the **GitHub connector (MCP)** to read and write this repository, or hand the commit to the maintainer to push from their own machine. Do not attempt to work around the network restriction.

## Tasks in this repository

| Task | Version | Schedule | Maintainer | Summary |
|---|---|---|---|---|
| [`datainsights-dump-refresh`](datainsights-dump-refresh/TASK.md) | 1.0 | Mondays ~6am (local machine) | Duncan Brown | Refreshes six SU DataInsights exports into the shared DataInsights Dump folder via a dedicated Chrome profile, with row-count verification and a delete-free fail-safe. |

## License

Apache License 2.0. See [`LICENSE`](LICENSE). The license travels with any fork or derivative task.
