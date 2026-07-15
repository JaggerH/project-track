# project-track

Claude Code skills for tracking a project's **live state** plus the **decision trail that produced it** — for projects with or without a repository.

## The problem

A project's real state lives in chat logs. Months later, *"what did we decide, and why?"* is unanswerable without re-reading everything. And the reasons are exactly what you need when the world changes and an old decision deserves another look.

Existing tools don't cover this. Code-project tooling tracks *engineering work*; a wiki tracks *durable knowledge*. Neither tracks a project's live state and the forks in the road that got it there — for a project that may have no code at all: a fundraise, a legal matter, a supplier selection, a hardware run.

**Goal:** anyone — you in six months, or an AI with no context — reads one file and can take over.

**Non-goal:** replacing your chat logs. This is a deliberately *lossy* record of forks in the road, not a transcript.

## Install

```
/plugin marketplace add JaggerH/project-track
/plugin install track@project-track
```

Then:

```
/track-init
```

It asks where the tracking root should live, creates it, and scaffolds your first project. It ends by offering to add a pointer to your `~/.claude/CLAUDE.md` so `/track` can find the root from anywhere — say no and it prints the snippet to paste yourself.

## The loop

After any conversation that moved a project forward:

```
/track <project>
```

That's the whole interface. The skill reads your snapshot, sorts what happened into three buckets, and writes:

| What happened | Where it goes |
|---|---|
| **An option was given up** — including choosing one of several for the first time | a `history/` entry |
| **Factual progress** — the world moved, nothing was chosen | the snapshot only |
| **External material** — a document, a regulation, a quote | a `refs/` entry, distilled on the way in |

Then it tells you exactly what it wrote, so you can catch a misclassification while you still remember what happened.

**It only writes when you ask.** It never records anything on its own.

## What it builds

```
<your root>/
├── TRACKING.md            marker + config (link style, date format, naming language)
├── README.md              project list, one line of current state each
└── <project>/
    ├── snapshot.md        current truth — the only file read by default
    ├── history/           decisions only, ~10-20 per half-year
    └── refs/              material, distilled to constraints on the way in
```

`snapshot.md` has a fixed skeleton:

```markdown
## Where things stand    one paragraph: current state + next step
## What's being tried    active directions, each linking the decision that chose it
## Constraints           hard limits, each with a stable ID and a link to its source
## Key dates             date + what decision comes due on it
## Options on the table  not yet tried, still live
## Background            what this is and why. near-static
```

Background sits last on purpose. You read a snapshot to learn *where things stand*; only a first-time reader needs the background, and they read the whole file anyway. Putting it on top taxes every returning reader with a screen of preamble.

## Why this beats a chat log

Two mechanisms, and they're the whole point.

### Reading is cheap because writing did the work

Distillation happens **at write time**, not read time. A 50-page regulation becomes one line of constraint in the snapshot, backed by a ref you open only if you doubt the distillation. The cost of making material usable is paid once per document — not once per document, per decision.

So the default read is **one file**. `history/` is opened only when you ask *why*, or when the skill needs it. That's what lets a cold AI take over from `snapshot.md` alone.

### Old decisions get re-opened when their foundation moves

Constraints carry stable IDs. Decisions cite them:

```markdown
## Constraints
- [C1] Foreign ownership capped at 25% → refs/2026-07-20-regulations.md
```

```markdown
constraints: [C1]
**Gave up:** The equity round. Revives only if the cap is raised.
```

When a constraint changes, the skill finds every decision resting on it and surfaces them:

> `C1` changed (≤25% → ≤49%). 1 decision rests on it: *drop-equity-round* — gave up the equity round, revives if the cap is raised. Reconsider?

A constraint moving is *the world changing*, not a choice — so it writes no history entry, and without this reverse lookup nothing would ever flag that a past decision's ground had shifted.

**Constraints are immutable once cited.** A change retires the old ID and mints a new one; editing `C1` in place from "25%" to "49%" would silently rewrite the stated reason of every decision citing it, so the old entry would read *"we gave up equity because the cap is 49%"* — the inverse of what happened.

### The snapshot doesn't rot

It links **only to decisions still in force**. When a position is overturned, its text and its link leave together; the new entry's `supersedes:` points at the old one, which stays on disk untouched forever.

```
back-to-equity ──supersedes──▶ bank-debt ──supersedes──▶ convertible-note
      ▲
   snapshot points only here
```

History self-links into chains; the snapshot only ever points at each chain's head. Its length stays roughly constant however long the project runs — instead of degrading into the changelog that kills most tracking systems.

## Honest limitations

**Forget to run it and it's gone.** The manual trigger is deliberate — a skill shouldn't write to your files unprompted — but the ceiling on this tool is your habit.

**It only knows what you tell it.** Say *"we chose X"* without saying *"we dropped Y"* and the `Gave up:` line will be thin — and that line is the one that matters in six months.

**Built for recording as you go.** Backfilling works (`date` is when you decided, `logged` marks it as reconstructed) but memory of *why* decays far faster than memory of *what*. A reason written six months late is a reconstruction.

## Configuration

`TRACKING.md` at your root holds everything environment-specific, so the skill itself hardcodes no paths:

| Setting | Default | Notes |
|---|---|---|
| `link_style` | `relative` | or `wikilink` (`[[…]]`) for Obsidian and similar |
| `date_format` | `YYYY-MM-DD` | filenames and frontmatter |
| `naming_language` | `en` | generated file and directory names |

`/track` finds your root by walking up from the working directory, then falling back to the `## Project Tracking` pointer in your global `CLAUDE.md`. Both topologies work: one central root for everything, or a root inside each repo — nearest wins.

## License

MIT
