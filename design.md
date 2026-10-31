# This fork — superpowers worked through the todo queue

A fork of [obra/superpowers](https://github.com/obra/superpowers) (MIT, Jesse Vincent). It changes
one thing: **where the work is written down.** Upstream writes specs and plans as markdown files in
the repo and tracks tasks in the harness's TodoWrite list. This fork puts the plan in the
[claude-todo](https://github.com/moatingodseye/claude-todo) queue, which hooks enforce and which
survives compaction, and puts the spec in the repo's own design document.

Everything else — the skills' judgement, TDD, review loops, the scripts — is upstream's, unchanged.

## Rules for the diff

- **The plugin is still called `superpowers`.** Skills name each other `superpowers:<skill>` about
  50 times (49 in 7.0.1); renaming would make every one of those lines conflict on every merge.
- **Change only the lines that name a file or a todo.** No rewording, no restyling. Every changed
  line is a merge conflict waiting to happen; the smaller the diff, the cheaper upstream stays.
- **Nothing private goes in.** The fork is public: no host names, addresses, employer, or work
  email. Commits are `rbeeston <dev@moatingodseye.co.uk>`.
- **The scripts are not touched.** They take a plan file; the fork gives them one (below).

## The mapping

| upstream | this fork |
|---|---|
| spec at `docs/superpowers/specs/DATE-topic-design.md` (brainstorming) | the repo's design document — `design.md`, or the `design/` folder where the repo has one — committed |
| plan at `docs/superpowers/plans/DATE-feature.md` (writing-plans) | a queue **heading with parts**: `promote --group` / `add --under`. The heading's text holds the plan's Goal, Architecture, Spec pointer and Global Constraints; each part is one Task, its full text (files, steps, tests) as the part's text |
| a TodoWrite todo per task (SDD, executing-plans, using-superpowers) | the parts themselves; `next` starts one, `done` finishes it |
| `Task N: complete` ledger line | `done <part> "<commit, test command and result>"` |
| `Ruling:` and `minor (deferred)` ledger lines | still written to the ledger as upstream does, and at Finish also posted with `comment <heading>` so they outlive the workspace |
| `PLAN_FILE` given to `sdd-workspace`, `task-brief`, `review-package`, `task-start`, `task-done` | a file the queue writes on demand (`todo plan`, below) into `.superpowers/sdd/`, git-ignored, never edited by hand, rewritten whenever it is needed |

The ledger (`.superpowers/sdd/<plan>/progress.md`) stays. It is git-ignored scratch, deleted at the
end, and threaded through ~40 lines of two skills; replacing it would be the largest diff in the
fork for little gain, since the queue already holds what must outlive it.

## What claude-todo needs for this

1. **`todo plan <heading> [<file>]`** — prints a heading and its parts as a plan in the shape the
   scripts parse (`# <heading title>`, the heading text, then `## Task N: <part title>` and the
   part text for each part, numbered by part). With a file, writes it there. This is what turns the
   queue into a `PLAN_FILE` without the skills hand-copying text.
2. **A part starting hands over its heading's text too.** Today `next` on part #44.2 prints #44.2's
   text; the plan's Goal and Global Constraints live on #44 and are not shown. They must be.

Both go in claude-todo's own design and queue; this fork depends on a release that has them.

## Skills changed

| skill | change |
|---|---|
| brainstorming | "Where it goes" names the repo's design document |
| writing-plans | "Save plans to" and the two hand-over sentences name the queue heading |
| executing-plans | todo lines → queue parts; plan file from `todo plan`; Finish posts rulings to the heading |
| subagent-driven-development | the same, for its setup, flowchart labels and Finish |
| using-superpowers | "create a todo per item" for a skill's checklist stays a harness todo: a checklist is how one step is done, not queued work. Plus **Testing and Review Rhythm**, below, which starts with sizing the work |
| writing-plans | plus **Test and Review Points**, below |

## Testing and Review Rhythm — RB, 2026-10-25

RB: *"ensure todo-powers tells agents to do change tests, then regression later and to do regular
reviews"*. Upstream tests and reviews inside a plan's execution only; a session working the queue
task by task, with no plan, never meets those rules. So `using-superpowers`, which every session loads
at start, gains one section of three rules: test each change against what it can affect; run the full
suite once, at the end of a heading, before a push or deploy, or at session end; review the diff
every heading, or every five finished tasks or before a deploy, whichever is first.

It is one of the two places this fork adds wording rather than renaming a file or a todo (the other
is below). Each is a new section, not edited upstream lines, so a merge from upstream conflicts only
if upstream rewrites the lines around it.

## Size the work first; a plan places its test and review points — RB, 2026-10-25

RB on todo #700: *"that is a todo-powers design failure it should have broken down into nice steps
and planned when to test when to review etc"*. The evidence was penates #666 on 2026-10-24: one task
(the Payment and Pot editors) grew for 45 minutes with no `done` — two sheets, a new stored field, a
deploy and a Windows build — from 19% to 32% of context, and the compactor, which runs only after a
`done`, never got its turn. Nothing told the session to stop and plan before starting.

- **`using-superpowers`, Testing and Review Rhythm, gains a first rule: size it before starting.**
  A task that will need more than one commit, or touches more than one area, is planned first with
  `writing-plans` — a queue heading with parts — and the parts are worked one `done` at a time.
- **`writing-plans` gains a section, Test and Review Points.** A plan names, as parts of their own,
  where the full regression suite runs and where the diff is reviewed: at least one of each, the
  review last before the heading closes, and one more after every five parts. Each task's own tests
  stay in the task's steps, as upstream has them.

claude-todo does not enforce task size (RB, #700): it is this fork's job to make the work small.

## Installing it

Uninstall the official-marketplace `superpowers`, then add this repo as a marketplace
(`/plugin marketplace add moatingodseye/superpowers`) and install `superpowers` from it. The
marketplace file is upstream's and is not edited. Updates come from this fork, never from upstream
directly.

## Keeping up with upstream

`upstream` is a git remote on `obra/superpowers`, push disabled. To see what upstream has gained:

    git fetch upstream
    git log --oneline main..upstream/main

GitHub's fork page shows the same count. Each upstream release is read and either merged whole
(`git merge upstream/main`, conflicts resolved in favour of the mapping above) or skipped with a
reason. The check is run by hand or from the end-of-day routine; nothing is scheduled.
