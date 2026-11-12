# First slice

Status: Proposed. Date: 2026-11-12.

## What it is

Convert ten to fifteen existing scout runs into records, then read them together for one hour.

- **Corpus:** ten to fifteen scout runs from the maintainers' fleet. Scouts are the most uniform
  task class, and each has a brief, a report and a trace. Use the one trace format that holds the
  most scout runs, so only one adapter is needed.
- **Build:** one converter that turns a run into one JSONL record with the fields `task_id`,
  `brief`, `report`, `condensed_trajectory` and `missing_fields`. The records feed an existing
  error-discovery skill (from `ai-evals-course/evals-skills`) unchanged, so the method is tested
  before any tool is built.
- **Session:** a person reads for one hour and writes a free-text note per run. Notes come
  first; an agent groups them afterwards, and the person names the failure modes.

The slice is done when it yields at least one named failure mode the reader did not already
know, and that leads to a named change (a brief rule, a skill edit or a harness default) within
two weeks.

## The falsification test

For each run, the reader first reads only the brief and the report and notes any failures, and
only then opens the trace. If the trace yields at most one failure mode not already visible in
the brief and report, then for this fleet the outcome artifact is the eval surface and the trace
is not. The harness would then be aimed at the wrong object.

Two further ways the slice can fail:

- The hour of reading does not happen within a week of being offered. The human-in-the-loop
  design then fails in its home setting.
- The session names failure modes but none changes anything within two weeks. That is the
  "evals that never changed a decision" pattern.

## Out of scope

Judges, CI gates, dashboards, the product-agent setting, chat paste, synthetic data, repeated
trials and consistency scoring, a new review UI, an `init-evals` skill, and every trace adapter
except the one. All of these wait until the slice produces a decision. If the existing
error-discovery skill turns out to be enough, the outcome is to use it and build nothing more.
