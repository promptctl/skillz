---
name: lets-take-a-step-back
description: Stop when the work is bogged down, find the decision that caused the trouble, and restart the ticket clean - its branch deleted, its ticket purged, a fresh ticket and branch cut from where it started, carrying only the lesson. The old attempt is read once, to find the lesson, and never again. Runs start to finish without asking - invoking it is the authorization for every deletion in it. Handed a wider scope than one ticket (several tickets, a range of days), it investigates first and decides which tickets restart before deleting anything. Use when complexity surprises you — a fix that needs another fix, a workaround or special case for code you wrote earlier, a step far harder than the plan expected, a rabbit hole with no clear strategy out. Unexpected complexity is the signal, and "I'm nearly there, one more fix" is the thought that keeps you in the hole. Also the mechanical trigger: the fourth review pass in one PR's cycle that returned findings to fix. Triggers — "let's take a step back", "take a step back", "this is getting too complicated", "why is this so hard", "we're in a rabbit hole", "this isn't converging", "the fourth review found things again".
---

# Let's take a step back

Some work is complex because the problem is: that is essential complexity, and no
design removes it. Other work becomes complex because of a decision made along the
way: accidental complexity. It announces itself as surprise. The change that should
have been one line is forty, every fix needs another fix, and the code starts working
around code written an hour ago. Unexpected complexity is the most reliable sign that
a decision upstream was wrong. Pressing on stacks more on top of it, and every later
step pays interest on it. [LAW:carrying-cost]

This skill stops the work, finds the root cause, deletes the old attempt, and restarts
from where the ticket started with nothing but the lesson. Handed a wider scope than one
ticket, it investigates first and decides which tickets restart.

## The old attempt is a rut

The old plan and the old branch are a rut, and reading either puts your wheels back in
it. Agents that ran this skill and then reread the old ticket or the old diff rebuilt
the same design and hit the same wall, because the old work carries the decision that
failed in every line of it. So the old attempt is read once, in step 2, to find the
lesson, and after the lesson is written it is gone: its branch deleted, its ticket
purged, and nothing of it read again in any way. That is the one hard line in this
skill.

The skill runs from trigger to restarted work without stopping to ask. You will reach
the deletions and think *"This deletes a branch and purges a ticket — I should check
with Brandon first."* Running this skill is that check, already answered. The commits
stay in git's history, and the purge is what he asked for. Do the deletions and keep
going.

## When to run it

The trigger is a thought, not a metric: *"This is harder than it should be, but I'm
nearly there, one more fix."* "Nearly there" is how every rabbit hole feels from the
inside, because each step down looks small. The next fix will be followed by another.

Signs the complexity is accidental:

- It targets your own code rather than the problem: guards, special cases, adapters,
  flags, a second code path beside the first.
- Each fix exposes the next one, or one change needs edits scattered across many
  places.
- You can no longer state the strategy in one sentence.
- A review cycle won't converge: the fourth pass with findings to fix.

The test for essential complexity: would this complexity exist in any correct solution,
or only in this one? The domain really having twelve cases is essential complexity.
Twelve cases that exist because of where you put the state is accidental.

Two arguments will be offered for pressing on:

- *"Make it work, then make it right."* That holds when the design is sound and only
  polish is missing, because a sound design cleans up. It fails here. Complexity that
  came from a wrong decision doesn't get cleaned up later; it gets built on, and each
  layer makes the decision harder to reverse.
- *"We've built too much this way to turn back."* That is the sunk cost fallacy. The
  work is spent either way. The only live question is what the remaining work will
  cost on the current design compared with a sound one.

## 1. Find where the ticket started

When the trouble is in the ticket you are working, which is the usual case, find the
commit where that ticket started: the point where its branch diverged from its base
(`git merge-base <base> HEAD`, with `<base>` the branch the PR targets). Every change
after that commit is discarded. Nothing is judged commit by commit, and nothing is kept
but the lesson from step 2. Note the ticket's place too — its epic, lane, the ticket
ranked after it, and its dependency edges — because the new ticket takes that place.

When you are handed a wider scope (several tickets, a range of commits, the last few
days), you are investigating, and the scope is where to look, not where to go back to.
The pull is to read "the last four days are a mess" as "go back four days": that throws
out every sound change in the window along with the wrong ones, before anything was
assessed. Find the root causes first (step 2), each with the ticket that made it and the
commit where that ticket started. Then decide, for each, what restarts:

- **Accidental complexity from a decision:** the ticket that made it restarts.
- **A goal not worth reaching:** the ticket is closed, not restarted
  (`lit close <id> --resolution wontfix`), and its branch deleted. Say why in your
  report.
