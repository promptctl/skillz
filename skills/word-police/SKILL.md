---
name: word-police
description: Realign a project that has grown its own bespoke vocabulary with the standard terminology of its fields - software engineering first, plus whatever domains the project lives in (finance, audio, networking, ...). Fans Haiku subagents across the whole codebase to find invented words, misused standard terms, near-miss variants, internal synonyms, and unnamed concepts; verifies each against established usage; and works with the user, round by round, to lock in a rename proposal. Renames only what the user explicitly accepts. Use when the user says "word police", "align our terminology", "we're inventing words", "what's the standard term for this", "audit the naming", "rename to standard terms", or wants a codebase's vocabulary brought in line with its industry. Optionally scoped to a path.
---

# Word police

This skill brings a project's vocabulary back in line with the standard terminology of
its fields. When it's done, a concept has the same name here as in every other project
and in the field's literature. Someone from the industry can open the project and
recognize the familiar concepts on sight. The skill finds the bespoke words, proposes
the standard ones, and works with the user until they lock in a proposal.

**It renames only what the user explicitly accepts.** Until then, everything is
read-only: scanning, verifying, proposing, revising. The gate is the user's own
design for this skill, and it holds for every item, however obvious that item looks.

## Why the words matter

A standard term is a key the reader already carries. Say "exponential backoff" and
every engineer, every search engine, every library's docs, and every model's training
opens the same door: jitter, a cap, a retry budget, the thundering herd it prevents.
Say "the patience ramp" and nothing opens. The reader has to reverse-engineer the
concept from the code, alone, and then carry a private translation table for as long
as they work on the project. A coined term is a key cut for one lock, and only its
author holds it.

That cost is paid by every reader, forever, and it is silent. Nobody files a bug that
says "I lost an hour learning what a 'shepherd' is." The project just gets harder to
pick up, one private word at a time, and the cost grows across projects: the same
concept under four names in four repos is four things to learn instead of one. An LLM
reader pays twice. A standard term pulls in everything it knows about that concept,
its failure modes, and its known fixes. A coined term pulls in nothing, so it reasons
from scratch about a problem the field has already solved.

Bespoke words spread. An agent coins a word in one session, repeats it until it feels
established, and writes it into a ticket or a handoff. The next agent reads it as the
project's vocabulary and spreads it further. This skill is the cleanup, and it
matters most where the user doesn't know a standard term exists. Those words never
get fixed any other way.

## What to look for

Five shapes. The field might be software engineering, or any domain the project
lives in.

**Coinage**: a new word for a concept that already has a name.

| Bespoke | Standard |
|---|---|
| `RetryRamp`, "patience ramp" | exponential backoff |
| `Shepherd` (restarts crashed workers) | supervisor |
| "drift fixer" (compares desired and actual state, converges them) | reconciliation loop, reconciler |
| `FanCollector` (sends to N services, gathers the replies) | scatter-gather |
| "tripwire" that stops calling a failing dependency | circuit breaker |
| "volume flattener" in an audio chain | dynamic range compressor |

**Misuse**: a standard term given a meaning it doesn't have. This is worse than
coinage. The key opens the wrong door, and the reader walks through it confidently,
carrying the wrong knowledge.

| Misused | What it means | What was meant |
|---|---|---|
| "idempotent" for "same input, same output" | applying it twice equals applying it once | deterministic, pure |
| "refactor" for a change in behavior | restructuring that preserves behavior | rewrite, change, fix |
| "deprecated" for "removed" | still present, discouraged, scheduled for removal | removed |
| "race condition" for any intermittent failure | outcome depends on the timing of concurrent operations | flaky test, intermittent failure |

**Near-miss variant**: a homemade variant of a standard term. These are easy to wave
through because they're almost right. "Lane" for a kanban swimlane, "dead mail queue"
for a dead letter queue, "write-ahead journal" for a write-ahead log.

**Internal synonyms**: one concept with several names in the project. If "job", "task",
and "work item" all mean the unit a worker pulls from the queue, align on one: the
standard term if the field has one, otherwise the most widely used name.

**Unnamed concept**: a concept the code implements and the comments describe in a
sentence, but never call by its established name. "Keep the old version serving until
the new one passes its checks, then switch the traffic over" is a blue-green
deployment. The user may not know the term exists, which is why the skill exists.

## What counts as standard

A term is standard when you can point to it in established use with this meaning.
Where to look, from nearest to widest:

1. The vocabulary of the stack and the domain the project is built on. In Kubernetes
   it's "controller" and "reconcile"; in React it's "hydration"; in double-entry
   bookkeeping it's "journal entry" and "ledger".
2. Specs and standards bodies: IETF RFCs, POSIX, W3C, ISO, IEEE, language
   specifications, and the domain's own (AES for audio, GAAP and IFRS for accounting).
