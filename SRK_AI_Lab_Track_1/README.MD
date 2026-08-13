# StanceEval 2026 — Track 1
## LLM-Based Prediction System Documentation

**Team:** SRK AI Lab  
**Track:** Track 1 — Seen Targets

---

## 1. Purpose

This document describes the prediction workflow and supporting evidence for
the Track 1 submission to StanceEval 2026.

The submitted system used a prompt-based ensemble of three Large Language
Models (LLMs):

- GPT
- Gemini
- Qwen

The system used **few-shot prompting**. The models were accessed directly
through their respective chat interfaces rather than through an API-based
automated inference pipeline.

The final prediction process combined the outputs of the three LLMs with
additional GPT-based auditing and evidence-based manual adjudication.

---

## 2. Task

Track 1 is the seen-target stance detection setting.

For each instance, the system receives:

- a Target;
- an Arabic Tweet; and
- the task's stance-label space.

The required stance labels are:

- `Favor`
- `Against`
- `None`

The decision was made with respect to the **specific Target**, rather than
treating the Tweet as a general sentiment statement.

---

## 3. Few-Shot LLM Prediction

For each LLM, the prediction process used a few-shot prompt containing
representative examples together with the Target and Tweet to be classified.

The three LLMs were used as independent prediction sources:

1. GPT
2. Gemini
3. Qwen

The purpose of using multiple LLMs was to obtain independent interpretations
of the Arabic Target–Tweet pair and to identify cases where the models
disagreed or where the stance was difficult to determine.

The supplied prompt and few-shot materials are preserved in the accompanying
evidence package where available.

---

## 4. Cross-Model Comparison

After obtaining the three model predictions, their outputs were compared.

Two broad situations were considered:

### 4.1 Agreement

When the models agreed, the prediction was considered together with the
Target–Tweet evidence.

Model agreement was treated as supporting evidence, not as an automatic
guarantee of correctness.

### 4.2 Disagreement or Difficult Cases

When GPT, Gemini, and Qwen disagreed, or when an instance appeared
linguistically difficult, the case received additional review.

Particular attention was given to the relationship between the Tweet and its
Target, including contextual meaning, negation, sarcasm, implicit stance,
hashtags, and wording that may describe a topic without expressing an
attitude toward it.

---

## 5. GPT-Based Audit

A dedicated GPT Audit workflow was used to review difficult and
disagreement cases.

The audit was provided with the relevant Target, Tweet, and model
predictions and was asked to reason about the stance expressed toward the
Target.

The audit was used as an **adjudication and quality-control aid**.

It was not treated as an infallible source of ground truth, and its output
was not applied as an unconditional automatic label-replacement rule.

The audit could support a decision, identify a potentially problematic
prediction, or indicate that the original prediction should be retained.

The audit record and its prompt are included in the supporting materials.

---

## 6. Manual / Evidence-Based Adjudication

The final prediction was determined using the combined model evidence,
GPT-based audit analysis, and direct examination of the Target–Tweet pair.

When an audit recommendation was available, it was considered together with
the actual linguistic evidence in the Tweet.

The final decision was therefore **not based solely on the GPT Audit**.

In particular, the process did not use a rule such as:

> "Always follow GPT Audit."

Instead, the final label was selected according to the strongest
Target–Tweet-supported interpretation after considering the available model
predictions and audit analysis.

This human-guided adjudication is an intentional part of the prediction
workflow.

---

## 7. Final Prediction Pipeline

The Track 1 workflow can be summarized as:

```text
Target + Arabic Tweet
        |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
   GPT Few-Shot       Gemini Few-Shot      Qwen Few-Shot
        |                   |                   |
        +-------------------+-------------------+
                            |
                            v
                   Cross-Model Comparison
                            |
                 +----------+----------+
                 |                     |
             Agreement          Disagreement /
                 |              Difficult Case
                 |                     |
                 |                     v
                 |                GPT Audit
                 |                     |
                 +----------+----------+
                            |
                            v
                 Target–Tweet Examination
                            |
                            v
                  Manual Adjudication
                            |
                            v
                    Final Prediction
```

---

## 8. LLM Access

The LLM predictions were generated through direct interaction with the GPT,
Gemini, and Qwen chat interfaces.

No API-based automated querying pipeline was used for these LLM interactions.

Therefore, there is no original API-querying code corresponding to the
interactive LLM calls.

This is intentional and is documented here to accurately describe the
submitted system.

Where local scripts were used for tasks such as comparing predictions,
organizing outputs, formatting labels, or preparing files, those scripts are
included separately if they were actually used.

---

## 9. Evidence Included

The Track 1 evidence package may contain:

```text
README.md
Final_Submission/
Prompts/
LLM_Outputs/
Screenshots/
Audit/
Code/
```

The relevant directories contain, where available:

- the final Track 1 prediction file;
- few-shot prompts;
- few-shot examples;
- GPT outputs;
- Gemini outputs;
- Qwen outputs;
- screenshots/conversation records;
- GPT Audit prompt;
- GPT Audit output/report;
- comparison or post-processing code actually used.

Only materials that were actually available or used are represented as system
evidence. No API implementation is claimed where none was used.

---

## 10. Reproducibility

Because the LLMs were accessed through interactive chat interfaces rather
than through a unified API inference pipeline, exact programmatic reproduction
of every interactive response cannot be guaranteed.

The evidence package nevertheless documents the prediction methodology and
preserves the available:

- few-shot prompts;
- examples;
- model outputs;
- screenshots;
- audit procedure;
- decision process; and
- final prediction file.

These materials provide the available record of how the Track 1 predictions
were generated.

---

## 11. Final Submission Record

The final prediction file included in this Track 1 package corresponds to
the prediction file submitted to the StanceEval evaluation leaderboard.

The final predictions represent the output of the complete workflow:

```text
GPT Few-Shot
      +
Gemini Few-Shot
      +
Qwen Few-Shot
      |
      v
Cross-Model Comparison
      |
      v
GPT Audit for Difficult / Disagreement Cases
      |
      v
Evidence-Based Manual Adjudication
      |
      v
Final Track 1 Prediction
```

The final prediction is therefore not claimed to be the unmodified output of
any single LLM.

---

## 12. Documentation Integrity

This README distinguishes between:

- individual LLM predictions;
- cross-model comparison;
- GPT Audit analysis;
- manual/evidence-based adjudication; and
- the final Track 1 prediction.

The audit was used to support the decision process and was not assumed to be
perfect.

The purpose of this documentation is to accurately describe the system that
generated the submitted predictions without introducing an API, automated
pipeline, or trained checkpoint that was not actually used.
