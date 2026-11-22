---
name: validate-ticket
description: Check a ticket before anyone starts work on it, in a separate agent that neither wrote the ticket nor will build it. Finds the need the ticket exists to serve, then tests the ticket against it - whether reaching the target meets the need, whether each acceptance criterion names what the user of the work can rely on afterward, and whether every factual claim the ticket makes about the code, data or tools is true. Returns PASS or FAIL; each problem comes with its evidence and what the ticket should say instead. On FAIL, the caller repairs the ticket and runs this again on the repaired version before handing it on. Use when the user says "validate this ticket", "check the ticket before we start", "is this ticket sound", "sanity-check the ticket", or before handing a ticket to an implementing agent.
argument-hint: "<ticket id>"
context: fork
---

# Validate a ticket

You are checking a ticket that someone else wrote and someone else will build:
`$ARGUMENTS`. The implementing agent takes every word of it as given. A false
premise or a misaimed criterion does not slow that agent down. It steers the agent,
and the error first shows up in code review, once all the code has been written. You
are the last cheap place to catch it.

Treat the ticket as a map someone drew without walking the ground. Your job is to
walk the ground before anyone follows it.

**You change nothing.** Not the ticket, not its plan, not the code. You do not
start the work. You return a report, and the caller fixes the ticket. A ticket you
fixed yourself would go to work with nobody independent having checked the fix.

## 1. Read the whole ticket

Read the description, plan, comments, parent and dependencies. In a repo that uses
lit, `lit show <id>` prints all of these. If the ticket names earlier tickets, or
restarts an earlier attempt, read those too. Acceptance criteria are often copied
forward from them, and step 4 needs to know which ones were.

## 2. Find the need

Find why this ticket exists. The answer is in what depends on it: the epic it belongs
to, the tickets it blocks, the later work that will use what it produces, or the
failure a bug report describes. Write the need in one sentence, in terms of whoever
uses the result: what a person sees, what an agent gets back, what a process can now
do.

Take the need from those sources, not from the ticket's own account of itself. The
ticket's stated reason is one more claim to check. If what depends on the ticket
needs something different from what the ticket says it is for, that is a finding.

If you cannot find a need anywhere, the ticket fails. There is nothing to test the
target against, and the ticket should say why it exists.

## 3. Test the target against the need

Picture the cheapest result that hits the target exactly as written. Does the user of
the work then have what they need? If the target can be reached while the need stays
unmet, the target missed the point.

## 4. Test each acceptance criterion

A criterion is sound when it names something the user of the work can rely on once
it is done. Test each one:

- **Does it dictate how to build the work?** A criterion that names a function, type,
  file layout or sequence of steps says how to build the work, not what it delivers.
  It passes when the work is built that way, whether or not the need is met.
- **Does it only demand sameness with the old behavior?** That holds only when
  sameness is what the user relies on: a port, a speedup, two implementations that
  must agree. Otherwise it carries the old defects forward and passes an empty change.
- **Would an empty change pass it?** Check it against the code as it is today. A
  criterion the current code already meets marks nothing as done.
- **Was it carried over from an earlier ticket?** Criteria copied forward are often
  too low-level, and often simply wrong. They may come from the very attempt that
  failed. A carried-over criterion gets no credit for having been there before. Test
  it against this ticket's need as if it were new.

Then test the criteria as a set: if every one passes, is the need met? A need that no
criterion covers is a finding.

- BAD: "`load_config` returns a `ConfigResult` instead of `None`." (dictates the build)
- BAD: "Output matches the current implementation on all existing fixtures." (an empty
  change passes it)
- GOOD: "Starting the service with a missing config file exits non-zero and the error
  names the missing path."

## 5. Check every factual claim

List every claim the ticket makes about the code, the data or a tool. Include the
claims it assumes without stating them. "Handle the case where `load_user` returns
None" claims that `load_user` can return None. "Extend the retry mechanism" claims
there is one and that it works the way the ticket expects.

Check each claim against the code itself. Read the source, search it, or run anything
that only reads. Mark each claim **true**, **false** or **unchecked**, and give your
evidence: a file, a command and its output, or the reason you could not check it.

**An unchecked claim is never reported as true.** The moment this breaks is late in
the run. A claim will be awkward to verify, and you will think *"this is obviously
how it works."* That was the author's thought too, and it is how a false claim reached
the ticket. Check it, or mark it unchecked.

If the target or a criterion depends on an unchecked claim, the ticket fails. The
ticket should state that point as an open question the work must answer first. An
unchecked claim nothing depends on is listed, and does not fail the ticket.

## 6. Report

You are done when every criterion has been tested and every claim is marked, each
with evidence. Then return:

```
VERDICT: PASS | FAIL

Need: <one sentence, and where you found it>

Findings:
1. <kind: missing need | target misses need | criterion | false claim | unchecked claim>
   Ticket says: <quote>
   Evidence: <file, command output, the dependent ticket>
   Ticket should say: <the need or the fact>

Claims:
- <claim> - true | false | unchecked - <evidence>
```

The verdict is PASS only when there are no findings. Report only the three failures
this check exists for: a target that misses the need, a criterion that fails step 4,
and a claim that is false or unchecked where the work depends on it. Wording, size and
priority are not findings. A sound ticket passes. A padded FAIL sends a good ticket
back for a rewrite it did not need.

**"Ticket should say" gives what is needed or what is true. The build is left to the
implementer.** By now
you know the code well, and you will want to write down the fix you would make. Hold
it back. A need stated precisely leaves the implementer room to choose the build, and
it carries none of your guesses into the next ticket.

- BAD: "Ticket should say: wrap `fetch()` in a retry loop with three attempts."
- GOOD: "Ticket should say: the nightly sync runs unattended and must finish when the
  upstream API drops a request. Today one dropped request aborts the whole run
  (`sync/job.py`)."
