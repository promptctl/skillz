# Scanner prompt

Fill `{{DOMAINS}}` and `{{FILES}}`, then send everything below the rule, as-is, to
one Haiku subagent per shard.

---

You are scanning part of a codebase for vocabulary that departs from the standard
terminology of its fields, so that every concept has the name the industry already
uses. You report leads only. **Do not edit, create, rename, or delete any file.**

Project domains: {{DOMAINS}}

Read each of these files in full:
{{FILES}}

## Report

- **coinage**: an invented word for a concept with an established name. `Shepherd`
  restarting crashed workers is a "supervisor"; "patience ramp" for waits of 1s, 2s,
  4s is "exponential backoff".
- **misuse**: a standard term with a meaning it doesn't have. "Idempotent" meaning
  "same input, same output" (that's "deterministic").
- **near-miss**: a homemade variant of a standard term. "Dead mail queue" for dead
  letter queue; "lane" for swimlane.
- **synonyms**: several names in your files for one concept. "Job", "task", and
  "work item" all meaning the unit a worker pulls from the queue.
- **unnamed**: a concept described in a phrase whose established name never appears.
  "Keep the old version serving until the new one passes checks, then switch" is a
  blue-green deployment.

Look in identifiers that carry concepts (types, modules, functions, important
variables), file and directory names, CLI flags, config keys, metric and log names,
doc prose, and comments.

## Do not report

- **Standard terms that sound odd**: strangler fig, bulkhead, circuit breaker,
  canary, tombstone, thundering herd, outbox, saga, dead letter queue, bikeshedding,
  yak shaving, local minimum, carrying cost. A term borrowed from math, finance, or
  another field and used with that field's meaning is standard.
- **Names the project doesn't own**: third-party APIs, protocol fields, library
  types.
- **Proper names**: the project, a product, a service.
- **Business-domain words** the domain's own experts use.
- **Generic names** like `data`, `tmp`, `handler`. Vagueness is a different problem.

Three bad leads: `{"term": "circuit breaker", "kind": "coinage"}` flags a standard
term. `{"term": "Shepherd", "proposed": ["Herder"]}` proposes another invented word.
`{"term": "Shepherd", "meaning_here": "probably manages things"}` guesses instead of
reading the code.

## Output

Return only a JSON array, with no prose before or after it; `[]` if you find nothing.
Each element:

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

- `kind`: one of `coinage`, `misuse`, `near-miss`, `synonyms`, `unnamed`. For
  `synonyms`, every competing name goes in `variants`.
- `meaning_here`: what the term refers to in this code, one sentence, written from
  reading the code rather than the word.
- `proposed`: the established terms that fit, best first.
- `evidence`: where each proposed term is established (a book, a spec, platform
  docs). Write `"unsure"` if you aren't; an honest "unsure" is useful and a made-up
  source is not.
- `locations`: up to three of the clearest occurrences, with the exact line as the
  snippet.
- `confidence`: `high`, `medium`, or `low`. Report medium and low leads too; they
  get checked later.
