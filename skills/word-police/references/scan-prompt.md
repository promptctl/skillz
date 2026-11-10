# Scanner prompt

Fill `{{DOMAINS}}` and `{{FILES}}` in the prompt below, then send it as-is to one Haiku
subagent per shard.

---

You are scanning part of a codebase for vocabulary that departs from the standard
terminology of its fields. The goal is a project where every concept has the name the
industry already uses, so anyone from the field can read it on sight. You report
leads. **Do not edit, create, rename, or delete any file.**

Project domains: {{DOMAINS}}

Your files. Read each one in full:
{{FILES}}

## What to report

Report a term when it fits one of these:

- **coinage**: an invented word for a concept that already has an established name.
  For example, `Shepherd` for a process that restarts crashed workers is a
  "supervisor"; "patience ramp" for retry waits of 1s, 2s, 4s is "exponential
  backoff".
- **misuse**: a standard term used with a meaning it doesn't have. For example,
  "idempotent" used to mean "same input, same output" (that's "deterministic").
- **near-miss**: a homemade variant of a standard term. For example, "dead mail queue"
  for a dead letter queue, or "lane" for a swimlane.
- **synonyms**: two or more names in your files for the same concept. For example,
  "job", "task", and "work item" all meaning the unit a worker pulls from the queue.
- **unnamed**: code or comments describe a concept in a phrase, and the concept has an
  established name that never appears. For example, "keep the old version serving
  until the new one passes checks, then switch" is a blue-green deployment.

Look in identifiers that carry concepts (types, modules, functions, important
variables), file and directory names, CLI flags, config keys, metric and log names,
doc headings and prose, and comments.

## What not to report

- **Standard terms that sound odd.** Strangler fig, bulkhead, circuit breaker, canary,
  tombstone, thundering herd, outbox, saga, dead letter queue, bikeshedding, yak
  shaving, local minimum, carrying cost. Terms borrowed from math, finance, or
  another field count as standard when they're used with that field's meaning.
- **Names the project doesn't own:** third-party APIs, protocol fields, library
  types.
- **Proper names:** the project's name, a product's name, a service's name.
- **Business-domain words** that the domain's own experts use.
- **Generic names** like `data`, `tmp`, or `handler`. Vagueness is a different
  problem.

Bad leads. Do not produce these:

```json
{"term": "circuit breaker", "kind": "coinage", ...}
```
The term is standard. Flagging it is wrong.

```json
{"term": "Shepherd", "proposed": ["Herder"], ...}
```
The proposal is another invented word.

```json
{"term": "Shepherd", "meaning_here": "probably manages things", ...}
```
The meaning is a guess, not read from the code.

## Output

Return only a JSON array, with no prose before or after it. Return `[]` if you find
nothing. Each element has this shape:

```json
{
  "term": "Shepherd",
  "variants": ["Shepherd", "shepherd", "SHEPHERD_RESTART_MS", "shepherd.ts"],
  "kind": "coinage",
  "meaning_here": "Watches worker processes and restarts any that exit.",
  "proposed": ["supervisor"],
  "evidence": "Erlang/OTP supervisor behaviour; supervisord",
  "locations": [
    {"file": "src/workers/shepherd.ts", "line": 12, "snippet": "export class Shepherd {"}
  ],
  "confidence": "high"
}
```

- `kind` is one of `coinage`, `misuse`, `near-miss`, `synonyms`, `unnamed`. For
  `synonyms`, put every competing name in `variants`.
- `meaning_here` is what the term refers to in this code, in one sentence. Write it
  from reading the code, not from the word.
- `proposed` lists the established terms that fit, best first.
- `evidence` names where each proposed term is established: a book, a spec, or
  platform docs. If you aren't sure, write `"unsure"`. An honest "unsure" is useful.
  A made-up source is not.
- `locations` gives up to three of the clearest occurrences, with the exact line as
  the snippet.
- `confidence` is `high`, `medium`, or `low`. Report medium and low leads too; they
  get checked later.
