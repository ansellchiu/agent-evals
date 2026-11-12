# agent-evals

Evidence base and harness for agent evaluation practice.

This repository holds the *evidence*: what is established about evaluating agents, with
sources and dates, plus - when a real trace exists to run it on - a harness.

It is deliberately not a skills repository. The skills that come out of this evidence
(`/draft-agent-eval` first) publish to the `agent-skills` monorepo as `skills/<name>/`.
The monorepo cites this repository; it never depends on it.

## Contents

- `research/2026-11-agent-eval-practice.md` - the November 2026 landscape scout:
  eval kinds and what each can prove, how practitioners find unanticipated failures,
  the failure modes with primary sources, and what remains unsettled.

## The rule this repository exists to hold

Read real runs before writing any metric. Use a deterministic check wherever a rule can
express the failure. Keep an automated judge narrow, validated against human labels, and
treat it as a regression floor rather than a detector - the strongest 2026 evidence is
that judges miss most real defects. Keep a human and production sampling permanently in
the loop.
