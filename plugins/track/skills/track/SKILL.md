---
name: track
description: Use when updating a tracked project after something happened — a decision was made, work progressed, or new material came in. Records the change into the project's snapshot, decision history, or references. Run /track-init first if no tracking root exists.
---

# Track

Update a tracked project's state from what just happened.

## 1. Locate the root

Two steps, **in this order**:

1. **Walk up from cwd** looking for `TRACKING.md`. First one found wins. Stop at `$HOME` or `/`.
2. **Fall back to the pointer** in the user's global `CLAUDE.md`:
   ```
   ## Project Tracking
   - Root: <absolute path>
   ```
   If several roots are listed, use the one containing the named project. If more than one does, ask.

Nearest wins, which is why the walk comes first: when you're inside a repo that has its own root, that root beats a central one.

**Step 2 is not optional.** Walking up only finds roots that are *ancestors* of cwd. A central root like `~/tracking` is a sibling of wherever you happen to be working — the walk will never see it. Skipping the fallback means reporting "no root found" while the root sits right there, and the user then builds a second one.

If neither step finds a root: tell the user to run `/track-init`. **Never guess a location and never create a root implicitly** — a root in the wrong place silently splits a project's history in two.

If the root is outside the current working directory, reading it may need the user's approval. Ask for it; do not treat a denied read as "no root."

Read `TRACKING.md` and honor its config in everything you write:
- `link_style: relative` → `refs/2026-07-20-foo.md`
- `link_style: wikilink` → `[[2026-07-20-foo]]`
- `date_format` → filenames and frontmatter
- `naming_language` → generated file and directory names

## 2. Locate the project

Look for `<root>/<project>/`. If the user didn't name one, or it doesn't exist, list the projects from `README.md` and ask. Do not create a new project directory without confirming — a typo must not become a second project.

## 3. Read the snapshot

Read `<root>/<project>/snapshot.md` and nothing else.

Do **not** read `history/` or `refs/` by reflex. `history/` is entered in exactly two cases:

1. The user asks why something was decided.
2. You need to find a `supersedes:` target, or resolve a constraint-ID reverse lookup.

`refs/` is entered only by following a link from the snapshot.

Default-reading them defeats the point of distilling material on the way in — the snapshot already carries the conclusions, and re-reading the sources is how this skill would end up re-ingesting its own output.

## 4. Sort what's new into three buckets

Go through what happened since the snapshot was last updated. Each item lands in exactly one bucket:

| Input | Bucket |
|---|---|
| **An option was given up** — including choosing one of several for the first time | Decision → section 5 |
| **Factual progress** — the world advanced, no choice was made | Snapshot only → section 6 |
| **External material** — someone else's document, data, or constraint | Reference → section 7 |

The threshold for a decision is **"was an option given up?"** — not "was this important?" A first-time choice qualifies: picking one of three FA firms gives up the other two.

Judge by these examples, not by your sense of significance:

| Decision (write history) | Not a decision (snapshot only) |
|---|---|
| "Picked FA firm X out of three" — gave up two | "They replied, meeting next week" — nothing chosen |
| "Equity round → convertible note" — gave up the equity path | "PCB samples arrived" — the world moved |
| "Shelved the Europe market" — gave up an option | "Term sheet signed" — signing was already decided |

Two failure modes to avoid, in both directions:
- **Logging everything.** If every `/track` run produces a history entry, the threshold is being ignored and history is now a changelog. ~10-20 entries per half-year is the calibration.
- **Logging nothing.** A first-time choice feels like "just progress" and gets waved through. It isn't — the reasons for the original choice are exactly what gets asked about later.

When genuinely unsure whether something gave up an option, **ask the user** rather than guessing. A wrong call is expensive in both directions and they know the answer in one word.

## 5. Bucket: a decision → write a history entry

Create `<root>/<project>/history/<date>-<verb>-<object>.md`, where `<date>` is when the decision was **made** (see below). Verb first — a decision is an act. Multiple entries on one day get a `-2` suffix.

```markdown
---
date: <YYYY-MM-DD>        # when the decision was MADE
logged: <YYYY-MM-DD>      # when it was recorded — omit if same as date
supersedes: <filename of the entry this overturns>   # omit if none
constraints: [C1, C3]                                # IDs this decision rests on
---
# <verb-first title>

**Was:** <prior position — "undecided" for a first-time choice>
**Now:** <new position>
**Why:** [C1] <reason> → <link to the ref, per link_style>
**Gave up:** <the option abandoned, and what would have to change for it to revive>
**Affects:** <which snapshot section this changes>
```

### `date` is when it was decided, not when you ran `/track`

Use today only when the decision happened in this conversation. **If the user says when — "last month," "back in January," "we decided this on the 15th" — that is the `date`,** and it drives the filename too. A run that files three decisions from different months must produce three different dates.

Set `logged:` when it differs from `date`, and omit it otherwise. It marks the entry as **reconstructed rather than contemporaneous** — which changes how much a future reader should trust it. Memory of *why* decays much faster than memory of *what*, so a `Gave up:` recorded six months late is a reconstruction, and the reader deserves to know.

**A contemporaneous note requires a contemporaneous entry.** Never attach one to a backfilled entry — you are not flagging a premise as it was made, you are second-guessing with hindsight the user did not have. If the reasoning looks wrong on a backfilled decision, say so in conversation instead.

Ask for the date when the user is clearly recounting the past but hasn't said when. Guessing produces a fake timeline, which is worse than no timeline — a wrong order makes the whole chain lie about cause.

