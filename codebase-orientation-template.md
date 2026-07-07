# Generate a Codebase Orientation Document

You are a staff engineer doing a rapid onboarding investigation of an unfamiliar
codebase. Investigate it using your tools and produce a single file,
`CODEBASE_ORIENTATION.md`, that gives a new developer an accurate, high-level mental
model of the system in one read.

The deliverable is a *map, not a manual*: breadth over depth, skimmable, and every claim
grounded in evidence you actually observed.

## Operating principles

1. **Outside-in, breadth-first.** Start from what the software does and work inward.
   Draw the whole map before going deep on any one area.
2. **Follow execution, not the file tree.** Understand the system along the path a real
   request/command takes.
3. **Nouns + verbs.** Identify the domain model (core entities/types = nouns) and the
   key flows (what happens to them = verbs). Together they are ~80% of understanding.
4. **Find the hot core.** Most files are glue/config/boilerplate. Use size and git churn
   to locate the handful of files that carry the real logic; prioritize those.
5. **Evidence or it didn't happen.** Every structural claim cites a real path; every
   flow step cites `file:line`. Never name a file, function, or symbol you have not
   actually opened or listed. Mark inferences `(inferred)` and put anything you couldn't
   verify under **Open questions** — do not guess in the body.
6. **Read-only.** You may run non-mutating recon (`ls`, `tree`, `git log`, `grep`/`rg`,
   `wc`) and, if a safe test command exists, run the test suite to confirm behavior.
   Do not modify files, and do not run destructive or networked commands. The only
   artifact you produce is `CODEBASE_ORIENTATION.md`.
7. **Timebox and scope.** Adapt depth to repo size; sample representative modules rather
   than reading everything. State explicitly what you did *not* explore.

## Investigation procedure

Work these steps in order. Each notes what to do and which output section it feeds.

**Step 0 — Map the terrain (always first).**
List the top 2–3 directory levels; read `README`/docs; open the dependency manifest;
check recent activity and churn to find hotspots. Detect language(s)/ecosystem from the
manifest and file extensions.
```bash
tree -L 2 -d  ||  find . -maxdepth 2 -type d
git log --oneline -20                                   # recent activity
git shortlog -sn | head                                 # ownership
git log --format= --name-only | sort | uniq -c | sort -rn | head   # churn = hotspots
find . -name '*.EXT' | xargs wc -l 2>/dev/null | sort -rn | head    # biggest = likely core
```

**Step 1 — Purpose & users.** From README/docs/description: what it does (one sentence),
who/what uses it, primary interface (CLI/library/HTTP API/UI/daemon/SDK), maturity.
→ feeds §1.

**Step 2 — Stack & architecture.** From manifest + tree: language/runtime, frameworks &
major libs (and their role), build/run/test commands, architecture style (single-file /
monolith / layered / feature-modules / microservices / plugin / event-driven). → §2, §3.

**Step 3 — Entry points.** Locate where execution begins and what triggers work.
```bash
rg -n "def main|if __name__|func main\(|public static void main|app\.(listen|run)|@(app|router)\.(get|post|route)"
```
→ §4.

**Step 4 — Core domain model (nouns).** Find the central entities/types in
`models`/`types`/`domain`/`entities`/schema/migrations or the main classes/structs.
Record what they are, where defined, and how they relate. → §5.

**Step 5 — Trace one key flow (verbs) — the centerpiece.** Pick the single most
representative operation and trace it end-to-end: entry point → logic → state → output.
Record each hop with `file:line`. This one trace teaches more than reading ten files. → §6.

**Step 6 — State & data.** Where persistent state lives (DB/files/cache/external), plus
config, secrets, and feature flags (env vars, config files). → §7.

**Step 7 — Boundaries & external dependencies.** What external systems/services it calls,
and what API surface this codebase itself exposes. → §8.

**Step 8 — Conventions, tests & quality.** How a typical feature is laid out; test
framework, location, and rough coverage; error-handling/logging approach; obvious tech
debt or big TODOs. → §9, §10.

**Step 9 — Synthesize & write.** Fill in the output structure below. Then re-read it as
if you were new: if a section is unverified or thin, either go check it or move the claim
to **Open questions**.

## Output: `CODEBASE_ORIENTATION.md`

Write exactly these sections. Prefer tables and tight bullets over prose. Link to files
(`path/to/file.py:42`) instead of quoting large blocks. Target ~1–2 pages of substance.

```markdown
# <Project Name> — Codebase Orientation
> _Generated <date>. Coverage: <what was investigated>. Not explored: <list>._

**TL;DR:** <one sentence: this is a ___ that ___, built with ___.>

## 1. What it is
- Purpose (one sentence):
- Users / consumers:
- Primary interface:
- Maturity & activity:

## 2. Stack & architecture
- Language(s) / runtime:
- Frameworks & major libraries (role of each):
- Architecture style:
- Build / run / test commands:

## 3. Directory map
| Path | What lives here |
|------|-----------------|
| ... | ... |

## 4. Entry points
| Trigger (route / command / event / job) | Location |
|---|---|
| ... | `file:line` |

## 5. Core domain model (the nouns)
| Concept | Defined in | Notes / relationships |
|---|---|---|
| ... | `file:line` | ... |

## 6. Key flow walkthrough (the verbs)
_Operation traced: <name>._
1. <entry> — `file:line`
2. <next hop> — `file:line`
3. ... → <result / persistence>

## 7. State & data
- Persistent state:
- Config / secrets / feature flags:

## 8. External dependencies & boundaries
- Calls out to:
- Exposes (public API surface):

## 9. Conventions & testing
- How a typical feature is structured:
- Testing (framework, location, coverage):
- Error handling & logging:

## 10. Gotchas, risks & tech debt
- <surprises, footguns, notable TODOs>

## 11. Start here
The 3–5 files to open first to understand this codebase, in order:
1. `path` — why
2. ...

## 12. Open questions / unverified
- <claims you couldn't confirm, areas skipped, things a human should check>
```

## Quality bar

- Every path and symbol referenced must be one you actually observed. No invented names.
- Tag inferences `(inferred)`; park anything unverified in §12 rather than the body.
- Keep it skimmable and short — a map, not documentation. Cut filler.
- Be explicit about coverage: what you investigated and what you skipped.

## Anti-patterns to avoid

- Reading files alphabetically/folder-by-folder instead of following execution.
- Trying to document everything instead of finding the hot core (80/20).
- Going deep on pass one — stay breadth-first; note questions and move on.
- Stating framework/behavior from prior knowledge without checking this repo's code.
- Padding sections with generic prose that isn't specific to this codebase.
