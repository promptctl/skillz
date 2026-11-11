---
name: word-police
description: Realign a project that has grown its own bespoke vocabulary with the standard terminology of its fields - software engineering first, plus whatever domains the project lives in (finance, audio, networking, ...). Fans Haiku subagents across the codebase to find invented words, misused standard terms, near-miss variants, internal synonyms, and unnamed concepts; and verifies each against established usage. Renames only what the user accepts. Use when the user says "word police", "align our terminology", "we're inventing words", "what's the standard term for this", "audit the naming", "rename to standard terms", or wants a codebase's vocabulary brought in line with its industry. Optionally scoped to a path.
---

# Word police

Bring a project's vocabulary in line with the standard terminology of its fields, so
a concept has the same name here as in the field's literature and someone from the
industry recognizes it on sight. The skill finds the bespoke words and proposes the
standard ones.

**It renames only what the user accepts.** Until then everything is read-only.

## Why the words matter

A standard term is a key the reader already carries. "Exponential backoff" opens the
same door for every engineer, search engine, and model: jitter, a cap, a retry
budget, the thundering herd it prevents. "The patience ramp" opens nothing, so the
reader reverse-engineers the concept from the code and carries a private translation
table for as long as they work here. The cost is silent and paid by every reader; an
LLM reader pays twice, because a coined term pulls in none of what it knows about the
concept's failure modes and fixes.

Bespoke words spread. An agent coins one, repeats it until it feels established,
writes it into a ticket, and the next agent reads it as the project's vocabulary.
The skill matters most where the user doesn't know a standard term exists, because
those words never get fixed any other way.

## What to look for

Five shapes, in software engineering or any domain the project lives in.

- **Coinage**: a new word for a concept that has a name. `Shepherd` that restarts
  crashed workers is a supervisor; "drift fixer" that converges desired and actual
  state is a reconciliation loop; a "volume flattener" in an audio chain is a
  dynamic range compressor.
- **Misuse**: a standard term with a meaning it doesn't have. Worse than coinage:
  the key opens the wrong door and the reader walks through it confidently.
  "Idempotent" for "same input, same output" (that's deterministic); "refactor" for
  a change in behavior; "deprecated" for "removed".
- **Near-miss**: a homemade variant of a standard term, easy to wave through because
  it's almost right. "Dead mail queue" for dead letter queue, "write-ahead journal"
  for write-ahead log, "lane" for swimlane.
- **Internal synonyms**: one concept, several names. If "job", "task", and "work
  item" all mean the unit a worker pulls from the queue, align on one: the standard
  term if the field has one, otherwise the most used name.
- **Unnamed concept**: the code implements it and the comments describe it in a
  sentence, but never by its name. "Keep the old version serving until the new one
  passes its checks, then switch the traffic" is a blue-green deployment. The user
  may not know the term exists.

## What counts as standard

A term is standard when you can point to it in established use with this meaning.
From nearest to widest: the vocabulary of the project's stack and domain (Kubernetes
"reconcile", React "hydration", bookkeeping "ledger"); specs and standards bodies
(IETF RFCs, POSIX, W3C, AES, GAAP); the field's canonical literature (GoF, Fowler,
*Enterprise Integration Patterns*, *Domain-Driven Design*, the Google SRE books,
*Release It!*, the domain's textbooks); broad established use (Wikipedia, major
vendor docs).

The quick check: search the term next to the field's name. If the first page
describes this concept, the term is standard. A different concept, misused. Nothing,
coined.

