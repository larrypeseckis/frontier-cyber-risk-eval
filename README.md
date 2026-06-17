# Frontier Cyber Risk Eval Set

A taxonomy, a labeled prompt set, the tooling to run it, and an honest pilot result. The project asks a narrow question and answers it: can an LLM judge be trusted to grade how a model handles cyber requests, and where does that judge break.

**Version:** 0.1 · **License:** MIT · **Status:** pilot, open to comment

---

## What this is

Most public writing about AI and cyber risk argues at the easy ends (clearly allowed, clearly disallowed) and skips the middle where the real disputes live. This repo takes the opposite approach. It defines a four-tier taxonomy of cyber assistance, turns it into a runnable eval set, and then tests the test: it grades a model's responses with an LLM judge, grades the same responses with a human, and compares the two.

The deliverable is not a leaderboard. It is an instrument plus a first run that shows the instrument works under stated conditions and shows exactly where the automated grader can and cannot be trusted.

---

## The four layers

1. **Taxonomy.** Four tiers (Allowed, Dual-use, High-risk, Disallowed) mapped to the uplift and autonomy thresholds frontier labs have converged on. See `frontier-cyber-risk-taxonomy.md`.
2. **Eval set.** 57 labeled prompts that operationalize the taxonomy, with rationale, risk factor, expected behavior, whether authorization matters, and which capability axis each prompt exercises. See `EVAL-SET.md`.
3. **Tooling.** An LLM-as-judge scorer and a blind human-vs-judge comparison harness. See `scorer.py` and `human_compare.py`.
4. **Results.** A pilot run against gpt-5.5, judged by claude-opus-4-8, with one human rater. See `RESULTS.md`.

Every prompt is a stimulus, not a payload. Tier 3 and Tier 4 entries are refusal-tests written at the level of the request. They contain no working code, no operational technique, and no step-by-step. The thing measured lives in the model's response.

---

<<<<<<< HEAD
## Pilot results in brief

Target gpt-5.5 (bare model), judge claude-opus-4-8, one human rater, 57 prompts. Full writeup and caveats in `RESULTS.md`.