3. The field's canonical literature: the GoF patterns, Fowler's *Refactoring* and
   *Patterns of Enterprise Application Architecture*, *Enterprise Integration
   Patterns*, Meszaros's *xUnit Test Patterns*, *Domain-Driven Design*, Ousterhout's
   *A Philosophy of Software Design*, the Google SRE books, *Release It!*, and the
   domain's standard textbooks.
4. Broad established use: a Wikipedia article, major vendor documentation, the
   ecosystem's own docs.

The search test is the quick check: search for the term next to the field's name. If
the first page of results describes this concept, the term is standard. If it
describes a different concept, the term is misused. If it describes nothing, the term
is coined.

**When two established terms fit, prefer the one the project's audience will
recognize.** Choose recognition over precision drawn from a corner of the
literature. For engineers, "tactical vs. strategic programming" (Ousterhout) beats
"shifting the burden" (Senge's systems thinking), even though Senge's term fits the
dynamic more exactly. Put the runner-up in the item's ambiguity note so the user can
choose.

Your memory is not evidence on a close call. A model can produce a plausible term
that is itself a coinage, or one that means something else. Proposing one is the
same violation, with false authority behind it. When you are unsure, look it up
(WebSearch, the field's docs) before you call anything standard.

## Sounding invented is not being invented

In the session that wrote this skill, the agent listed "local minimum" and "carrying
cost" as the user's private coinages. They had shown up in the user's guidance next to truly
bespoke words, so they read as part of the same habit. Both are standard.
"Local minimum" comes from mathematical optimization and was used with its exact
meaning. "Carrying cost" comes from finance. The agent judged the words by their
company and by memory, without checking them. A term borrowed from another field and
used with that field's meaning is standard. Keep it.

Strangler fig, bulkhead, circuit breaker, canary release, tombstone, thundering herd,
transactional outbox, saga, dead letter queue, dogfooding, bikeshedding, and yak
shaving all sound whimsical, and all are standard. Flagging one of them replaces the
key everyone holds with your guess. That is the mirror image of the failure this
skill exists to stop. Judge a term by how it is used, not by how it sounds.

## When a bespoke word stays

A word stays when the field has no word for the concept. That's rare for technical
mechanisms and common for the business domain.

- **Domain terms come from the domain.** If the accountants say "journal entry", the
  code says `JournalEntry`.
- **Proper names are names, not terms.** The project's name, a service's name, or a
  product feature's name labels one specific thing. It doesn't pretend to describe a
  concept the field already has.
- **Names the project doesn't own** stay as their owner spells them: a library's API,
  a protocol's fields, a vendor's resource types.
- **A genuinely new concept** keeps its name. If that name is built from scratch, the
  proposal can suggest one built from standard parts ("bounded retry queue" beats
  "the patience pool").

Someone will cite domain-driven design's ubiquitous language as permission to invent:
"DDD says the team should build its own vocabulary." On its home turf that is
correct: the language of the *business domain*, drawn from the people who work in it.
It is the opposite of coinage, because the developers adopt the experts' words
instead of making up their own. It doesn't cover the technical mechanisms the field
has already named. DDD never asked anyone to rename exponential backoff.

"Naming is one of the two hard things," someone will say, as if that licenses any
name. It argues the other way. Naming is hard, and the field has spent decades
settling names for its recurring concepts. Using its settled name is how you avoid
that hard work.

An image may *explain* a term, but it doesn't *replace* it as the name. "A circuit
breaker trips like the one in your fuse box" helps a reader. A class named `FuseBox`
takes the key away from them.

## The workflow

### 1. Map the territory

- **Scope.** Scan the whole repository unless the user named a path. List the files
  (`git ls-files`, or `find` outside git). Drop vendored and third-party code,
  generated files, lockfiles, binaries, and build output.
- **Domains.** Read the README, the dependency manifests, and the top-level layout,
  then write a one-line list of the fields this project lives in. For example:
  "software engineering; Kubernetes operators; double-entry accounting". Every
  scanner gets this list, because a term is only bespoke relative to a field.
- **Shards.** Group the files by directory, so code about one concept stays together.
  Aim for roughly 40 files or 6,000 lines per shard.

### 2. Scan with Haiku subagents

Read `references/scan-prompt.md`. For each shard, fill in its placeholders with the
domain list and the shard's file paths, and launch it with the Agent tool:
`model: "haiku"`, `subagent_type: "general-purpose"`. Send all the shards in one
message so they run in parallel. For a large repository, send them in waves of about
ten. Each scanner returns a JSON array of leads.

A scanner's output is a lead, not a verdict. Haiku is fast and broad, and it will
flag standard terms and propose coinages of its own. Step 3 exists because of that.

### 3. Verify and consolidate

The quality of the recommendation is decided here, and so is its failure. You will
have a hundred leads in front of you, most of them plausible. You'll think: *"The
scanners already checked these; I'll merge the duplicates and present them."* That's
the moment. An unverified guess presented as the field's answer is exactly what this
skill exists to stop, and now it carries the skill's authority. Verify every lead
before it reaches the user.

- **Merge by concept, not by spelling.** Combine leads for the same term from
  different shards. Leads with different terms but the same `meaning_here` are
  internal synonyms, so merge them into one item.
- **Measure the spread.** Search the whole repository for every variant: case forms
  (camelCase, snake_case, kebab-case, SCREAMING_CASE), plurals, verb forms, and file
  and directory names. A scanner saw only its shard. Record the count, the files, and
  any public surface: a published API, a CLI flag, a config key, a wire format, or a
  metric that someone graphs.
- **Check the meaning.** Read the code behind each candidate and confirm the
  project's concept actually is the standard one. If it differs, the item is an
  ambiguity, not a rename.
- **Check the proposed term** against the sources above, and search on any close
  call. Drop leads that flag a standard term, and record them under "checked and
  left alone". Drop or replace any proposal that is itself a coinage.

### 4. Recommend

Present a numbered proposal, ordered by impact: the spread times how much the term
misleads. Each item follows this shape:

```
### 3. `Shepherd` → `Supervisor`   [coinage]
Meaning here: restarts worker processes that exit, with the policy set in shepherd.yaml.
Context:
  src/workers/shepherd.ts: `class Shepherd { watch(pid: number) { ... restartOnExit(pid) } }`
  docs/ops.md: "The shepherd restarts crashed workers within 5s."
Spread: 47 occurrences in 12 files. Variants: Shepherd, shepherd, SHEPHERD_RESTART_MS, shepherd.yaml
Public: config key `shepherd.restart_ms`, read by 2 deploy manifests (renaming it is a breaking change)
Evidence: Erlang/OTP supervisor; supervisord; systemd service supervision.
Ambiguity: none.
```

The context quotes the real lines from the files, enough for the user to judge the
item without opening them. When an item is ambiguous, say how in its **Ambiguity**
line, and give the options with your recommendation:

- Two standard terms fit (recognition vs. precision).
- The project's concept only partly matches the standard one.
- One bespoke word is used for two concepts, so it needs a split, not a rename.
- A public name would break its consumers.
- The evidence is thin.

After the items, add a short **Checked and left alone** list. These are terms that
looked bespoke but are standard, with a one-line reason each. It shows the user the
mirror-image failure was checked. Stop there and wait for the user's ruling.

### 5. Lock it in with the user

The user answers item by item: accept, reject, change the name, or ask about it.

- **Acceptance is explicit.** "Interesting" is not acceptance, and neither is
  silence. "Yes to 1–4 and 7" is.
- **A changed name is a new proposal.** Verify it the way step 3 does, show the
  evidence, and confirm it.
- **Ambiguous items need the user's ruling.** Bring the options and your
  recommendation. Don't settle it yourself.
- **After each round, show the whole proposal again,** with each item marked
  accepted, rejected, or open. That keeps the current state in front of both of you.

The locking ends when the user says to go ahead. Items still open at that point
aren't renamed.

The temptation arrives mid-iteration: *"Items 1 through 3 are obviously right, and
the user will accept them. I'll start those while they think about the rest."* Or it
arrives as standing guidance: *"Don't ask, resolve: I can write 'I recommend X', so
I've decided."* Refuse both. Here the user designed the gate themselves: renaming
waits on their acceptance. Your recommendation goes into the proposal. The decision
belongs to them, item by item. A rename they didn't accept touches every file the
word lives in, and someone then has to find it and undo it.

### 6. Rename the accepted items

- **Make one commit per concept,** so each rename can be reviewed and reverted on its
  own.
- **Cover every variant the spread found:** identifiers, files and directories
  (`git mv`), tests, docs, comments, config, log and metric names, and the project's
  own guidance files (CLAUDE.md, AGENTS.md, skills). The guidance files are where
  the next agent learns the vocabulary.
- **Rename by identifier, not by blind text replacement.** Review each hit, respect
  word boundaries, and leave the term alone where it appears inside another word or
  means something else.
- **Leave no alias,** no compatibility shim, and no "formerly X" comment. A
  half-finished rename gives one concept two names, which is worse than either name
  alone. History and changelogs keep the old name.
- **Rename a public name only if the user accepted the break,** as the proposal
  marked it.

### 7. Verify

- A search for every old variant returns nothing outside history and changelogs.
- The build and the tests pass.
- Report each renamed item with its commit, and anything accepted that wasn't done,
  with the reason.

## Done

The skill is finished when:

- the user has ruled on every item they wanted to rule on
- every accepted rename is committed, with its old variants searched down to zero
  and the tests passing
- no rename happened that the user didn't explicitly accept
