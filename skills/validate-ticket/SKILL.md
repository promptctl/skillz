---
name: validate-ticket
description: Make a ticket ready for implementation. Confirms the ticket states its goal and that its title, goal and acceptance criteria agree, rewrites acceptance criteria that describe implementation as the outcomes they stand for, and edits the ticket to fix whatever is missing or inconsistent, using related tickets where the ticket itself is unclear. Works from tickets only, never the code. Returns the corrected ticket. Use when the user says "validate this ticket", "clean up this ticket", "get this ticket ready", or before handing a ticket to an implementing agent.
argument-hint: "<ticket id>"
context: fork
---

# Validate a ticket

Ticket: `$ARGUMENTS`

Work from tickets only. Do not read the code.

If `$ARGUMENTS` names no ticket or the ticket cannot be read, say so and stop.

## 1. Goal

Read the ticket: title, description, comments. State its goal in one sentence, in
terms of whoever uses the result: what a person sees, what an agent gets back, what a
process can now do. "Add X" is a method, not a goal.

If the ticket doesn't make the goal clear, read its parent and the tickets that
depend on it. They show what this work is needed for.

The goal you write is the one those sources give. Restate it; do not broaden, narrow
or replace it.

If no ticket states or implies a goal, return the ticket unchanged and say the goal
is missing.

## 2. Acceptance criteria

Each criterion describes an outcome the user of the work can rely on once it is
done. A criterion that names an implementation detail (a function, a type, a file
layout, a sequence of steps) is rewritten as the outcome it stands for.

Taken together, the criteria cover the goal: work that met every criterion and still
missed the goal would mean a criterion is missing or wrong.

Criteria carried over from an earlier ticket are judged against this ticket's goal
like any other.

- BAD: "`load_config` returns a `ConfigResult` instead of `None`."
- BAD: "Output matches the current implementation on all existing fixtures."
- GOOD: "Starting the service with a missing config file exits non-zero and the error
  names the missing path."

If the ticket has no criteria, write them from the goal.

## 3. Consistency

The title names the goal. The description, goal and criteria describe the same piece
of work, and no two parts contradict.

## 4. Fix the ticket

Edit the ticket in the tracker so it has a title, a goal and acceptance criteria that
meet steps 1–3.

- Keep the author's content. Move misplaced text instead of deleting it.
- Add no design. The ticket states what is needed; the implementer chooses how.

## 5. Return

Return the ticket as it now stands:

```
Ticket: <id>
Title: <title>
Goal: <one sentence>
Acceptance criteria:
- <criterion>
Changed:
- <what changed, and why>
```
