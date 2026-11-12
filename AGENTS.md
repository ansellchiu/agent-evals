# Repository rules

This file is the single source of truth for how agents work in this repository.
`CLAUDE.md` imports it; edit this file, never the pointer.

## Scope

Evidence base and harness for agent evaluation practice. This repository holds the *evidence* -
what is established about evaluating agents, with sources and dates - plus a harness once a real
trace exists to run it on.

It is deliberately not a skills repository. The skills that come out of this evidence
(`/draft-agent-eval` first) publish to the `agent-skills` monorepo. The monorepo cites this
repository; it never depends on it.

## The rule this repository exists to hold

Read real runs before writing any metric. Use a deterministic check wherever a rule can express
the failure. Keep an automated judge narrow, validated against human labels, and treat it as a
regression floor rather than a detector - the strongest 2026 evidence is that judges miss most
real defects. Keep a human and production sampling permanently in the loop.

## Scope boundary

Discovery - reading runs, taking notes, grouping them, naming failures - works on any harness,
including a pasted chat transcript. Measurement - gates, repeated trials, pass rates - needs runs
that can be re-executed, so it only works where the agent can be run again.

## This repository is public

Methods, sanitised examples and public-source evidence only. No raw fleet traces, no captain
conversation, no customer data, no credentials. Extract the pattern; leave the source behind.

## Evidence conventions

Every claim carries its date and its source. A claim from a vendor's own engineering post is
first-party; a blog summarising it is secondhand and must be labelled. Recency matters: prefer
the current year and mark older material as background rather than current practice.
