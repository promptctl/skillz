---
name: word-police
description: Keep a project's vocabulary on industry-standard software engineering terms - catch invented words where an accepted term already exists, and standard terms used with a meaning they do not have. Use when naming anything (a type, function, module, file, CLI flag, config key, metric, ticket, doc heading); when writing or reviewing docs, comments, commit messages, tickets, or handoffs; and when the user says "word police", "check the terminology", "are these standard terms", "we're inventing words", "what's the real name for this", "audit the naming", or "rename this to the standard term". Optionally scoped to a path, a diff, or a document.
---

# Word police

A standard term is a key the reader already carries. Say "exponential backoff" and
every engineer, every search engine, every library's docs, and every model's training
opens the same door: jitter, a cap, a retry budget, the thundering herd it prevents.
Say "the patience ramp" and nothing opens. The reader has to reverse-engineer the
concept from the code, alone, and then carry a private translation table for as long
as they work on the project. A coined term is a key cut for one lock, and only its
author holds it.

That cost is paid by every reader, forever, and it is silent. Nobody files a bug
that says "I lost an hour learning what a 'shepherd' is." The project just gets
harder to understand and work on, one private word at a time, and the agent who
coined each word never sees the bill. An LLM reader pays twice: a standard term
pulls in everything it knows about that concept, its failure modes, and its known
fixes. A coined term pulls in nothing, so it reasons from scratch about a problem
the field has already solved.

The rule: **when the field has a widely accepted term for a concept, use it, with
the meaning the field gives it.**

## The two violations

**Coinage**: a new word for a concept that already has a name.

| Coined | Standard |
|---|---|
| `RetryRamp`, "patience ramp" | exponential backoff |
| `Shepherd` (restarts crashed workers) | supervisor |
| "drift fixer" (compares desired and actual state, converges them) | reconciliation loop, reconciler |
| `FanCollector` (sends to N services, gathers the replies) | scatter-gather |
| "tripwire" that stops calling a failing dependency | circuit breaker |
| "stamp" on a row to detect a concurrent edit | version column, optimistic locking |
| "shadow object" in tests that records its calls | spy (a kind of test double) |
| "the replay book" of events that rebuilds state | event log, event sourcing |

**Misuse**: a standard term given a meaning it doesn't have. This is worse than
coinage. The key opens the wrong door, and the reader walks through it confidently,
carrying the wrong knowledge.

| Misused | What it means | What was meant |
|---|---|---|
| "idempotent" for "same input, same output" | applying it twice equals applying it once | deterministic, pure |
| "refactor" for a change in behavior | restructuring that preserves behavior | rewrite, change, fix |
| "deprecated" for "removed" | still present, discouraged, scheduled for removal | removed |
| "race condition" for any intermittent failure | outcome depends on the timing of concurrent operations | flaky test, intermittent failure |
| "singleton" for any module-level value | a class constrained to one instance | global, module-level state |

## What counts as standard

A term is standard when you can point to it in established use with this meaning.
Where to look, from nearest to widest:

1. The vocabulary of the stack the project is built on. In Kubernetes it's
   "controller" and "reconcile"; in React it's "hydration"; in Postgres it's
   "vacuum". Code that uses its platform's words reads as part of the platform.
2. Specs and standards: RFCs, POSIX, W3C, language specifications.
3. The field's canonical literature: the GoF patterns, Fowler's *Refactoring* and
   *Patterns of Enterprise Application Architecture*, *Enterprise Integration
   Patterns*, Meszaros's *xUnit Test Patterns*, *Domain-Driven Design*, the Google
   SRE books, *Release It!*.
4. Broad established use: a Wikipedia article, major vendor documentation, the
   term's common meaning in the ecosystem's own docs.

The search test is the quick check: search for the term next to the ecosystem's
name. If the first page of results describes this concept, the term is standard.
If it describes a different concept, the term is misused. If it describes nothing,
the term is coined.

