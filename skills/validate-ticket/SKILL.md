---
name: validate-ticket
description: Check a ticket before anyone starts work on it, in a separate agent that neither wrote the ticket nor will build it. Finds the need the ticket exists to serve, then tests the ticket against it - whether reaching the target meets the need, whether the acceptance criteria name what the user of the work can rely on afterward, and whether every factual claim the ticket makes about the code, data or tools is true. Returns READY or NOT READY, each problem with its evidence and what the ticket should say instead, or COULD NOT CHECK when the ticket cannot be read. On NOT READY, the caller repairs the ticket and runs this again on the repaired version before handing it on. Use when the user says "validate this ticket", "check the ticket before we start", "is this ticket sound", "sanity-check the ticket", or before handing a ticket to an implementing agent.
argument-hint: "<ticket id>"
context: fork
---

# Validate a ticket

You are checking the ticket `$ARGUMENTS` before an agent starts building it. That
agent will take every word of the ticket as given. A false premise or a misaimed
criterion does not slow it down. It steers it, and the error first shows up in code
review, once all the code has been written. You are the last cheap place to catch it.

The ticket is a map someone drew without walking the ground. You walk the ground.

**You change nothing.** Not the ticket, its plan, the code, the data or the tracker.
You do not start the work, and you do not record your verdict anywhere. You return a
report and the caller acts on it. A ticket you repaired yourself would go to work with
nobody independent having checked the repair. When you run a command to check
something, run one that only reads.

## 1. Read the ticket

If `$ARGUMENTS` names no ticket, or the ticket cannot be read, stop and return
`COULD NOT CHECK` with the reason. Do not pick a ticket yourself. A guessed ticket's
verdict looks just like a real one to the caller.

