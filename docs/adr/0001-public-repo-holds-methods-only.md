# 0001. This public repository holds methods only

Status: Accepted. Date: 2026-11-12.

## Decision

This repository is public. It holds methods, sanitised examples and public-source evidence.
It never holds raw fleet traces, conversations with the people who run the fleet, customer
data or credentials.

## Why

The repository covers fleet-first work: evaluating agents that run in a fleet its maintainers
operate. The material that work is built from is the fleet's own traces. A trace is the full
record of a run, so it can contain private conversation, customer data, file contents,
environment details and credentials. A public repository cannot hold what a fleet trace
contains, and redacting a trace by hand is not a reliable way to make one safe.

The line is therefore drawn at the kind of material, not at how carefully it was cleaned.
Extract the pattern from a run and leave the source behind.

## Consequences

- Examples are written from scratch or reduced to a pattern. A sanitised example shows the
  shape of a failure, not a real run.
- Evidence is public and dated, with its source. A claim from a vendor's own engineering post is
  first-party; a blog summarising it is secondhand and is labelled so.
- Any harness code in this repository is tested on sanitised or synthetic inputs. Real traces
  stay where they were produced.
- Skills that come out of this work publish to the `agent-skills` monorepo. The monorepo cites
  this repository and never depends on it, so nothing private is pulled in through a link.
