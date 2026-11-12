# 0002. Discovery is portable; measurement is not

Status: Accepted. Date: 2026-11-12.

## Decision

Discovery works on any harness, including a pasted chat transcript. Measurement needs runs that
can be re-executed, so it only works where the agent can be run again. The repository promises
discovery everywhere and measurement only where re-running is possible. See `GLOSSARY.md` for
both words.

## The constraint that gave way

The original goal was one eval approach that works across settings, "ideally even in contexts
with no harness at all". That constraint holds for discovery and gives way for measurement.

It gives way because the two halves depend on different things:

- Discovery's active ingredient is a person reading real runs. The 2026 evidence calls this the
  only method that found unexpected failures in any study read. It is harness-agnostic by nature.
  Only ingestion differs: each source (fleet logs, product spans, a pasted transcript) is
  converted into one common run record, with the fields it lacks listed.
- Measurement needs a place to run the agent again: repeated trials (consistency, not just pass
  rate), a controlled environment (infrastructure alone moved one benchmark by 6 points), a
  regression gate and a judge held-out set that persists. A pasted transcript has none of these.
  Promising a gate there would ship a green check that measures nothing.

A user with only a transcript therefore gets an error-analysis session that ends in a named
failure and a decision, and is told plainly that they cannot get a regression gate.

## Consequences

- One discovery method, with an input adapter per source, and one shared vocabulary.
- Measurement features are specific to the setting that can re-run the agent.
- Do not make agnosticism depend on a trace standard that is still moving. OpenTelemetry GenAI
  conventions were still in Development status in 2026 (secondary sources, November 2026), so
  the repository uses its own minimal run record.
- Unverified: whether official chat exports include tool calls, search results or reasoning.
  No official schema was found (November 2026). Until checked, treat chat as discovery over
  final text.
