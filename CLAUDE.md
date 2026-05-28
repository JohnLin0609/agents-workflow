# CLAUDE.md — Dev-Process Skills

A library of **skills** that codify a software development lifecycle. Each skill is one
**role** (a sub-agent): requirements analyst, architect, planner, implementer, reviewer,
QA, release engineer, SRE, retro facilitator. Skills collaborate by reading and writing
**Markdown artifacts** derived from shared **templates**. This file holds the rules that
apply to the *whole repo*; per-role knowledge lives inside each skill.

---

## Golden rules (always apply)

1. **English only.** All skill content — `SKILL.md`, spoke files, templates, scripts,
   comments — is written in English. (This file is English for the same reason.)
2. **One skill = one role.** A skill does its role's job and nothing else. If a skill
   starts doing a second role's work, that work belongs in another skill.
3. **Progressive disclosure is the law.** Keep what is always-loaded tiny; load detail
   only when needed. Budgets below.
4. **Templates are copied, never mutated.** Read from `templates/`, write filled copies
   into the work directory. The source template stays pristine.
5. **Artifacts are the interface between skills.** Skills hand off via files on disk, not
   by assuming shared memory. Declare inputs and outputs explicitly.
6. **Practice what we preach.** These skills teach good engineering; the repo itself must
   model it — small focused files, clear names, no dead content.

---

## The three loading tiers (and their budgets)

| Tier | What | When loaded | Budget |
|------|------|-------------|--------|
| **Frontmatter** | `name` + `description` | **Always** (every session) | Keep `description` lean: 1–3 sentences, must state the trigger |
| **SKILL.md body** | The role's core playbook | **On trigger** (description matches task) | **Target ~2k tokens.** Hard ceiling ~5k words / 500 lines (official) |
| **Spoke files** | `references/*.md` deep dives | **On demand** (body points to them) | Each focused; no fixed cap, but split rather than bloat |
| **Scripts** | `scripts/*` | **Executed, not loaded** | n/a — runs via Bash, never enters context |

**Rule of thumb:** if a SKILL.md body crosses ~2k tokens, move the heavy detail into a
spoke file under `references/` and leave a one-line pointer in the body
(e.g. *"For the full edge-case taxonomy see `references/edge-cases.md`."*).

---

## Repo layout

```
.
├── CLAUDE.md                      # this file — repo-wide rules
├── .claude/
│   └── skills/                    # the skills (one folder per role)
│       └── <role-name>/
│           ├── SKILL.md           # frontmatter + body (~2k tokens)
│           ├── references/        # spoke files, loaded on demand
│           ├── scripts/           # executed, not loaded
│           └── assets/            # optional binaries / fixtures
├── templates/                     # shared Markdown templates (copied, not edited)
│   ├── 01-requirements/ … 09-retrospectives/
└── work/                          # per-project artifacts (filled template copies)
    └── <project-name>/
        └── 01-requirements/ …     # outputs flow stage → stage here
```

---

## Writing a skill

**Frontmatter** — only `name` and `description` (optionally `allowed-tools`). The
`description` is the *only* thing the model sees before triggering, so it must earn its
place. Use this shape:

```yaml
---
name: requirements-analyst
description: >
  Use when starting a new feature or project and the problem/scope is not yet pinned down.
  Elicits the why, users, success metrics, constraints, and out-of-scope; produces a PRD
  artifact. Do NOT use for design or task breakdown.
---
```

A good `description` says **when to use**, **what it does**, **what it produces**, and —
when roles are adjacent — **when not to use** (to prevent the wrong skill firing).

**Body** — the playbook for this role. Keep it to the steps, the decisions, and the
quality bar. Aim ~2k tokens. Push examples, long checklists, and taxonomies into
`references/`.

**Spoke files** — one concern per file, named for what they hold
(`review-checklist.md`, not `notes.md`). The body links to them; they load only when the
task reaches that depth.

**Scripts** — deterministic work (validators, generators, linters) goes in `scripts/` and
is *run*, not read into context.

---

## Skills as collaborating sub-agents

Each skill declares its **handoff contract** in its body:

- **Inputs** — which upstream artifact(s) it reads (e.g. `work/<proj>/01-requirements/requirements.md`).
- **Outputs** — which artifact(s) it writes, and where.
- **Done when** — the gate the artifact must pass before the next role picks it up.

Stages flow through `work/<project>/` in numbered order. A downstream skill must not
invent missing upstream context — if its input artifact is absent or fails its gate, it
stops and says so rather than guessing. If a skill discovers an upstream artifact is
wrong, it updates that artifact (or flags it), keeping docs and reality in sync.

---

## Templates

- Templates are Markdown, version-controlled, and **read-only in spirit**.
- To use one: copy it into `work/<project>/…`, then fill the copy.
- Never edit a file under `templates/` to record project-specific content.
- Improving a *template itself* is a deliberate change (its own commit), not a side effect
  of doing project work.

---

## Naming

- Skill folders: `kebab-case`, named for the role (`code-reviewer`, `release-engineer`).
- Spoke files & templates: `kebab-case.md`, named for content.
- Artifacts in `work/`: keep the template's stage number and name so lineage is obvious.

---

## Don'ts

- Don't write skill content in any language other than English.
- Don't let a SKILL.md body sprawl — split to spoke files past ~2k tokens.
- Don't duplicate the same knowledge across skills; if two roles need it, make it a
  shared spoke file or template and point both at it.
- Don't mutate source templates to hold project data.
- Don't have a skill silently do another role's job, or fabricate upstream input.
- Don't put secrets, credentials, or real user data anywhere in the repo.
```