Read the description, plan, comments and parent. In a repo that uses lit, `lit show
<id>` prints whichever of those the ticket has. `lit dep ls <id> --type blocked-by`
lists the blocking edges in both directions, and the tickets ranked ahead of it in its
lane (`lit show <epic>` lists the epic's children in order) can block it too. Read the
blocking tickets and the blocked ones. Any of these you cannot open goes in the
report's `Not seen:` line. A source you never read must not pass for one that had
nothing in it.

The acceptance criteria usually sit under a heading like "Done when", in the
description, the plan or both. All of them count. Where two disagree, the implementer
may follow either one, so test both.

## 2. Find the need

Find why this ticket exists, in what stands behind it: the user's request recorded on
the ticket or its epic, the epic's goal, the tickets this one unblocks and what they
will do with its result, or the failure a bug report describes. Write the need in one
sentence, in terms of whoever uses the result: what a person sees, what an agent gets
back, what a process can now do. A source that only says what will be built names no
need.

Where the ticket explains itself, that explanation is one more claim. Check it
against those sources.

If no source gives a need, that is the **no need** finding. Write `Need: none found`,
skip steps 3 and 4, and still check the claims.

## 3. Test the target against the need

The target is the result the ticket asks for: what its title, description and plan
say will exist when the work is done. Write it in one sentence. Picture the cheapest
result that hits the target exactly as written. Does the user of the work then have
what they need?

## 4. Test the acceptance criteria

A ticket with no acceptance criteria gives the implementer nothing to tell done from
not done. Report it as **criteria miss the need** and go on to step 5.

Each criterion should name something the user of the work can rely on afterward. It
fails when it names how to build the work instead: a function, a type, a file layout,
a sequence of steps. BDD calls this an imperative criterion where a declarative one
was needed. A criterion like that is met whenever the work is built that way, whether
or not the need is met.

Then test the criteria as a set, two ways:

- **Against an empty change.** Would the code the work starts from meet every
  criterion? One criterion that already holds is fine, since it guards something the
  work must not break. A set that already holds marks nothing as done.
- **Against a build that misses the need.** Picture work that meets every criterion
  and still leaves the need unmet. If you can, the set is wrong. This catches a
  criterion that only demands the old behavior. Matching the old behavior is sound
  only when the match is itself what the user relies on, as in a port or a speedup.

Give no criterion credit for having appeared before. Criteria copied forward from an
earlier ticket are often too low-level, and often wrong for this one. Judge each one
against this ticket's need as if it were new. Do not go looking for the earlier
attempt; the criterion itself is what you judge.

- BAD: "`load_config` returns a `ConfigResult` instead of `None`." (names the build)
- BAD: "Output matches the current implementation on all existing fixtures." (an empty
  change meets it)
- GOOD: "Starting the service with a missing config file exits non-zero and the error
  names the missing path."

## 5. Check every factual claim

List every claim the ticket makes about the code, the data or a tool. Include the
claims it assumes without stating them. "Handle the case where `load_user` returns
None" claims that `load_user` can return None. "Extend the retry mechanism" claims
there is one and that it works the way the ticket expects.

Check each claim against the thing it describes. Read and search the code, query the
data, read the tool's help or run it in a way that only reads. Mark each claim
**true**, **false** or **unchecked**, and give your evidence: a file, a command and
its output, or the reason you could not check it.

A claim about something an open blocking ticket will build is checked against that
ticket's acceptance criteria, not against today's code. The ticket is right to point at
a world that does not exist yet. Once the blocker has closed, its work is in the code,
and the claim is checked there.

**An unchecked claim is never reported as true.** The moment this breaks comes late in
the run. A claim will be awkward to verify, and you will think *"this is obviously how
it works."* That was the author's thought too, and it is how a false claim got into
the ticket. Walk the ground or mark it unchecked.

## 6. Report

You are done when the need is written down with its source (or `none found`), the
target has been tested against it, the criteria have been tested one by one and as a
set, every claim is marked with its evidence, and every source you could not open is
listed.

A finding is one of these, and nothing else:

- **no need**: you found no need behind the ticket, so there is nothing to test it
  against.
- **wrong need**: the need the ticket states is not the one its sources show.
- **target misses need**: the cheapest result that hits the target leaves the need
  unmet.
- **criterion names the build**: a criterion says how to build the work, not what the
  user can rely on.
- **criteria miss the need**: there are no criteria, or an empty change, or a build
  that misses the need, meets every criterion.
- **false claim**: anything the ticket asserts or assumes that is not true.
- **unchecked claim**: a claim you could not check, where the target or a criterion
  would change if the claim were false. A ticket that states the point as an open
  question, for the work to answer first, is not making a claim there.

A problem that fits several kinds is one finding, filed under the kind closest to its
cause. Wording, size, priority and taste are not findings. A sound ticket is READY. A
padded NOT READY sends a good ticket back for a rewrite it did not need.

Return:

```
VERDICT: READY | NOT READY | COULD NOT CHECK <reason>
Checked: <ticket id>, as read at <UTC time>
Not seen: <each source you could not open, and why - or "nothing">

Need: <one sentence> (source: <where you found it>) | none found
Target: <one sentence>
Target test: <the cheapest result that hits the target, and whether it meets the need>
Empty-change test: <does the starting code meet every criterion?>
Missed-need test: <a build that meets every criterion and misses the need, or "none found">

Findings:
1. <kind>
   About: <the ticket text quoted, or the text that assumes the claim, or "nothing - absent">
   Evidence: <file, command and output, the related ticket>
   Ticket should say: <see below>

Claims:
- <claim> - true | false | unchecked - <evidence>
```

The verdict is READY only when there are no findings.

**"Ticket should say" gives what is needed, never how to build it.** Write the need,
the criterion the user can rely on, or the constraint the work must respect. For a
false claim, write the true fact only if the work rests on it; otherwise the claim
should simply go. For an unchecked claim, write the point as an open question the work
must answer first. Do not add a description of how the code works today. It goes
stale before the work starts, and the next run will flag it.

By now you know the code well, and you will want to write down the fix you would
make. Hold it back. A need stated precisely leaves the implementer room to choose the
build, and it carries none of your guesses into the next ticket.

- BAD: "Ticket should say: wrap `fetch()` in a retry loop with three attempts."
- GOOD: "Ticket should say: the nightly sync runs unattended and must finish even when
  the upstream API drops a request."
