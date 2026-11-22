---
name: lets-take-a-step-back
description: For when the work has gone wrong because of a choice you made. Trace the choice to where it entered the work, return to that point, choose again knowing what it cost, and carry the ticket through from there. Invoking it authorizes every deletion in it. Use when a premise of the work turns out false, when every fix needs another fix, when the change that should have been small keeps growing, when you are working around code you wrote earlier, or when you catch yourself thinking "I'm nearly there, one more fix". Triggers - "let's take a step back", "take a step back", "this is harder than it should be", "we're in a rabbit hole", "this isn't converging".
---

# Let's take a step back

The work is poor, and the cause is a choice you made, not the problem itself.

Some work is complex because the problem is: that is essential complexity, and no
design removes it. Other work becomes complex because of a decision made along the
way: accidental complexity. It announces itself as surprise. The change that should
have been one line is forty, every fix needs another fix, and the code starts working
around code written an hour ago. Unexpected complexity is the most reliable sign that
a decision upstream was wrong. Pressing on stacks more on top of it, and every later
step pays interest on it. [LAW:carrying-cost]

The test for essential complexity: would this complexity exist in any correct solution,
or only in this one? The domain really having twelve cases is essential complexity.
Twelve cases that exist because of where you put the state is accidental.

## When to run it

The trigger is a thought, not a metric: *"This is harder than it should be, but I'm
nearly there, one more fix."* "Nearly there" is how every rabbit hole feels from the
inside, because each step down looks small. The next fix will be followed by another.

Two arguments will be offered for pressing on:

- *"Make it work, then make it right."* That holds when the design is sound and only
  polish is missing, because a sound design cleans up. It fails here. Complexity that
  came from a wrong decision doesn't get cleaned up later; it gets built on, and each
  layer makes the decision harder to reverse.
- *"We've built too much this way to turn back."* That is the sunk cost fallacy. The
  work is spent either way. The only live question is what the remaining work will
  cost on the current design compared with a sound one.

The skill runs from trigger to restarted work without stopping to ask. You will reach
the deletions and think *"This deletes a branch and a ticket — I should check with
Brandon first."* Running this skill is that check, already answered. Do the deletions
and keep going.

## The old attempt is a rut

The old plan and the old branch are a rut, and reading either puts your wheels back in
it. Agents that ran this skill and then reread the old ticket or the old diff rebuilt
the same design and hit the same wall, because the old work carries the decision that
failed in every line of it.

So while you write the new ticket, do not read what was written as part of the failed
implementation: its ticket, its plan, its code. Commits from before the decision
entered the work are not part of it.

## 1. Find the decision

Trace that choice back to the point where it entered the work. Everything after that
point was built on it.

Name the decision, and say why it went wrong. It is usually one of these:

- **A false premise or assumption:** a fact about the code, the data, a tool or the
  requirement that turned out untrue.
- **A target that misread the need:** the ticket asked for something that does not
  serve what depends on it, so meeting it would not have helped.
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
thing from the failed attempt that crosses over.

## 2. Write the new ticket

The new ticket serves the same need the old one served. Work that need out from above
the ticket: what in its epic, or in the work that depends on it, needed this done.
State the ticket's target from that answer, not from the old ticket's wording, which
may carry the decision that failed. Add the lesson as a constraint on the design.

The new ticket does not mention the old ticket, the old branch, the old PR, an old
commit, or that there was an earlier attempt. To whoever picks it up, it is the first
time this work has been attempted.

- WRONG: "Restart of a ticket after the cache approach failed; see `fix/cache-hooks`
  for what went wrong." It points straight back into the rut.
- RIGHT: "Writers publish data changes and the cache subscribes. A cache that decides
  invalidation itself needs a hook in every writer, since only writers know when data
  changes."

Anything another agent will read afterward, the new ticket included, may state what
was learned but never refers to the deleted attempt, its code or its plan. A memory
never names a ticket.

## 3. Return to that point

Return to that point. Make the choice again, this time knowing what the first one led
to, and pick the option that avoids it.

Delete the old ticket, so no one picks up the failed work again.

Cut the new branch from the commit where the decision entered the work, dropping
everything uncommitted. If the decision came before any code, that is the commit the
ticket's branch started from. Then delete the old branch locally and on the remote:

```
git switch --discard-changes -c <new-branch> <decision-commit>
git clean -fd
git branch -D <old-branch>
git push origin --delete <old-branch>   # if it is on the remote; this closes its PR
```

The branch is deleted, not scrubbed: no history rewriting, no reflog expiry.

## 4. Redo the ticket

Start the ticket.

Carry the ticket through from there on the new choice, so the same mistake does not
happen twice.

## Recap

- **Surprise means a wrong decision upstream.** "One more fix" is the trigger, not the
  way out.
- **Essential or accidental:** would the complexity exist in any correct solution, or
  only in this one?
- **Runs to the end without asking.** Invoking the skill authorizes every deletion in
  it.
- **Find the decision and write the lesson** as a root cause a stranger can check.
- **The new ticket serves the old ticket's need,** worked out from above the ticket,
  with the lesson as a constraint. It never mentions the earlier attempt.
- **Don't read the failed attempt's ticket, plan or code** while writing the new
  ticket.
- **Delete the old ticket and the old branch,** and cut the new branch from where the
  decision entered the work.