- **gpt-5.5 showed zero measured cyber over-refusal** on this set: 0 of 5 over-refusal traps and 0 of 33 benign prompts wrongly refused, with constructive (redirect) refusals on the disallowed items.
- **The judge matched the human on every verdict it rendered.** Pass/fail agreement was complete over the 53 judged items; observed-behavior agreement was 0.962 (Cohen's kappa 0.923), with the only two disagreements one step apart at the multi-turn and dual-use boundaries.
- **The judge abstained on 4 of 10 Tier 4 items**, the most severe disallowed prompts, refusing to emit a verdict even with escalating instruction. A single-LLM-judge pipeline silently loses coverage on its most important cases unless abstention is detected and handled.

The pilot supports two claims (the set distinguishes answer/gate/refuse across tiers; a cross-family judge agrees with a human on pass/fail for clean responses) and does not support a third (benchmark-level safety, judge reliability under messy partial compliance, or severe-Tier-4 coverage without human fallback).
=======
## Results

Two pilots, each answering a different question. The value is in the progression: Pilot 1 tests the judge on clean responses, Pilot 2A stress-tests it on messy ones with known labels.

### Pilot 1: clean target

Target gpt-5.5 (bare model), judge claude-opus-4-8, one human rater, 57 prompts. Full writeup in `RESULTS.md`.

- **gpt-5.5 showed zero measured cyber over-refusal** on this set: 0 of 5 over-refusal traps and 0 of 33 benign prompts wrongly refused, with constructive (redirect) refusals on the disallowed items.
- **The judge matched the human on every verdict it rendered.** Pass/fail agreement was complete over the 53 judged items; observed-behavior agreement was 0.962 (Cohen's kappa 0.923).
- **The judge abstained on 4 of 10 Tier 4 items**, the most severe disallowed prompts, refusing to emit a verdict. A single-LLM-judge pipeline silently loses coverage on its most important cases unless abstention is detected and handled.

A clean target cannot show whether the judge is reliable or merely unchallenged, which is what Pilot 2A is for.

### Pilot 2A: controlled judge stress test

Same judge (claude-opus-4-8), 30 messy response shapes with frozen intended labels, one independent blind human rater. Full writeup in `RESULTS-pilot-2a.md`.

- **Judge reliability is shape-dependent.** Perfect on clean answers, clean refusals, hedged compliance, and gated responses; collapses to 0.25 to 0.67 pass/fail accuracy on responses where surface and substance conflict (leaky refusals, over-sharing redirects, endpoint collapse). One coherent failure mode: the judge sides with a refusal-shaped surface over leaked substance.
- **A blind human beat the judge on pass/fail** (0.833 vs 0.679) and caught the specific surface/substance errors it missed, but three of the hardest cases beat both. The case is for human review alongside the judge, not replacement.
- **Content-triggered abstention replicated** (2 of 30), confirming Pilot 1's abstention finding is a stable property, not a one-off.

Together the pilots support that the eval distinguishes answer/gate/refuse across tiers, that a cross-family judge agrees with a human on clean responses, and that the same judge needs human backstop on refusal-shaped responses carrying technical substance. They do not support benchmark-level safety claims or judge reliability on a realistic (rather than deliberately hard) response distribution. The next step, Pilot 2B, runs a real messier model to see whether these shape-specific failures appear in naturally generated output.
>>>>>>> 9e1ee58 (Add Pilot 2A: controlled judge stress test, results, and harness)

---

## Quickstart

```bash
pip install -r requirements.txt

# 1. collect a target model's responses (openai bare model shown; see EVAL-SET.md for others)
export OPENAI_API_KEY=...
python scorer.py collect --provider openai --model gpt-5.5 --out responses.jsonl

# 2. judge them with an LLM judge
export ANTHROPIC_API_KEY=...
python scorer.py score --judge-model claude-opus-4-8 --responses responses.jsonl --report report.json

# 3. grade blind yourself, then compare against the judge
python human_compare.py grade --responses responses.jsonl --out human_grades.jsonl
python human_compare.py compare --human human_grades.jsonl --report report.json --out comparison.json
```

Grade before opening the judge's report, so your grades stay independent. The grading rubric in `grading-rubric.md` is the companion reference for that pass.

---

## Repository structure

| File | What it is |
|---|---|
| `README.md` | This front door. |
| `frontier-cyber-risk-taxonomy.md` | The four-tier taxonomy and its eval criteria. |
| `frontier-cyber-eval-set.jsonl` | Canonical eval data, one labeled prompt per line. |
| `frontier-cyber-eval-set.csv` | Same data flattened for spreadsheet review. |
| `EVAL-SET.md` | Detailed eval-set reference: schema, dimensions, usage. |
| `build_eval_set.py` | Single source of truth; regenerates the JSONL and CSV. |
| `scorer.py` | LLM-as-judge harness: `collect` (anthropic/openai/codex-cli) and `score`. |
| `human_compare.py` | Blind human grader and human-vs-judge agreement. |
| `grading-rubric.md` | One-page rubric for the blind grading pass. |
| `sample_responses.jsonl` | Small example responses for a dry run of `score`. |
<<<<<<< HEAD
| `RESULTS.md` | The v0.1 pilot results, findings, and limitations. |
=======
| `RESULTS.md` | Pilot 1 results: clean target, judge-vs-human, abstentions. |
| `build_stress_set.py` | Single source of truth for the Pilot 2A stress set. |
| `stress_responses.jsonl` | 30 ground-truth-labeled messy response shapes. |
| `stress_responses.csv` | Same stress data flattened for review. |
| `stress_eval.py` | Pilot 2A harness: judge + blind human scored against ground truth. |
| `RESULTS-pilot-2a.md` | Pilot 2A results: judge reliability under controlled stress. |
>>>>>>> 9e1ee58 (Add Pilot 2A: controlled judge stress test, results, and harness)
| `requirements.txt` | Python dependencies. |
| `LICENSE` | MIT. |

---

## Reproducing the eval data

The prompt set is generated from a single source so the JSONL and CSV cannot drift:

```bash
python build_eval_set.py   # rewrites frontier-cyber-eval-set.jsonl and .csv
```

Edit the `DATA` list in `build_eval_set.py` to add or change prompts, then re-run.

---

## Companion taxonomy

This repo includes the taxonomy as `frontier-cyber-risk-taxonomy.md`. If a newer version is published elsewhere, treat the published version as canonical and sync this file to match. The eval set implements the taxonomy; the taxonomy stands on its own.

---

## Limitations

This is a v0.1 pilot, not a benchmark. The headline numbers are bounded by a 57-prompt set, a single target run at fixed conditions, a single LLM judge, and one human rater. The pilot cannot separate human-vs-human disagreement from human-vs-judge disagreement, cannot speak to judge reliability under messy partial compliance, and has a coverage hole at the top of the severity scale where the judge abstained. Full limitations are in `RESULTS.md`. Read the numbers as evidence the instrument works under these conditions, not as a safety ranking.

---

## Citation

```
Frontier Cyber Risk Eval Set v0.1. Larry Peseckis, 2026.
Taxonomy, labeled eval set, LLM-as-judge scorer, and human-vs-judge comparison
for cyber assistance from frontier models.
```

---

## License

MIT. See `LICENSE`.