Every field is mandatory except `supersedes`. **`Gave up` is the one that earns its keep** — it is what a future reader needs and what nobody remembers. If you cannot fill it in, this was not a decision; re-check section 4.

### If the stated reason doesn't hold up, record it anyway — and say so

When the user's `Why` contradicts a recorded constraint, or is internally inconsistent, **do not quietly fix it, argue them out of it, or refuse to file**. Write the decision exactly as they made it, then append:

```markdown
**Contemporaneous note — recorded as decided; the premise is flagged, not resolved.**
<what doesn't hold up, and why, in two or three sentences>
```

The decision is theirs and it is already made — your job is the record. But a reason that was shaky *at the time* is the single most valuable thing a future reader can find, and it is exactly what nobody remembers later. Months on, "someone said so at the time" is worth more than the outcome itself.

Keep this rare and specific. It fires when the reasoning **conflicts with something already written down** — not when you merely disagree, would have chosen differently, or think the odds are bad. A note on every entry means the bar is too low, and it trains the user to skip the notes entirely.

Ask instead of noting when you can — a question beats a footnote. But when they've said not to ask, or the conflict surfaces while you're writing, the note is how it survives.

Cite constraints by **ID** in `constraints:` and in `Why:`, not just by linking the ref file. Section 8 depends on those IDs being greppable.

Never edit an existing history entry to reflect a change of mind. A reversal is a **new entry** with `supersedes:` pointing at the old one.

## 6. Bucket: factual progress → snapshot only

Update the relevant snapshot section in place. Write no history entry.

Most `/track` runs are entirely this bucket. That is correct and healthy — it means the threshold is doing its job.

## 7. Bucket: material → write a ref

Create `<root>/<project>/refs/<date>-<slug>.md`.

**Distill on the way in.** The whole point is that the cost of making material usable is paid once per document — not once per document per decision. A 3-line note and a 50-page regulation both end up as one line of constraint in the snapshot.

```markdown
---
kind: ref
date: <YYYY-MM-DD>
source: <URL or local path>
---
## What this means for the project

- [C1] <actionable constraint or fact> (<where in the source>)
- [C2] <…>

## Excerpts

<only the passages the above rests on>
```

`## What this means for the project` is **mandatory and first**. If you cannot write it, the material has no actionable bearing on this project and does not belong in `refs/` — say so instead of filing it.

**Never copy large material wholesale.** Store the `source:` pointer plus the excerpts the judgment rests on. Full text is read by nobody and turns search into noise; the excerpts are the only part anyone revisits.

New constraints get the next free ID for the project (see section 8) and a line under the snapshot's `## Constraints`.

## 8. Did a constraint change? Reverse-look-up the decisions resting on it

A constraint changing is **the world moving, not a choice** — so by section 4 it writes no history entry. That leaves a gap: nothing would ever flag that a past decision's foundation is gone. This section is that flag.

**A cited constraint is immutable. Never edit its text in place.** History entries cite constraints by ID forever, so rewriting `C1` from "capped at 25%" to "capped at 49%" silently rewrites the stated reason of every decision resting on it — the old entry now reads "we gave up equity because ownership is capped at 49%," which is the exact opposite of what happened. A constraint that changes is **retired**, and its replacement gets a **new ID**.

When the snapshot's `## Constraints` gains a constraint, or one is retired and replaced:

1. Grep `history/` for the constraint's ID:
   ```bash
   grep -l "C1" history/*.md
   ```
2. For each hit, read its `Gave up:` line.
3. Surface them to the user before doing anything else:

   > `C1` changed (≤25% → ≤49%). 1 decision rests on it:
   > `2026-07-21-drop-equity-round-for-convertible.md` — gave up the equity round,
   > revives if the cap is raised. Reconsider?

4. If they reconsider and reverse, that reversal **is** a decision — section 5, with `supersedes:`.

Do not decide for them. Surface and ask; a relaxed constraint does not automatically mean the old choice was wrong.

**Constraint IDs are `C<n>`, monotonic per project, and never reused.** A retired ID must keep resolving, because history entries cite it forever — reusing `C1` would silently re-point an old decision's stated reason at an unrelated constraint. To find the next free ID, scan both `## Constraints` and every `constraints:` line in `history/`; the next ID is one past the highest ever used, not one past the current list.

Retired constraints leave `## Constraints` (which lists only what is in force) and move into the ref that superseded them, so the ID still resolves. The replacement's snapshot line should say what it replaced — "capped at 49%, raised from 25% on <date>" — so the chain is readable without opening the ref.

## 9. Update the snapshot and README

**The snapshot links only to decisions still in force.** This is what keeps it from decaying into a changelog — the failure mode that kills tracking systems.

When a position is overturned:
1. Its text **and its link** leave the snapshot together — not just the text.
2. The new history entry's `supersedes:` points at the old entry.
3. The old entry stays on disk, untouched, forever.

History self-links into chains; the snapshot only ever points at each chain's **head**. Snapshot length then stays roughly constant however long the project runs.

If an abandoned option might revive later, it moves to `## Options on the table` with its revival condition — not into a growing list of dead links.

Update `## Where things stand` on every run. It is the only screen most readers will ever see; a stale first paragraph makes the whole file lie.

Set `updated:` in the frontmatter from `date_format`.

Then **regenerate** the project's row in `<root>/README.md` from `## Where things stand`. Never maintain it by hand — one source of truth, no drift.

## 10. Report

Tell the user exactly what you wrote: which history entries, which refs, which snapshot sections changed. They need to be able to catch a misclassification while they still remember what happened.
