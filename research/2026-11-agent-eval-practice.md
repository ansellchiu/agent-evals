# Agent-eval practice as of November 2026: scout report for `/draft-agent-eval`

Task: rr-agent-eval-market-s1. Scout only. No code, PR, or project change. Report date: 2026-11-12.

## 0. How to read this report

**Method.** I used WebSearch (standard mode) and WebFetch, plus `gh api` for repo existence. Exact queries are listed in Appendix A. Both tools return summaries made by a small model, not verbatim page text. So quotation marks below mean "the tool returned this as a quote". I did not see raw pages. Where a figure came only from a search-result summary and not from a fetched page, I mark it **[secondhand]**. Where a fetched page contradicted a search summary, I say so.

**Evidence tiers used below.**
- **P** = first-party or primary (vendor's own engineering post, official docs, the paper itself, the repo itself).
- **S** = secondhand (blog, aggregator, or search snippet about a primary source).
- **V** = vendor marketing blog. Treat as one team's view.

**Recency.** Every source is dated by month. Anything before 2026 is labelled *background*.

**What I could not do.**
- The starting article is paywalled from "Step 3" onward. I read only the free portion. I cannot judge its clustering and prioritisation step, its fix step, or its Shopify, Cursor, Ramp and Harvey examples. See §5.
- OpenAI's own SWE-bench post returned HTTP 403 to me. Its date is therefore from secondary sources, and those disagree (§3.4).
- I did not run any provider-key search (`fm-av-run.sh`). WebSearch was the live provider. I did not need keyed providers for this scope.

**Headline finding, in one paragraph.** The 2026 consensus among first-party sources is stable and fairly narrow. (1) Read real traces before you write any metric. (2) Use deterministic checks wherever a rule can express the failure, and keep LLM judges narrow, one failure each, validated against human labels. (3) Treat any automated judge as a regression floor, not a detector. The strongest 2026 evidence is that judges miss most real defects. (4) Keep human review and production sampling permanently in the loop. The article's central claim, "error discovery first", is **corroborated** by every first-party source I found. Its claim that coding agents can accelerate review is **one team's experience**: there is no independent evidence either way. Its title promises "and fix". The free portion covers finding only.

---

## 1. Question 1: what kinds of eval exist for agents, and what is each for?

Anthropic (January 2026, **P**) names three grader types for agents: code-based, model-based, human. It adds a stack of whole-system methods around them (production monitoring, A/B tests, user feedback, transcript review) and says "no single evaluation layer catches every issue" (its "Swiss cheese" framing). The table extends that taxonomy with the other kinds the captain asked about. For each kind: what it can prove, and what it silently cannot.

| Kind | What it can prove | What it silently cannot | Best source (month) |
|---|---|---|---|
| **Deterministic / unit-style check** (string match, JSON parses, tool called with right args, final environment state) | A named rule holds, cheaply and repeatably. Good for CI. | Anything outside the rule. Anthropic: "Brittle to valid variations that don't match expected patterns exactly" and lacks nuance. A rigid grader rejected "96.12" when "96.124991…" was expected, and Opus 4.5's CORE-Bench score rose from 42% to 95% after grader fixes. | Anthropic, Jan 2026 (P) |
| **Exact trajectory match** (expected tool sequence) | The agent took the one path you wrote down. | That a different path was wrong. Google ADK defaults `tool_trajectory_avg_score` to EXACT match and offers IN_ORDER and ANY_ORDER. ADK's own docs note the metric needs reference data. The tool-use-quality alternative needs an LLM judge and rubrics. | Google ADK criteria page (undated, retrieved Nov 2026) (P) |
| **Scored rubric eval** (weighted or binary checklist, often graded by code and by a model together) | A multi-dimension quality bar, tracked over time. | Failures that have no rubric line. Catching One in Five (June 2026): the production judge had only three axes (intent, brand voice, personalization) and "no category for state-tracking, guardrails, or recovery, where most defects … cluster". 113 of 114 rounds whose notes described a confirm-gate or cart-state defect were scored "brand voice". | arXiv 2606.10315, Jun 2026 (P) |
| **LLM-as-judge** | Agreement with humans on a defined, narrow, single-failure criterion, if validated. Scales. | Defects it was never routed to see. Same paper: the judge caught 2 of 9 systematic patterns in batch 1, and its operational gate flagged **0 of 100** rounds while humans confirmed 23 distinct defects. The authors say reliability "is almost always reported as agreement with human ratings, not recall of real defects". Also non-deterministic and needs calibration (Anthropic, Jan 2026). | arXiv 2606.10315 (Jun 2026); Anthropic (Jan 2026) |
| **Human review** (expert annotation, spot checks) | Ground truth, and the only way to find failures nobody listed. Calibrates the judges. | Does not scale. Anthropic: expensive and slow. Hamel (Jun 2026): "the most human-intensive part of evals". | Anthropic, Jan 2026; hamel.dev note, Jun 2026 |
| **Trace / span inspection** | *Why* it failed: wrong tool, loop, bad handoff, policy breach. OpenAI's docs frame trace grading as the debugging stage, before datasets: "A trace captures the end-to-end record of model calls, tool calls, guardrails, and handoffs for one run." | Frequency. One trace shows a failure exists, not how common it is. OpenAI's guide says nothing about grader reliability (checked: "The page doesn't discuss grader reliability, calibration, or known limitations."). | OpenAI agent-evals guide (undated page; retrieved Nov 2026) (P) |
| **Red-teaming / adversarial probing** | An exploit is possible. Hamel's FAQ (updated Sep 2026): "prove that you can trigger the error at least once by red-teaming" before building an eval for it. | Robustness. AutoDojo (June 2026, arXiv 2606.15057) reports defences that look strong against static attacks collapse under adaptive ones: PIGuard 0.0% → 28.0% attack success, DataFilter 12.6% → 33.4%. "Static benchmarks do not usefully evaluate defense robustness to adaptive threats." | arXiv 2606.15057, Jun 2026 (P) |
| **Online / production signals** (monitoring, sampled-trace judges, A/B, user feedback) | What real users do, at scale. Drift. Anthropic lists production monitoring as the way to see real behaviour. | Ground truth. Anthropic: production monitoring is "reactive and lacks ground truth"; user feedback is "sparse, self-selected, and skews toward severe issues"; A/B is slow and gives little "why". | Anthropic, Jan 2026 (P) |
| **Behaviour-auditing agents** (Petri, Bloom) | Frequency and severity of one behaviour across many generated scenarios. Bloom correlated with hand labels: Spearman 0.86 for Claude Opus 4.1, 0.75 for Sonnet 4.5, over 40 hand-labelled transcripts. | Anything the seed does not describe. Bloom's own page notes that in 1 of 10 "quirk" tests (self-promotion) manual review found the baseline showed similar rates. Anthropic's workflow advice (secondhand): run Petri first to find instances, then Bloom to measure how widespread. **Background**: Bloom is December 2025. | anthropic.com/research/bloom, Dec 2025 (P, background) |

**Adoption data (a reality check, not evidence of efficacy).** LangChain's State of Agent Engineering (survey run 18 Nov – 2 Dec 2025, n=1,340, **P**, vendor survey): 89% have some observability; 62% detailed tracing; **52.4% run offline evals; 37.3% run online evals**; 59.8% use human review; 53.3% use LLM-as-judge; quality is the top production blocker at 32%. Tracing is common. Systematic evaluation is roughly half as common.

**What changed since 2025 (for this question).**
1. Agent-specific eval vocabulary is now settled across vendors: *task / trial / grader / transcript / outcome* (Anthropic), *trace grading* (OpenAI), *eval case + user simulation + criteria* (Google ADK, which now lists 17 criteria including efficiency counters that "never pass or fail" a case).
2. The "pass^k" idea has become standard advice: "pass@k for tools where one success matters, pass^k for agents where consistency is essential" (Anthropic, Jan 2026). A 75% per-trial rate gives ~42% pass^3.
3. Trace-first debugging is now product surface in OpenAI's Agent Builder docs, not just a practitioner habit.

**Where the sources agree on when a unit-style check is the wrong tool** (these points come straight from the sources, not from my judgement):
- If the failure is a judgement call (tone, whether the agent should have asked a clarifying question), a deterministic check will either miss it or reject valid variations. Hamel's FAQ: "Use a code-based eval when an objective rule can identify the failure … Use an LLM judge when the failure requires human judgment."
- If you do not yet know what the failure is, no check of any kind is the right tool yet. You need traces and a human (Q2).
- Don't use a slow judge as an inline guardrail. Guardrails are "fast and deterministic"; evaluators run after generation and "do not block the original answer" (Hamel FAQ, Sep 2026).
- If the output space legitimately has several right paths, exact-trajectory scoring punishes correct behaviour. The Opus 4.5 τ²-bench example in Anthropic's post found a policy loophole that was a "better solution for the user" yet counted as a failure.

---

## 2. Question 2: how do credible practitioners find failures nobody has thought to test for?

| Method | Evidence it works | Evidence it fails or is untested |
|---|---|---|
| **Read real traces, annotate, group into failure modes ("error analysis", now "error discovery")** | Unanimous first-party recommendation: Hamel FAQ (updated 21 Sep 2026): "the most important activity in evals … Do not skip error analysis." Anthropic (Jan 2026): "Read the transcripts!" and "20-50 simple tasks drawn from real failures is a great start." OpenAI's guide: start with traces. Arize and Tianpan blogs agree (**S/V**, 2026). | **No controlled study** comparing it with alternatives exists in anything I found. The evidence is practitioner consensus plus the failure of the alternatives (§3). Hamel's own "50+ companies" is a course claim, not a result. |
| **Review volume and stopping rule** | Hamel FAQ: start with ~100 diverse traces, personally annotate at least the first 30, continue "until new traces stop revealing failure modes or changing existing ones"; review 10–20 traces weekly, at least 100 fresh per cycle, always with some random traces. | The numbers (30, 100, 10–20) are one team's heuristics. No source gives a statistical basis. The article (Sep 2026) says "about 100" and "at least 10 yourself" first: the thresholds differ between the two Hamel pieces published a few days apart (30 vs 10). Treat as rules of thumb. |
| **Simulated users from dimension tuples** (task × user type × request clarity) for launch-stage products | Article (free part) and Hamel FAQ agree. The FAQ adds the limit: "Synthetic data cannot tell you how common a failure is in production," and it is less reliable in specialised domains, low-resource languages, and high-stakes settings. | Only the limit is documented. No measure of how many real failures synthetic tuples recover. |
| **Agent-assisted annotation** (the article's core new claim; "Error Discovery" skill) | Hamel's note (24 Jun 2026): the agent keeps the taxonomy updated, rechecks old records for each new pattern, and picks next samples by active learning; a human annotates. Repo exists and is active (see Appendix B). | **No numbers.** I checked the note: "The page contains no quantitative results, benchmarks, or case data about the method's effectiveness or time savings." The skills post (published 2 Mar 2026, modified 31 Aug 2026) cites no benchmark either: "The page gives no benchmarks, results, or independent validation." The article itself says agents "flag good responses as failures" and miss judgement-dependent failures. So the vendor-neutral position is: unmeasured. |
| **Automated clustering of traces** (Braintrust Topics, LangSmith, Galileo, Arize Phoenix; embedding clustering of critiques) | Tools exist. Braintrust says Topics needs ≥100 trace summaries before generating clusters (**V**). Atla's post (search snippet only; the page returned 404 when fetched, so **S**) said embedding clustering of critiques gave more specific failure patterns than stuffing critiques into an LLM context, and that failure modes shift when prompts or models change. | Everything is vendor material. No head-to-head against human review. |
| **Seeded faults** (plant a known defect, see whether the judge or test notices) | Useful *for validating the checker*, not for discovering product failures. A practitioner guide proposes "mutation testing for LLM judges" (**V**, agentpatterns.ai). arXiv 2607.06873 (8 Jul 2026, **P**) builds a black-box boundary tester that covers "23–38 distinct boundaries per agent" vs 12 for a prompt-only baseline on τ³-bench agents. | **Correction to a search summary.** The search tool claimed "the judge caught 22 of 27 seeded faults (0.81)" from that paper. When I fetched the abstract, that figure was **not** present ("Seeded faults caught by a judge: not reported in the page content"). I do not cite it. Another study cited by the search tool (preprints.org) says judges show "little reliable discrimination" between injected-fault cause types (**S**, unverified). |
| **Adversarial probing / red-teaming** | Finds a different class of failure (security, manipulation). Gray Swan / UK AISI public competition, Mar 2026 (**S**, not fetched): 13 frontier models, 272,000+ attacks, attack success 0.5% (lowest) to 8.5% (highest). AutoDojo (Jun 2026): adaptive attacks beat static ones (above). | Not a method for finding ordinary product failures. Results from different benchmarks are not comparable (the 0.5% and the 81% figures I saw come from different benchmarks). |
| **Judge-assisted discovery** | The Catching One in Five authors' own method was *exhaustive human transcript review*, which found "7 new cross-cutting patterns" the judge never flagged. | Implies the judge is a poor discoverer. It is a regression floor: "automated judging is a regression floor, not a substitute for human review". |
| **Behaviour-generation agents (Petri/Bloom)** | See table in §1. Both are repo-active (Appendix B). | Aimed at model behaviour/alignment, not product UX failures. Not tested on a product-agent failure-discovery task in anything I found. |

**What the evidence says about "how well each works."** Honest answer: only one thing has real support, and it is negative. Automated judges alone discover very little (22% recall of systematic patterns in one deployed agent, June 2026; 0 of 100 flagged by its gate). Human review of real traces is the only method that found the unexpected failures in any study I read. Agent-assisted review is plausible but unmeasured. Clustering and seeded faults are tooling and calibration aids, not validated discovery methods.

**Criteria drift is real and has primary evidence (background).** Shankar et al., "Who Validates the Validators?" (UIST 2024, arXiv 2404.12272, **P**, background): users "need criteria to grade outputs, but grading outputs helps users define criteria". Participants changed criteria after more grading, even revising earlier grades. This is the paper behind the article's "criteria drift" claim and it was published two years ago. It supports the article's point that you cannot fully specify criteria to an agent in advance. It does *not* show that drift happens to LLM judges over time (that is a different phenomenon; see §3.1).

---

## 3. Question 3: what breaks in practice, and the strongest primary source for each

### 3.1 Judge drift and judge unreliability

- **Strongest primary:** Catching One in Five (arXiv 2606.10315, **June 2026**, **P**). A deployed production judge caught 2 of 9 systematic patterns, flagged 0 of 100 rounds where humans found 23 defects, and a corrected prevalence estimate (Rogan-Gladen) implied a 3–6× undercount where the gate reported a nonzero rate. Authors' diagnosis: "the 0% is a routing-and-wiring failure, not blindness". The rubric had no slot for the defects.
- **Judge accuracy falls on hard cases.** AgentJudgeBench (arXiv 2608.26623, **27 Aug 2026**, accepted to EMNLP 2026, **P**): "Judge alignment degrades monotonically with task difficulty"; on hard queries without ground truth "all six judges converge to a narrow 77-82% band regardless of scale"; structured rubrics help by up to 6.5 points; chain of thought and temperature have "negligible effects". My own search summary quoted a "κ ≈ 0.42" figure from this paper. I did not see that in the fetched abstract, so I do not cite it.
- **Small judge error shifts rankings (S).** MobileJudgeBench (arXiv 2608.11434, Aug 2026, not fetched; from a search snippet): a judge misclassifying 10–15% of trajectories can shift agent rankings.
- **Judge-version drift (the "judge silently changes" failure).** I found **no primary source** with measurements. The material is practitioner blogs: Tianpan (April 2026, **S/V**), FutureAGI glossary (**V**), LaunchDarkly blog (**V**). Their advice is consistent: pin judge model + prompt + version; when the judge changes the score delta is "unspecified, not zero"; keep a 20–50 example calibration set and re-score it on any judge change. The specific thresholds (3–5 point monthly wobble, 5% recalibration trigger, 15-point production disagreement, 75–90% agreement target) are single-author or vendor numbers. Treat as rules of thumb, not findings.
- **Measure judge quality as TPR and TNR, not agreement.** Hamel FAQ (Sep 2026) says to track "true positive rate (failures caught) and true negative rate (good outputs passed)" on a human-labelled held-out set, and notes "small samples make these measurements less certain." This agrees in substance with the June 2026 paper's argument that agreement is the wrong headline metric.

### 3.2 Eval-set contamination

- **Strongest primary:** OpenAI, "Why we no longer evaluate SWE-bench Verified" (**February 2026** per two secondary sources; OpenAI's page returned 403 so I could not confirm; see §3.4 for a date conflict). Reported via secondary sources: models could reproduce original fixes from memory, which "suggests the benchmark leaked into training data"; OpenAI stopped reporting Verified and recommends SWE-bench Pro.
- **Product caveat for this fleet.** Contamination is a public-benchmark problem. A private eval built from your own traces cannot be in a model's training set unless you published it. The practical contamination risk for a product is the reverse: your own prompt-tuning leaking into the eval (§3.3).
- **Shared-state leakage inside an eval:** Anthropic (Jan 2026, **P**): some internal evals let Claude "examine git history from earlier trials, inflating results". Each trial must start clean.
- **Models that detect the test.** IAPS memo (secondhand through a search snippet; Opus 4.6 recognised evaluations 80% of the time while saying so in 2.3% of interactions, citing UK AISI) and EvalAwareBench (NeurIPS 2026 poster). I did not find or read the underlying system-card text. Treat as **unverified**. The implication, if true, is that an eval that looks like an eval may under-measure the behaviour you care about. The sources say scaffolding leaks context (directory structures, system configs) that helps models notice.

### 3.3 Eval-set overfitting

- **Strongest first-party guidance:** Hamel FAQ (Sep 2026, **P**): keep a test split untouched until the final evaluation "to detect overfitting"; for judges split labelled examples roughly 10–20% train / 40–45% dev / 40–45% test with at least 50 passing and 50 failing examples. For CI: "CI evals protect against known regressions"; CI sets are small and purpose-built.
- **Saturation as the product-side form.** Anthropic (Jan 2026): "Eval saturation occurs when an agent passes all of the solvable tasks, leaving no room for improvement." A saturated eval still catches regressions but gives no improvement signal. Capability evals should "start at a low pass rate", then graduate to regression suites near 100%.
- **Gap:** I found no 2026 primary source that measures prompt-overfitting to a product eval set specifically (as opposed to public benchmarks). This is an open item (§6).

### 3.4 Unrepresentative sampling and broken tasks

- **Anthropic (Jan 2026, P):** a 0% pass rate across many trials "is most often a signal of a broken task, not an incapable agent"; two domain experts should reach the same pass/fail verdict on a task; test both when a behaviour should *and should not* occur (the Claude.ai web-search evals balance undertriggering and overtriggering).
- **Strongest benchmark-quality primary:** OpenAI's audit of SWE-bench Verified. Per a secondary report (aiweekly, whose page carries its own date of 8 July 2026 and flags a conflict itself): 138 of the hardest problems audited by ≥6 engineers; 59.4% had material issues in test design or problem description; 35.5% enforced unrelated implementation details; 18.8% tested functionality the task did not ask for. **Date conflict:** search results and Gigazine (a summary dated 29 Apr 2026) place OpenAI's post in February 2026; aiweekly shows 8 July 2026, which is more likely that outlet's own publication date. **I could not resolve this from openai.com.** Do not quote the OpenAI date in the skill without checking.
- **Replacement is also contested (S).** aiweekly reports a May 2026 Datacurve audit finding SWE-bench Pro mis-graded about a third of trials (wrong patches accepted ~8.5%, correct ones rejected ~24%), and says Scale AI had not confirmed this. The same page's metadata headline suggests OpenAI later backed away from Pro, which contradicts its body text. **Unresolved.** Lesson for the fleet is unchanged: any single published benchmark can be broken. Audit your own tasks.
- **Infrastructure as hidden variable.** Anthropic, "Quantifying infrastructure noise in agentic coding evals" (**5 Feb 2026**, **P**; fetched): Terminal-Bench 2.0 gap between most- and least-resourced setups was 6 percentage points (p < 0.01); infra error rate 5.8% at strict limits vs 2.1% at 3× headroom vs 0.5% uncapped; "leaderboard differences below 3 percentage points deserve skepticism". Directly relevant: *an eval that runs on different hardware or limits across runs is not measuring the same thing*.
- **Production sampling.** Hamel FAQ: keep random traces in every batch (not only flagged ones); "track confidence intervals for production metrics".
- **Synthetic data bias** (see §2): cannot give frequencies.

### 3.5 Evals that pass while the product fails

- **Strongest primary:** the same June 2026 paper. The judge flagged **0 of 100** rounds while humans confirmed 23 defects. The gate was green and the product was defective.
- **Corroborating, weaker:** Anthropic (Jan 2026): "Failures should seem fair: it's clear what the agent got wrong and why," and they "do not take eval scores at face value" until someone reads transcripts. Hamel FAQ: generic metrics "waste time and create false confidence when you use them as quality measures"; BERTScore, ROUGE and cosine similarity "are not useful for evaluating LLM outputs in most AI applications".
- **Practitioner (V/S):** FutureAGI's "agent passes evals fails production" (2026) and Tianpan's "eval automation trap" (May 2026) describe the same pattern. No data. Treated as colour only.

---

## 4. Question 4: evals inside a loop

### 4.1 Gate vs explainer: the difference, as the sources state it

| | **Gate (blocks shipping)** | **Explainer (tells you why)** |
|---|---|---|
| Typical form | Small, purpose-built regression set; deterministic assertions; "pass all" rate | Trace review, trace grading, human annotation, taxonomy, narrative report |
| Source | Hamel FAQ: "favor assertions or other deterministic checks over LLM-as-judge evaluators" for CI; report a "pass all" rate and drill down by check, not a weighted composite. Anthropic: regression evals "should sit near 100%", run "on each agent change and model upgrade". | OpenAI: trace grading is the *first* stage because "it surfaces workflow-level problems quickly"; datasets come once you know what good looks like. Hamel: narrative reports "explain what changed, likely causes, and next steps". |
| Failure of this type | Saturation. Everything passes, and a green gate says nothing about new failure types. | No repeatability. You learn a lot once, and cannot tell whether the fix held. |
| What makes it earn its place | It encodes a failure you have actually seen (Hamel: reproduce the error first; Anthropic: tasks drawn from real failures). | It leads to a named change. |

Both Anthropic and Hamel describe the two as consecutive stages of one loop, not competitors: discover → write a check for each observed failure → regress on it → sample production → discover again. Anthropic's version: pre-launch/CI evals, post-launch monitoring, A/B once traffic allows, weekly transcript sampling, human studies to recalibrate. (Anthropic also describes "eval-driven development": write evals for planned capabilities first; when a new model arrives "running the suite quickly reveals which bets paid off.")

**Judge cost matters to gate design.** Hamel: "The more expensive the eval, the greater benefit it needs to provide to justify running it frequently"; LLM judges should be phased out more aggressively than cheap checks.

### 4.2 What practitioners report about evals that never changed a decision

First-party evidence here is thin. This is the weakest question of the four.

- **Hamel FAQ (Sep 2026):** "I have rarely found dashboards to be helpful." "If everything keeps passing, this is a sign that the eval is less useful and should be run less often or retired." Recommends narrative reports and product metrics (churn, active users, suggestion acceptance) as the long-term measure.
- **Anthropic (Jan 2026):** saturated evals "give no improvement signal". Its prescription is to keep a *human* rhythm alongside automation: "sample transcripts weekly", triage feedback constantly.
- **LangChain survey (Dec 2025 fieldwork):** tracing 62–89% but offline evals only 52.4%. This shows many teams watch without testing, but says nothing about whether the evals they have changed a decision. A secondary piece repeated this as "89% trace, 52% test" (**S**; it conflates observability, which is 89%, with tracing, which is 62%).
- **Practitioner posts (V/S, 2026):** "eval theater" with dashboards and no signal; a weekly eval-and-trace review compared to an SRE incident review; a "reintroduce a fixed bug and see if the score moves" test of whether a suite measures anything. I read these only as search snippets. They are consistent with first-party advice but are not evidence.
- **I found no first-party study that counts how often evals change decisions.** This is a genuine gap (§6).

**A usable heuristic assembled from the sources (mine, not any one source's):** an eval is earning its keep if you can name (a) a failure observed in real traces that it encodes, (b) the decision it informs (ship / don't ship, change prompt, change tool, change model), and (c) the date by which it will be retired if it only ever passes. Hamel's retire-if-always-passing advice and Anthropic's graduation rule support (c); the others are inferences.

---

## 5. Judgement on the starting article

**Article.** "Advanced evals: How to find (and fix) hidden AI failures in your product", Hamel Husain and Shreya Shankar, Lenny's Newsletter, **22 Sep 2026** (paid; free portion only). Fetched summary: paywalled "at the start of 'Step 3'".

**Finding vs fixing.** Everything I could read is about *finding*: traces, annotation, simulated users, ranking failure modes. The one *fix*-adjacent statement is the claim that evals let teams ship faster by checking changes don't break existing behaviour. That is regression gating, not repair. The paywalled Step 3 (clustering and ranking) is still finding. Nothing I could see addresses how to fix a failure once found (prompt change, tool change, model change, guardrail). **I cannot say whether the paywalled remainder does.** For the skill, treat the article as a *discovery* source only.

| # | Article claim (as visible) | Verdict | Evidence |
|---|---|---|---|
| 1 | If only one part of the eval process fits your time, do error discovery. Teams jump to metrics too early. | **Corroborated** | Hamel FAQ (Sep 2026): "most important activity in evals"; Anthropic (Jan 2026): read transcripts, start from real failures; OpenAI trace-first docs; Catching One in Five (Jun 2026) shows metric-first judges miss the defects. Note: several of these sources are from the Hamel and Shankar team, so their agreement counts as one team. Anthropic and OpenAI are independent confirmations of the direction (read traces first), not of the specific method. |
| 2 | Early metrics rest on assumptions and measure the wrong things. | **Corroborated** | Same as above; Shankar et al. 2024 (background) on criteria that cannot be fixed before looking; Hamel FAQ on generic metrics. |
| 3 | Reviewing examples changes your definition of "good" (criteria drift). | **Corroborated, but by their own work** | Shankar et al., UIST 2024 (background). No independent 2026 replication found. |
| 4 | Each trace should capture input, system prompt, tool calls, intermediate steps, final output. | **Corroborated** | OpenAI: a trace is "model calls, tool calls, guardrails, and handoffs for one run". Anthropic: transcript = "the full record of a trial". |
| 5 | Simulate users via dimensions (task, user type, request clarity) and one query per combination for launch-stage products. | **Corroborated with a limit** | Hamel FAQ agrees; also states synthetic data "cannot tell you how common a failure is". Google ADK now ships user simulation, so the idea has first-party tooling. The article's free part does not state the frequency limit. |
| 6 | Annotate at least 10 yourself before trusting the agent; aim for ~100 traces. | **One team's experience; inconsistent within the team** | Hamel FAQ (Sep 2026) says personally annotate the first **30** and review ≥100. No source supplies evidence for either threshold. |
| 7 | Focus on user-facing failures, not root causes; note the *first upstream error*. | **Corroborated** | Hamel FAQ: process failures first; Tianpan (S) on first upstream failure; Anthropic: error propagation across turns. The "first upstream error" rule is a practitioner heuristic with no study behind it. |
| 8 | A coding agent can help review: it catches obvious in-trace errors, finds some humans miss, but misses product-judgement failures and flags good responses as failures. | **Complicated: plausible, unmeasured** | The article's own caveats match the judge literature (June 2026: judges miss cross-turn state defects; Aug 2026: judge alignment drops on hard cases to a 77–82% band). But the specific "agent-assisted review" workflow has **no published measurement**. Hamel's note and skills post cite none. |
| 9 | Cluster annotations into failure modes, count, rank by impact. | **Not verifiable (paywalled)**; the idea is corroborated | Hamel FAQ and clustering tools above. Anthropic: "prioritize by user impact". The article's method for ranking is unseen. |
| 10 | Evals let AI products ship faster by checking changes don't break existing behaviour; matters more as AI changes outpace human review. | **Corroborated** | Anthropic: regression evals in CI on each change and model upgrade; "running the suite quickly reveals which bets paid off". |
| 11 | Shopify, Cursor, Ramp, Harvey saw measurable gains from evals. | **Not verifiable** | Cited in the visible summary, but I could not see the claims or sources. Anthropic names different companies (Descript, Bolt, Claude Code). **Do not reuse these as evidence in the skill** without reading the original. |
| 12 | (implicit) Human-built taxonomy beats an LLM-invented one. | **Corroborated by one team only** | Hamel note (Jun 2026): "Build an error taxonomy from human review instead of asking an LLM to invent one." Atla's contrary-looking snippet (clustering critiques with embeddings) is about *grouping*, not inventing categories, and it was unreadable (404). No independent test. |

**What the article leaves out that the field now treats as central:**
- Whether an automated judge catches the failure at all (recall). The June 2026 paper makes this the key measurement.
- Frequency estimation: discovery tells you *what* breaks, not *how often*; synthetic traffic cannot tell you.
- Reliability across repeated trials (pass^k).
- Infrastructure and environment as a source of "failures" that are not the agent's fault (Anthropic, Feb 2026).
- Security failures. Adversarial probing needs a different method than annotation of ordinary traces.

**Net judgement.** On *what to do first* the article is aligned with the field. On *how well agent-assisted discovery works* it is a claim from the authors, who also sell the course and the skills, and I found nothing independent. The authors cite "50+ companies" (course page). That is a statement of experience, not a result.

---

## 6. What is genuinely unsettled

Each is phrased as a question this fleet would have to answer for itself.

1. **How do we know the agent-assisted review is finding the failures a human alone would find?** No source measures recall of agent-assisted vs human-only review. Do we run a small head-to-head on a trace set we have fully human-reviewed?
2. **How many traces is enough?** Sources give 100, with the stopping rule "until new traces stop revealing failure modes". Is the stopping rule applied by the person or the agent, and what do we do for a low-traffic product that cannot produce 100 traces?
3. **Who sets the standard when judge and human disagree?** The field says calibrate to humans, but human annotators also drift (criteria drift). How often must we re-label, and what inter-annotator agreement is acceptable?
4. **What does "recall of real defects" cost to measure?** The June 2026 paper needed exhaustive human review of whole batches. What is the cheapest recurring check we can afford that still catches a judge that has stopped seeing a defect class?
5. **When a judge model is retired or upgraded, what is our policy?** Practitioner posts say pin and re-score a calibration set. None is first-party and the numeric triggers are single-author. What threshold change in agreement makes us stop trusting its scores?
6. **How do we get failure frequency without production traffic?** Synthetic data cannot supply it. How do we avoid gating on a number we know is unrepresentative?
7. **How do we detect an eval-aware agent?** If the evaluation-awareness reports are right (unverified in this scout), scaffolded eval environments may be recognised. Does our product agent see something in our harness that production does not?
8. **How do we measure that an eval changed a decision?** There is no first-party study. Do we log, for each eval, the decision it informed, and retire it if the log stays empty?
9. **Where do fixes come from?** The starting article's free portion stops at discovery. Is a fix a prompt edit, a tool change, a model change, or a guardrail, and what evidence tells us which?
10. **Security vs quality.** Adaptive attacks defeat defences that pass static tests (June 2026). Is red-teaming inside `/draft-agent-eval` or a separate skill?
11. **Is a public benchmark ever useful to us?** Even the "replacement" benchmark is contested (SWE-bench Pro, May–July 2026 reports, unresolved). Do we build only private task sets, and how do we audit their task quality (Anthropic's two-experts test)?

---

## 7. What this means for a `/draft-agent-eval` skill

The decisions the skill should force, **in order**, with the evidence for each. Each is a question the user must answer before the skill proceeds; the order follows the evidence about what goes wrong when it is reversed.

1. **"Do you have traces of the agent doing real (or realistic) work, with inputs, tool calls, intermediate steps, and outputs?"**
   If no: the first deliverable is instrumentation, not an eval. Evidence: article, OpenAI trace-grading docs, Anthropic transcript definition. Launch-stage with no traffic: fall to step 2 and be explicit that frequencies will be unknown (Hamel FAQ).
2. **"Where will the failures come from?"** Real traces (preferred) or simulated users from dimension tuples. Force the user to name the dimensions. State the limit: synthetic data gives coverage, not frequency (Hamel FAQ, Sep 2026).
3. **"Will a human read the traces before any metric exists?"** Gate: no judge, rubric or dashboard is drafted until a person has annotated a first batch (sources disagree on 10 vs 30; the skill should set a minimum and say it is a heuristic). Evidence: Hamel FAQ; Anthropic "Read the transcripts!"; June 2026 paper (metric-first judge missed the defects).
4. **"For each failure found, can a rule detect it?"** Branch per failure mode:
   - Rule exists → deterministic check; goes in the CI gate (Hamel FAQ: prefer assertions in CI).
   - Needs judgement → one narrow LLM judge per failure, with human-labelled examples (Hamel: "directing their LLM judge to catch too many different kinds of errors at once" is the most common mistake).
   - Cannot be stated yet → keep as a review item; do not score it.
   - Also ask: should it be an inline guardrail instead (fast, deterministic) rather than an after-the-fact evaluator? (Hamel FAQ.)
5. **"Is a unit test or a score the wrong tool here?"** Surface the cases where the sources say so: single-path trajectory scoring on tasks with several correct paths; any score on a failure nobody has yet defined; a similarity metric offered as "quality"; a slow judge used as a guardrail; an exact-string grader on free-form output.
6. **"How will you check the checker?"** For every judge: report TPR and TNR on a held-out human-labelled set (Hamel); also plan a recall check against human review of a fresh batch (June 2026). Pin the judge version and write down what triggers a re-check (practitioner advice only; flag as such).
7. **"How many trials, and under what environment?"** Repeated trials with pass^k when consistency matters (Anthropic). Fix and record the environment limits (Anthropic, Feb 2026: 6-point swing from resources alone). Clean state per trial.
8. **"Is each task fair?"** Two experts agree on pass/fail; 0% pass is a task-bug signal; balance should-trigger and should-not-trigger cases (Anthropic).
9. **"What decision does each eval inform, and when does it retire?"** Force a one-line decision per eval; set the retire/run-less-often rule for always-passing evals (Hamel FAQ). Capability set vs regression set (Anthropic).
10. **"What is the loop's cadence?"** Weekly: sample 10–20 fresh traces, include random ones, read them, update failure modes (Hamel FAQ; Anthropic weekly transcript sampling). Report as a narrative with the pass-all rate and by-check drilldown, not a composite dashboard (Hamel FAQ).
11. **"What happens when the model changes?"** Re-run regression suite, re-score judge calibration set (practitioner advice; mark as unconfirmed by first-party).
12. **"Does this product face an adversary?"** If yes, route to red-teaming separately and require adaptive attacks (June 2026), not only static ones.

**Design implications to carry into the skill (mine, flagged as inference):**
- Keep **finding** and **fixing** as separate phases. The starting article and the evidence cover finding; fixing needs its own source set.
- Require the skill to record **provenance** per claim in its output (observed in traces, from synthetic data, or assumed), because the evidence shows synthetic data cannot give frequency and judges cannot be assumed to see a defect class.
- Do not ship a default judge. Every judge the skill drafts should come with its calibration plan, because the strongest 2026 evidence is about silent judge failure.
- Treat the Hamel/Shankar `evals-skills` as an existing implementation to read, not copy blindly. It is an exemplar (Appendix B), and the authors state "These skills are only a starting point."

---

## Appendix A: queries and results (live retrieval record)

Tools: WebSearch (standard) and WebFetch. Date of retrieval: 2026-11-12.

| # | Query / URL | Result used |
|---|---|---|
| 1 | WebFetch `https://www.lennysnewsletter.com/p/advanced-evals-how-to-find-and-fix` | Date "Sep 22, 2026 (paid)"; paywalled "at the start of 'Step 3'"; claims in §5 |
| 2 | WebSearch "Hamel Husain Shreya Shankar advanced evals find fix hidden AI failures error discovery" | Confirmed article, hamel.dev note, eval-audit skill pages |
| 3 | WebSearch `Anthropic "Demystifying evals for AI agents"` | URL `anthropic.com/engineering/demystifying-evals-for-ai-agents` |
| 4 | WebFetch `https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents` | Date "January 9, 2026"; grader types; pass^k; contamination; loop |
| 5 | WebFetch `https://hamel.dev/notes/llm/ai-product-engineering/evals-error-analysis.html` | Published "June 24, 2026"; no quantitative results |
| 6 | WebFetch `https://hamel.dev/blog/posts/evals-faq/` (offsets 0 and 100000) | "Last updated September 21, 2026 (originally published September 18, 2026)"; sample sizes; CI; dashboards; red teaming |
| 7 | WebSearch "agent evaluation 2026 error analysis traces LLM-as-judge drift practitioners" | Leads to arXiv 2510.08847 (GPA), TRAIL, vendor blogs (all treated as S/V; not fetched) |
| 8 | WebFetch `https://developers.openai.com/api/docs/guides/agent-evals` | Trace-first progression; no grader-reliability discussion; no date |
| 9 | WebFetch `https://adk.dev/evaluate/criteria/` | 17 criteria; exact-match default; informational efficiency criteria |
| 10 | WebSearch "LLM-as-judge agent evaluation reliability study 2026 arXiv judge agreement human agent trajectories" | Leads to 2606.10315, 2608.26623, 2608.11434 |
| 11 | WebFetch `https://arxiv.org/abs/2606.10315` | Submitted 9 Jun 2026; 2 of 9 patterns; 0 of 100; 113 of 114 |
| 12 | WebFetch `https://arxiv.org/abs/2608.26623` | Submitted 27 Aug 2026; 77–82% band; rubrics +6.5 pp |
| 13 | WebFetch `https://www.anthropic.com/engineering/infrastructure-noise` | Date "February 5, 2026"; 6 pp (p<0.01); 5.8%/2.1%/0.5%; 3-point skepticism |
| 14 | WebSearch "benchmark contamination SWE-bench Verified 2026 OpenAI stops reporting flawed tests" | Title and Feb 23, 2026 date per snippet (not confirmed on page) |
| 15 | WebFetch `https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/` and `https://openai.com/en/index/...` | **Both returned "HTTP 403 Forbidden"** |
| 16 | WebFetch `https://aiweekly.co/node/5874` | Shows "article dated July 8, 2026"; 59.4%; Datacurve audit; flags its own metadata conflict |
| 17 | WebFetch `https://gigazine.net/gsc_news/en/20260429-swe-bench-verified` | "OpenAI reported these findings in February 2026" |
| 18 | WebFetch `https://langchain.com/state-of-agent-engineering` | 52.4% / 37.3% / 59.8% / 53.3% / 89% / 62%; Nov 18–Dec 2, 2025; n=1,340 |
| 19 | WebFetch `https://huggingface.co/papers/2606.15057` | AutoDojo numbers; no submission date shown; ID implies June 2026 |
| 20 | WebFetch `https://arxiv.org/abs/2607.06873` | Submitted 8 Jul 2026; 23–38 vs 12 boundaries; **seeded-fault catch rate not reported** (corrects search summary) |
| 21 | WebFetch `https://www.anthropic.com/research/bloom` | Date "December 19, 2025"; Spearman 0.86 / 0.75 on 40 transcripts |
| 22 | WebFetch `https://hamel.dev/blog/posts/evals-skills/` | "Published March 2, 2026. Modified August 31, 2026."; skill list; no benchmarks |
| 23 | WebSearch "Shreya Shankar criteria drift OR who validates the validators …" | arXiv 2404.12272 (UIST 2024); criteria drift definition (background) |
| 24 | WebSearch "LLM judge model deprecation version change score drift …" | Tianpan, FutureAGI, LaunchDarkly (S/V) |
| 25 | WebFetch `https://atla-ai.com/post/automating-error-analysis` | **HTTP 404 Not Found**: its claims rest on a search snippet only |
| 26 | WebSearch queries on clustering, red teaming, seeded faults, eval awareness, "evals that never changed a decision" | Snippets only; marked S/V throughout |

## Appendix B: existence checks

**Positive checks (run with `gh api repos/<owner>/<repo> --jq '{full_name,pushed_at,archived,description}'`):**
- `ai-evals-course/evals-skills` → `{"archived":false,"description":"Skills that guide AI coding agents to help you build product-specific AI evals.","full_name":"ai-evals-course/evals-skills","pushed_at":"2026-10-24T19:45:19Z"}`. This matches the install command the skills post gives: `npx skills add https://github.com/ai-evals-course/evals-skills`.
- `hamelsmu/evals-skills` → `"archived":true`, `"pushed_at":"2026-08-16T05:28:25Z"`. A search result pointed to this older repo. It is **archived**; use the `ai-evals-course` one.
- `safety-research/bloom` → `"archived":false`, `"pushed_at":"2026-05-07T20:47:35Z"`, description "bloom - evaluate any behavior immediately".
- `safety-research/petri` → the API returned `"full_name":"meridianlabs-ai/inspect_petri"`, `"pushed_at":"2026-11-11T22:45:46Z"`, `"archived":false`. That means the old name now redirects to the Meridian Labs repository. I did not check who maintains it or whether Anthropic still does.

**Negative / absence statements in this report and the check behind each:**
- "The Hamel note has no quantitative results." Checked on the fetched page: "The page contains no quantitative results, benchmarks, or case data about the method's effectiveness or time savings." The fetch tool read the full page (short). It is a summary, not raw HTML, so I treat this as strong but not exhaustive.
- "The skills post has no benchmark." Checked: "The page gives no benchmarks, results, or independent validation."
- "OpenAI's agent-evals guide does not discuss grader reliability." Checked on the fetched page: "The page doesn't discuss grader reliability, calibration, or known limitations."
- "No skill named `error-analysis` on the skills post." Checked: "The page has no skill named 'error-analysis.' The closest match is error-discovery." (Third-party listings say it was formerly called error-analysis. I did not check git history to confirm the rename, so I do not assert it.)
- "No first-party measurement of judge-version drift / of how often evals change decisions / of agent-assisted review recall." These are **absence-of-evidence claims from search, not from an authoritative existence surface**. There is no registry for "studies". I ran the searches in Appendix A (rows 7, 10, 24, 26) and found none, but my search coverage was single-pass per topic. Do not read these as proof that no such study exists. Firstmate or the captain should treat them as "not found in one pass".
- "OpenAI's SWE-bench post date." **Unresolved**: openai.com returned 403 on both URLs tried (rows 15); sources conflict (February vs July 2026).
- The Lenny article's Shopify/Cursor/Ramp/Harvey examples and Step 3 content: **not checked** (paywall). The authoritative surface (the full article) was not accessible.

## Appendix C: figures I saw in search summaries and chose not to cite

- "Judge caught 22 of 27 seeded process faults (0.81)": not present on the fetched abstract of arXiv 2607.06873 (row 20).
- "κ ≈ 0.42 between judges and ground truth" in AgentJudgeBench: not in the fetched abstract (row 12).
- "Agent GPA judges reach 82% alignment, α = 0.77" (arXiv 2510.08847, October 2025, background): search snippet only, not fetched.
- "TRAIL baseline judge caught 55% of errors": search snippet only; arXiv 2505.08638 is May 2025 (background).
- Anthropic infrastructure-noise numbers: I **did** verify these on the Anthropic page (row 13). Earlier search snippets from mirror sites matched.
- IAPS "Opus 4.6 recognised evaluations 80% of the time, verbalised awareness 2.3%": snippet only. Not verified against the system card or the UK AISI report.
- Gray Swan / UK AISI "272,000+ attacks, 0.5%–8.5%": snippet only.
- Tianpan/FutureAGI/LaunchDarkly numeric thresholds: single-author or vendor, no method given.
