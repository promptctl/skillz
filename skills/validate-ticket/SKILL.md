---
name: validate-ticket
description: Bring a ticket into workable shape before an agent starts on it. Works out what the ticket is for, checks that it has a purpose, a scope and outcome-based acceptance criteria that agree with each other, and edits the ticket to fill or correct whatever is missing, using the ticket's parent, related tickets and the code as sources. Returns READY, UPDATED with the changes, INCOMPLETE with what no source could supply, or COULD NOT CHECK. Use when the user says "validate this ticket", "check the ticket before we start", "clean up this ticket", "is this ticket ready", or before handing a ticket to an implementing agent.
argument-hint: "<ticket id>"
context: fork
---

# Validate a ticket

Ticket: `$ARGUMENTS`

If `$ARGUMENTS` names no ticket or the ticket cannot be read, return
`COULD NOT CHECK <reason>`.

## 1. Read

Read the ticket in full: title, description, plan, comments. Then read what surrounds
it: its parent, the tickets it links to, and any request or bug report it quotes.
These are the sources for every fix in step 3.

## 2. Check

A workable ticket has:

- **Title:** names the outcome, not the method.
- **Purpose:** why the work is needed, in terms of whoever uses the result: what a
  person sees, what an agent gets back, what a process can now do. "Add X" is a
  method, not a purpose.
- **Scope:** what changes, and where the work stops.
- **Acceptance criteria:** conditions that tell done from not done. Each one describes
  an outcome the user of the work can rely on, not an implementation detail (a
  function, type, file layout or sequence of steps). The set as a whole is not
  already met by the code as it stands, and cannot be met by work that leaves the
  purpose unmet. Criteria carried over from an earlier ticket get no credit for it;
  judge them against this ticket's purpose.
- **Consistency:** the title, purpose, scope and criteria describe the same piece of
  work. The scope serves the purpose, the criteria cover the scope, and no two parts
  contradict.
- **References that resolve:** files, functions, commands and tickets the ticket names
  exist under those names.

- BAD criterion: "`load_config` returns a `ConfigResult` instead of `None`."
- BAD criterion: "Output matches the current implementation on all existing fixtures."
- GOOD criterion: "Starting the service with a missing config file exits non-zero and
  the error names the missing path."

## 3. Fix

Edit the ticket in the tracker to correct every gap from step 2.

- Fill each missing or broken part from the sources in step 1. Where a source states
  the purpose or the expected outcome, use it.
- Rewrite implementation criteria as the outcome they were standing in for.
- Correct references that point at the wrong name; remove ones that point at nothing.
- Keep the author's content. Move misplaced text to the right part instead of
  deleting it.
- Add no design. The ticket states what is needed; the implementer chooses how.
- Never invent a purpose, scope or outcome no source supports. Leave that part as it
  is and list it under `Unresolved`.

## 4. Report

```
STATUS: READY | UPDATED | INCOMPLETE | COULD NOT CHECK <reason>
Ticket: <id>
About: <one or two sentences: what the work is and who it serves>

Changes:
- <part>: <what was wrong> -> <what it says now>

Unresolved:
- <part>: <what is missing, and which sources were checked>
```

- `READY`: nothing needed changing.
- `UPDATED`: changes made; nothing unresolved.
- `INCOMPLETE`: at least one unresolved item. The ticket is not ready to start.