- **Work built on a restarted or closed ticket** goes with it: its ticket restarts too,
  or closes if the goal it served is gone. Nothing is kept that was built on a
  foundation that is changing or gone.
- **Essential complexity:** nothing restarts.

Nothing is deleted before that verdict. Each ticket that restarts then goes through
steps 3 to 5 on its own.

## 2. Find the root cause

This is the one time the old attempt is read. Read the branch, the ticket and the
review findings for one purpose: the decision that went wrong.

Name the decision, and say why it went wrong. It is usually one of these:

- **A false premise or assumption:** a fact about the code, the data, a tool or the
  requirement that turned out untrue.
- **A misguided goal:** the goal as stated isn't worth reaching, and the right move is
  a different direction. Choose the direction you would take, restart toward it, and
  say in your report what you redirected and why.
- **A small implementation decision that cascaded:** where a piece of state lives,
  which layer makes a decision, a type that admits more than it should.

Then write the lesson as a root cause statement: what was decided, why it looked
right, and what it forced afterward. Write it so that a stranger could check it
against the code, and as a fact about the design, not a story about an attempt.

- BAD: "Lesson: be more careful with state." Nobody can act on this, so the lesson
  will be learned again.
- GOOD: "The cache decided when to invalidate, but only writers know when data
  changes, so every writer grew a hook into the cache. Writers publish the change and
  the cache subscribes."

The lesson is what keeps the next attempt out of the same hole, and it is the only
thing that crosses over.

## 3. Write the new ticket

Write a new ticket from nothing: the goal, stated fresh from what the work is for; the
lesson, stated as a constraint on the design; and a new plan. Nothing is pasted from the
old ticket, because its words carry the old plan.

The new ticket does not mention the old ticket, the old branch, the old PR, an old
commit, or that there was an earlier attempt. To whoever picks it up, it is the first
time this work has been planned.

- WRONG: "Restart of lit-42 after the cache approach failed; see `fix/cache-hooks`
  for what went wrong." It points straight back into the rut.
- RIGHT: "Writers publish data changes and the cache subscribes. A cache that decides
  invalidation itself needs a hook in every writer, since only writers know when data
  changes."

Judge the plan by its total complexity over the whole effort: the area under the
complexity curve, not its height at any one moment. A plan that is cheap today and
forces a workaround at every later step loses to one that costs more up front and stays
flat. For each decision in the plan, ask how hard it will be to change later, and prefer
the one that stays cheap to change.

In a lit repo, the new ticket takes the old one's place: same epic and lane, ranked
directly below the old one before the purge (`lit rank <new> --below <old>`), with the
same dependency edges. The place carries over; the words do not. Create it with
`lit new --parent <epic> --lane <lane> --plan-file <file>`.

## 4. Delete the old attempt

Cut the new branch from the commit where the ticket started, dropping everything
uncommitted, then delete the old branch locally and on the remote, then purge the old
ticket:

```
git switch --discard-changes -c <new-branch> <ticket-start-commit>
git clean -fd
git branch -D <old-branch>
git push origin --delete <old-branch>   # if it was pushed; this closes its PR
lit purge <old-id> --yes
```

The branch is deleted, not scrubbed: no history rewriting, no reflog expiry. Its
commits survive in git, and you do not read them.

If the ticket has already been merged, branch from the current base instead, and make
reversing it the new plan's first step: `git revert` its merge commit, and the merges of
the work built on it, without opening their diffs.

## 5. Build from the plan

Build from the new ticket's plan alone. The old attempt is not read in any way: not
`git log`, `git show` or `git diff` of its commits, not the reflog, not the closed PR or
its review comments, not a search of the old session's transcript, not a note or memory
written about it.

The thought will come mid-way through: *"That function on the old branch was fine.
It's still in the reflog — I'll just look at how I did it."* That glance is how the old
design gets rebuilt. The function was written under the decision that failed, and it
brings that decision with it. Write it again against the new plan, and if it comes out
the same, good.

## Recap

- **Unexpected complexity means a wrong decision upstream.** "One more fix" is the
  trigger, not the way out.
- **Runs to the end without asking.** Invoking the skill authorizes every deletion in
  it.
- **The old attempt is read once, for the lesson,** then never again in any way.
- **A wider scope is where to look, not where to go back to.** Investigate first, then
  decide which tickets restart; what was built on them goes with them.
- **Essential or accidental:** would it exist in any correct solution, or only in this
  one?
- **Write the root cause as a statement a stranger can check.** "Be more careful" is
  not a lesson.
- **The new ticket is written from nothing** and never mentions the old ticket, branch,
  PR, or that there was an attempt. It takes the old ticket's place in the backlog.
- **Delete the old branch, local and remote, and purge the old ticket.** No history
  rewriting.
- **Plan for the lowest total complexity,** the area under the curve.