Your memory is not evidence on a close call. A model can produce a plausible term
that is itself a coinage, or one that means something else. Proposing one is the
same violation, with false authority behind it. When you are unsure, look it up
(WebSearch, the stack's docs) before you call anything standard.

**Sounding invented is not the same as being invented.** Strangler fig, bulkhead,
circuit breaker, canary release, tombstone, thundering herd, transactional outbox,
saga, dead letter queue, dogfooding, bikeshedding, and yak shaving all sound whimsical,
and all are standard. Flagging them is the mirror image of the failure this skill
exists to stop. It replaces the key everyone holds with your guess. Judge a term by
how it is used, not by how it sounds.

## When a new word is right

A new word is right when the field has no word for the concept. That's rare for
technical mechanisms and common for the business domain. A coined name is still
fine in three cases:

- **Domain terms come from the domain.** If the accountants say "journal entry", the
  code says `JournalEntry`. That is domain-driven design's ubiquitous language, and
  it is the opposite of coinage: the developers adopt the experts' words instead of
  making up their own.
- **Proper names are names, not terms.** The project's name, a service's name, or a
  product feature's name labels one specific thing. It doesn't pretend to describe a
  concept the field already has.
- **A genuinely new concept** gets a name built from standard parts where it can
  ("bounded retry queue" beats "the patience pool"). Define it once, where readers
  will look (a glossary or the module's doc), and use it the same way everywhere.

Someone will cite ubiquitous language as permission to invent: "DDD says the team
should build its own vocabulary." On its home turf that is correct: the language of
the *business domain*, drawn from the people who work in it. It doesn't cover the
technical mechanisms the field has already named. A retry policy is not a domain
concept, and DDD never asked anyone to rename exponential backoff.

"Naming is one of the two hard things," someone will say, as if that licenses any
name. It argues the other way. Naming is hard, and the field has spent decades
settling names for its recurring concepts. Using its settled name is how you avoid
that hard work.

An image may *explain* a term, but it doesn't *replace* it as the name. "A circuit
breaker trips like the one in your fuse box" helps a reader. A class named `FuseBox`
takes the key away from them.

## The moments it slips

You will be naming a thing mid-implementation, and you'll think: *"The standard term
doesn't quite fit. Ours is a little different."* That's the moment. Real differences
almost always have names of their own: a backoff with a cap is "capped exponential
backoff", and a cache that serves stale data while refreshing it is
"stale-while-revalidate". Use the standard term, plus the standard qualifier if one
exists. A name that fits your case perfectly and nobody else's is a key only you
hold.

You will think: *"My name is more vivid, and it'll stick."* It will stick for you.
Every later reader is left translating it, and the vividness is no help to someone
who doesn't know what it refers to.

You will be three hours into a session, writing a ticket or a handoff, and a word you
coined in the second hour will feel standard, because you've used it a dozen times.
Familiarity from your own context is not industry standard. Before a term leaves the
session in a ticket, a commit message, a doc, or a handoff, ask whether a reader who
wasn't here would know it.

You will meet a coined term the codebase already uses everywhere, and think:
*"Everyone here says it; I'll match it."* Don't spread it into new code or new docs.
Report it with its standard equivalent, and leave the rename to a change of its own
(see below).

You will read a term that sounds like jargon and think: *"Nobody says that, flag it."*
Run the search test first. *Strangler fig* is standard, and so is *tombstone*.

## Doing the work

**While writing.** Before you name something, ask what the field calls it. If you'd
have to explain your name with a sentence that contains a standard term, that term
is the name.

**Auditing.** The scope is what the user named: a path, a diff, or a document. If
they named none, use the current diff when there is one, and the whole repo when
there isn't. In scope, gather the candidate terms:

- names of types, modules, files, and public functions
- CLI flags, config keys, metric names, and log names
- recurring nouns in docs, comments, and tickets
- words set in quotes, capitalized as if they were concepts, or defined in a glossary

Classify each one, and report each finding in this shape:

| Term | Where | Violation | Standard term | Evidence |
|---|---|---|---|---|
| `Shepherd` | `src/workers/shepherd.ts`, docs/ops.md | coinage | supervisor | Erlang/OTP supervisor; systemd and supervisord docs |

The evidence column is the one that matters. A finding without it is your guess,
handed over as if it were the field's answer.

**Renaming.** Rename the concept everywhere in one change: code, tests, docs,
comments, config, and log and metric names. Then the concept has one name. Don't
leave an alias or a "formerly X" comment. A half-finished rename gives one concept
two names, and that is worse than either name alone. A public name is a breaking
change: a published API, a CLI flag, a config key, a wire format, a metric someone
graphs. Report it with its consumers rather than renaming it in place, unless the
user asked for the break.

**Done** means every term in scope is one of these, each with its evidence:

- a standard term used with its standard meaning
- a domain term taken from the domain
- a proper name
- a defined coinage the field has no word for

Anything else is still a coinage or a misuse, and the audit isn't finished.
