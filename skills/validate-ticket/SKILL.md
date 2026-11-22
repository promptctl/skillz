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
<id>` prints whichever of those the ticket has, and `lit dep ls <id>` lists what blocks the ticket and what it
blocks. Read the blocking tickets and the blocked ones too. The acceptance criteria
usually sit under a heading like "Done when", in the description, the plan or both. All
of them count. Where two disagree, the implementer may follow either one, so test both.

## 2. Find the need

Find why this ticket exists, in what stands behind it: the user's request recorded on
the ticket or its epic, the epic's goal, the tickets this one unblocks and what they
will do with its result, or the failure a bug report describes. Write the need in one
sentence, in terms of whoever uses the result: what a person sees, what an agent gets
back, what a process can now do. A source that only says what will be built names no
need.

Where the ticket explains itself, that explanation is one more claim. Check it
against those sources.

## 3. Test the target against the need

Picture the cheapest result that hits the target exactly as written. Does the user of
the work then have what they need?

## 4. Test the acceptance criteria

Each criterion should name something the user of the work can rely on afterward. It
fails when it names how to build the work instead: a function, a type, a file layout,
a sequence of steps. A criterion like that is met whenever the work is built that way,
whether or not the need is met.

Then test the criteria as a set, two ways:

- **Against an empty change.** Would the code the work starts from meet every
  criterion?
  One criterion that already holds is fine, since it guards something the work must
  not break. A set that already holds marks nothing as done.
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

A claim about something a blocking ticket will build is checked against that ticket's
acceptance criteria, not against today's code. The ticket is right to point at a world
that does not exist yet.

**An unchecked claim is never reported as true.** The moment this breaks comes late in
the run. A claim will be awkward to verify, and you will think *"this is obviously how
it works."* That was the author's thought too, and it is how a false claim got into
the ticket. Walk the ground or mark it unchecked.

## 6. Report

You are done when the need is written down with its source, the target has been
tested against it, the criteria have been tested one by one and as a set, and every
claim is marked with its evidence.

A finding is one of these, and nothing else:

- **no need**: you found no need behind the ticket, so there is nothing to test it
  against.
- **wrong need**: the need the ticket states is not the one its sources show.
- **target misses need**: the cheapest result that hits the target leaves the need
  unmet.
- **criterion names the build**: a criterion says how to build the work, not what the
  user can rely on.
- **criteria miss the need**: an empty change, or a build that misses the need, meets
  every criterion.
- **false claim**: anything the ticket asserts or assumes that is not true.
- **unchecked claim**: a claim you could not check, where the target or a criterion
  would change if the claim were false. A ticket that states the point as an open
  question, for the work to answer first, is not making a claim there.

A problem that fits several kinds is one finding, filed under the kind closest to its
cause. Wording, size, priority and taste are not findings. A sound ticket is READY. A padded
NOT READY sends a good ticket back for a rewrite it did not need.

Return:

```
VERDICT: READY | NOT READY | COULD NOT CHECK <reason>
Checked: <ticket id>, as read at <UTC time>

Need: <one sentence> (source: <where you found it>)

Findings:
1. <kind>
   About: <the ticket text quoted, or the text that assumes the claim, or "nothing - absent">
   Evidence: <file, command and output, the related ticket>
   Ticket should say: <the need or the fact>

Claims:
- <claim> - true | false | unchecked - <evidence>
```

The verdict is READY only when there are no findings.

**"Ticket should say" gives what is needed or what is true. The build is left to the
implementer.** By now you know the code well, and you will want to write down the fix
you would make. Hold it back. A need stated precisely leaves the implementer room to
choose the build, and it carries none of your guesses into the next ticket.

- BAD: "Ticket should say: wrap `fetch()` in a retry loop with three attempts."
- GOOD: "Ticket should say: the nightly sync runs unattended and must finish when the
  upstream API drops a request. Today one dropped request aborts the whole run
  (`sync/job.py`)."
