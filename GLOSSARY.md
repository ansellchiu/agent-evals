# Glossary

This repository is the authority for these words. Other repositories that cite it use them in
these senses. Definitions are short on purpose; the evidence behind them is in `research/`.

**Trace** - The full record of one run of an agent: input, system prompt, model calls, tool
calls, intermediate steps and final output. Anthropic calls the same thing a *transcript*; OpenAI
calls it a *trace*. A trace shows what the agent saw and did, not whether what it saw was true.

**Grader** - Anything that scores a run or part of a run. There are three kinds: code-based
(a rule), model-based (an LLM) and human.

**Judge** - A model-based grader. In this repository a judge covers one failure, is validated
against human labels (true-positive and true-negative rate, not just agreement), and is never
assumed to see a defect class it was not built for.

**Error analysis** - Reading real runs, writing free-text notes, grouping the notes into
failure modes and naming them. A human does the noticing and the naming; an agent may help
organise. Also called *error discovery*.

**Regression floor** - What an automated check or judge is good for: telling you a known
failure came back. It is not a detector for new failures, and a green floor says nothing about
defects nobody has named yet.

**Discovery** - The half of the loop that finds out what is going wrong: read runs, take notes,
group them, name the failures, decide what to change. It needs runs to read, not the ability to
run the agent again, so it works on any harness, including a pasted chat transcript.

**Measurement** - The half of the loop that counts: gates, repeated trials, pass rates, judge
calibration. It needs runs that can be re-executed in a controlled environment, so it does not
work on a transcript someone pasted.

**Pass rate versus consistency** - Pass rate (pass@k) is the chance an agent succeeds at least
once in k tries; it suits tools where one success matters. Consistency (pass^k) is the chance it
succeeds on all k tries; it suits agents that must be reliable. A 75% per-trial pass rate gives
about 42% consistency over three trials.
