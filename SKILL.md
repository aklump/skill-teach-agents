---
name: teach-agents
description: Only use when the user explicitly invokes this skill by name (e.g. "/teach-agents", "teach agents what we learned this session"). Reviews the current session for durable, non-obvious nuances and proposes AGENTS.md updates so future agents don't rediscover them.
---

# Capture Session Learnings into AGENTS.md

The goal: every session that fights through a real bug, a tooling quirk, or a
non-obvious architectural constraint should leave that knowledge behind for
the next agent, instead of making them rediscover it. This skill runs at the
end of (or during) a session, over the conversation you already have full
context of — do NOT delegate this to a subagent via the Agent tool, since a
fresh agent has none of this session's context and would have nothing to
review.

## Process

### 1. Mine the session for candidate nuances

Review the conversation from the start. Look specifically for things that
were **surprising, costly, or non-obvious from reading the code alone**:

- A bug whose root cause wasn't where it looked (wrong layer, wrong
  assumption, a library/runtime limitation rather than a logic error).
- Something that took more than one attempt to get right, or that you had to
  verify empirically (a script, a repro, a real render) rather than reason
  out from the source.
- A tool/library/framework limitation discovered the hard way (e.g. "mpdf
  ignores `transform`", "this test runner needs X installed first").
- A convention, seam, or existing mechanism in the codebase that turned out
  to be the "right place" for a fix, especially if you almost put the fix
  somewhere else first.
- A class of bug that could recur elsewhere (e.g. "never key rows by column
  label — labels aren't unique") rather than a one-off typo fix.
- Anything the user corrected you on that reflects a standing project
  preference, not just a one-time instruction.

Skip anything that's: obvious from reading the code, specific to this one
session's throwaway data/numbers, already documented, or a one-off
preference that doesn't generalize.

If nothing from the session clears this bar, say so plainly and stop — don't
force an update to justify running the skill.

### 2. Route each nugget to the right file

- Root `AGENTS.md`: cross-cutting conventions that apply repo-wide (style,
  test tooling, workflow).
- A subsystem `AGENTS.md` next to the code it governs (e.g.
  `web/modules/custom/tsb/src/Report/AGENTS.md`): domain/architecture
  knowledge scoped to that area.
- If the relevant subsystem doesn't have an `AGENTS.md` yet, propose
  creating one, plus a one-line pointer from the root `AGENTS.md`'s
  "Subsystem docs" section (follow the pattern already there).
- **Global vs. repo-scoped — ask per nugget, every time**: for every
  candidate nugget, judge whether it's inherently tied to this repo (its own
  classes, conventions, domain logic) or would apply unchanged in any other
  project — a general tool/library gotcha, an authoring-syntax pitfall (e.g.
  a Mermaid rendering quirk), or a generic engineering heuristic. Whenever a
  nugget looks like it could be either, ask the user to choose, in spirit:
  **this repo only, global only, or both** — don't silently default to the
  current repo, and don't infer the answer from a past session. The
  underlying question for the user: is this one nugget worth remembering for
  every project, or just this one? This is a per-nugget judgment call, not a
  standing setting — ask it fresh each time a nugget qualifies, even within
  the same skill run; never cache or reuse a prior "always global"/"always
  local" answer across nuggets or across sessions.
  - The global destination itself has no ratified, vendor-neutral standard
    location yet. Before creating a global file for the first time, check
    whether one already exists (`Bash(ls -la ~/.claude/CLAUDE.md
    ~/.claude/AGENTS.md ~/.config/agents/AGENTS.md)`); if one does, use it
    as-is and skip the question below — even if it's not at the
    "recommended" path. That path was a deliberate choice made once; don't
    suggest migrating it, on this run or any future one. This skill's job
    is capturing the current nugget, not auditing global config layout —
    if the user wants to migrate later, that's a separate, explicit ask.
    Otherwise (no global file exists anywhere yet) present these options
    and ask the user to pick (as of this writing):
    1. `~/.claude/CLAUDE.md` **(recommended)** — Claude Code's own
       documented, confirmed-auto-loaded global memory file. Not named
       `AGENTS.md`, but it's the only one of the three actually guaranteed
       to be read by this harness today.
    2. `~/.claude/AGENTS.md` — matches this skill's `AGENTS.md` naming
       convention, but auto-loading at the user level is unconfirmed.
    3. `~/.config/agents/AGENTS.md` — bets on an independent, unmerged
       cross-vendor proposal (as of this writing not adopted by any tool
       natively, including this one).
    Once the user picks and a global file exists, reuse that same path for
    subsequent global nuggets rather than asking about the path again —
    only the per-nugget repo/global/both routing decision (above) is asked
    every time, not the path.
  - **Whenever a nugget is written to the global file** (global-only or
    both), also ensure the current repo's root `AGENTS.md` carries a short
    pointer to it (e.g. a "Global notes" section near the top) — something
    like: "`<global path>` holds cross-project conventions; read it too."
    This is mandatory, not optional: auto-loading of a user-level file
    isn't guaranteed by every harness, so the pointer is what actually
    makes the global file discoverable from within a normal project
    session. If the repo's root `AGENTS.md` already has such a pointer,
    don't duplicate it — confirm it's present and move on.

Use `Bash(ls -La ...)` / `Glob` to find existing `AGENTS.md` files in the
repo before deciding whether one already covers the area — don't assume the
root file is the only one.

### 3. Draft the additions

Match the existing tone and density exactly: terse, factual, causal (explain
*why*, not just *what*), no marketing language, no restating what the code
already makes obvious by being well-named. Look at how nearby sections are
written (e.g. `web/modules/custom/tsb/src/Report/AGENTS.md`'s "mpdf's CSS
subset" section) and write in that voice — short paragraphs or tight bullet
lists, concrete over abstract, cite the specific class/method/file the
nuance is anchored to.

If a nugget updates or contradicts something already written, edit that
passage in place rather than appending a second, conflicting note.

Keep it short. One paragraph or a few bullets per nugget. This file is
meant to be read, not skimmed past.

### 4. Present and confirm before writing

Show the user the proposed addition(s) per file — the actual markdown text,
not a vague summary — and ask for confirmation before editing. Do not edit
`AGENTS.md` files silently. If the user wants changes, revise and re-show.

### 5. Write, don't commit

Once confirmed, write the edits with the Edit tool. Do not create a git
commit unless the user separately asks — this skill's job ends at "the file
now says the right thing," matching this repo's general commit policy.

## Anti-patterns to avoid

- Padding out the file with things any competent agent would infer from
  reading the code — that's noise that dilutes the genuinely useful nuances
  already there.
- Writing a session narrative ("we tried X, then Y, then discovered Z") —
  write the conclusion as a standing fact, not a story of how you got there.
- One giant root `AGENTS.md` — prefer a subsystem file next to the code, per
  the existing split.
- Capturing task-specific detail (ticket numbers, specific PR numbers,
  specific report names) instead of the generalizable rule behind it.
