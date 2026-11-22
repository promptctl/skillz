---
name: validate-ticket
description: Check a ticket before work starts on it, in a separate agent from the one that wrote it and the one that will build it. Confirms the ticket states the need it serves, that its target meets that need, that its acceptance criteria describe outcomes rather than implementation, and that its factual claims about the code, data and tools are true. Returns READY, NOT READY with findings, or COULD NOT CHECK. Use when the user says "validate this ticket", "check the ticket before we start", "is this ticket sound", "sanity-check the ticket", or before handing a ticket to an implementing agent.
argument-hint: "<ticket id>"
context: fork
---

# Validate a ticket

Ticket: `$ARGUMENTS`

Read-only. Do not modify the ticket, the code, the data or the tracker, and do not
record the verdict anywhere. Return the report.

## 1. Read the ticket

If `$ARGUMENTS` names no ticket or the ticket cannot be read, return
`COULD NOT CHECK <reason>`.

Read the description, plan, comments and parent. With lit:

- `lit show <id>`: description, plan, comments, parent
- `lit dep ls <id> --type blocked-by`: blockers and blocked tickets
- `lit show <epic>`: siblings in rank order; tickets ranked ahead in the lane can block it

Read the blocking and blocked tickets. List anything you could not open under
`Not seen:`.

Acceptance criteria are usually under "Done when", in the description, the plan or
both. Check all of them. If two disagree, check both.

## 2. Need

State in one sentence why the ticket exists, in terms of whoever uses the result
(what a person sees, what an agent gets back, what a process can do). Sources: the
request recorded on the ticket or its epic, the epic's goal, the tickets this one
unblocks, the failure a bug report describes. A description of what will be built is
not a need. Treat the ticket's own rationale as a claim and check it against those
sources.

If no source gives a need: `Need: none found`, finding **no need**, skip steps 3–4,
continue to step 5.

## 3. Target

State in one sentence what the ticket says will exist when done. Take the minimal
result that satisfies it as written. If that result leaves the need unmet, the finding
is **target misses need**.

## 4. Acceptance criteria

No criteria: finding **criteria miss the need**; continue to step 5.

Each criterion must describe an outcome the user of the work can rely on, not an
implementation (a function, type, file layout or sequence of steps). An
implementation criterion is **criterion names the build**.

Test the set:

- **Empty change:** if the starting code already meets every criterion, the set is
  **criteria miss the need**. A single criterion that already holds is fine; it
  guards against regression.
- **Missed need:** if a build can meet every criterion and still leave the need
  unmet, the set is **criteria miss the need**. Parity with old behavior is a valid
  criterion only when parity is the need (a port, a speedup).

Judge criteria carried over from earlier tickets against this ticket's need.

- BAD: "`load_config` returns a `ConfigResult` instead of `None`." (implementation)
- BAD: "Output matches the current implementation on all existing fixtures." (an empty change meets it)
- GOOD: "Starting the service with a missing config file exits non-zero and the error names the missing path."

## 5. Factual claims

List every claim the ticket makes or assumes about the code, data or tools, including
claims inside quoted requests. "Handle the case where `load_user` returns None"
claims `load_user` can return None. "Extend the retry mechanism" claims one exists and
works as described.

Verify each one by reading the code, querying the data, or running read-only
commands. Mark it **true**, **false** or **unchecked**, with evidence (file, command
and output, or why it couldn't be checked). Never mark an unverified claim true.

A claim about something an open blocker will build is checked against that blocker's
acceptance criteria. Once the blocker is closed, check it against the code.

## 6. Report

Finding kinds:

- **no need**: no source gives a need.
- **wrong need**: the ticket's stated need is not the one its sources show.
- **target misses need**: the minimal result that hits the target leaves the need unmet.
- **criterion names the build**: a criterion specifies implementation, not outcome.
- **criteria miss the need**: no criteria, or an empty change or a need-missing build meets them all.
- **false claim**: something the ticket asserts or assumes is untrue.
- **unchecked claim**: a claim you could not verify, where the target or a criterion would change if it were false. A point the ticket poses as an open question is not a claim.

File each problem once, under the kind closest to its cause. Wording, size, priority
and style are not findings.

```
VERDICT: READY | NOT READY | COULD NOT CHECK <reason>
Checked: <ticket id>, as read at <UTC time>
Not seen: <each source not opened, and why - or "nothing">

Need: <one sentence> (source: <where>) | none found
Target: <one sentence>
Target test: <minimal result that hits the target; does it meet the need?>
Empty-change test: <does the starting code meet every criterion?>
Missed-need test: <a build that meets every criterion and misses the need, or "none found">

Findings:
1. <kind>
   About: <quoted ticket text, or "nothing - absent">
   Evidence: <file, command and output, related ticket>
   Ticket should say: <replacement text>

Claims:
- <claim> - true | false | unchecked - <evidence>
```

READY only when there are no findings.

"Ticket should say" states the need, an outcome criterion, or a constraint, never an
implementation. For a false claim, give the true fact only if the work depends on it;
otherwise delete the claim. Leave quoted requests verbatim and put the true fact
beside them. For an unchecked claim, rewrite it as an open question the work answers
first. Don't describe current code beyond the one fact the work depends on.

- BAD: "Ticket should say: wrap `fetch()` in a retry loop with three attempts."
- GOOD: "Ticket should say: the nightly sync runs unattended and must finish even when the upstream API drops a request."