When two established terms fit, prefer the one the project's audience will recognize
over the more precise one from a corner of the literature ("tactical vs. strategic
programming" over Senge's "shifting the burden"), and put the runner-up in the item's
ambiguity note.

Your memory is not evidence on a close call. A model can produce a plausible term
that is itself a coinage, and proposing it is the same violation with false
authority behind it. When unsure, look it up (WebSearch, the field's docs) before
calling anything standard.

## Sounding invented is not being invented

The mirror-image failure is flagging a standard term because it sounds whimsical or
sits next to bespoke words. The session that wrote this skill listed "local minimum"
(optimization) and "carrying cost" (finance) as coinages because they appeared beside
real ones; both are standard, used with their home field's meaning. Strangler fig,
bulkhead, circuit breaker, canary release, tombstone, thundering herd, saga, dead
letter queue, bikeshedding, and yak shaving are all standard. Judge a term by how it
is used, not by how it sounds.

## When a bespoke word stays

- **Domain terms come from the domain.** If the accountants say "journal entry", the
  code says `JournalEntry`. That is what DDD's ubiquitous language means: developers
  adopt the experts' words. It never licensed renaming exponential backoff.
- **Proper names** (the project, a service, a product feature) label one thing and
  don't claim a concept the field already has.
- **Names the project doesn't own** stay as their owner spells them: a library's
  API, a protocol's fields, a vendor's resource types.
- **A genuinely new concept** keeps its name. If the name is built from scratch,
  suggest one built from standard parts ("bounded retry queue" over "the patience
  pool").

"Naming is hard" argues for the settled name, not against it. An image may explain a
term ("a circuit breaker trips like the one in your fuse box"), but a class named
`FuseBox` takes the key away.

## Workflow

### 1. Map the territory

- **Scope**: the whole repository unless the user named a path. List files with
  `git ls-files` (or `find` outside git); drop vendored and generated code,
  lockfiles, binaries, and build output.
- **Domains**: from the README, dependency manifests, and top-level layout, write a
  one-line list of the fields the project lives in ("software engineering;
  Kubernetes operators; double-entry accounting"). Every scanner gets it, because a
  term is only bespoke relative to a field.
- **Shards**: group files by directory, roughly 40 files or 6,000 lines each.

### 2. Scan with Haiku subagents

Read `references/scan-prompt.md`, fill its placeholders for each shard, and launch
each with the Agent tool (`model: "haiku"`, `subagent_type: "general-purpose"`). Send
all shards in one message so they run in parallel, in waves of about ten for a large
repository. Each returns a JSON array of leads.

A lead is not a verdict. Haiku is fast and broad, and it will flag standard terms and
propose coinages of its own.

### 3. Verify and consolidate

This step decides the quality of the recommendation. With a hundred plausible leads
in front of you, the thought will be *"the scanners already checked these; I'll merge
the duplicates and present them."* An unverified guess presented as the field's
answer is what this skill exists to stop, and now it carries the skill's authority.
Verify every lead.

- **Merge by concept, not spelling.** The same term across shards is one item;
  different terms with the same `meaning_here` are internal synonyms, one item.
- **Measure the spread.** Search the whole repository for every variant (case forms,
  plurals, verb forms, file and directory names); a scanner saw only its shard.
  Record the count, the files, and any public surface: a published API, a CLI flag,
  a config key, a wire format, a graphed metric.
- **Check the meaning.** Read the code and confirm the project's concept is the
  standard one. If it differs, the item is an ambiguity, not a rename.
- **Check the proposed term** against the sources above, searching on any close
  call. Move leads that flag a standard term to "checked and left alone"; drop or
  replace any proposal that is itself a coinage.

### 4. Recommend

A numbered proposal ordered by impact (spread times how much the term misleads), each
item in this shape:

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

Context quotes real lines, enough to judge the item without opening the files. The
**Ambiguity** line gives the options and your recommendation when two standard terms
fit, the concept only partly matches, one word covers two concepts (a split, not a
rename), a public name would break its consumers, or the evidence is thin.

After the items, a short **Checked and left alone** list: terms that looked bespoke
but are standard, one-line reason each. Then stop and wait for the user's ruling.

### 5. Apply

Ask the user which changes, if any, they want to accept. Make the changes the user accepts.

### 6. Verify

A search for every old variant returns nothing outside history and changelogs, and
the build and tests pass. Report each rename made, and anything accepted but not
done, with the reason.

## Done

- The user has ruled on every item they wanted to rule on.
- Every accepted rename is made, its old variants searched down to zero, tests
  passing.
- No rename happened that the user didn't accept.
