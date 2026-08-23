# StanceEval 2026 — Track 2

## LLM-Based Prediction System Documentation

**Team:** SRK AI Lab  
**Track:** Track 2 — Unseen Targets  
**Task:** StanceEval-2026 Arabic Stance Detection  
**Benchmark:** Mawqif-v2

---

## 1. Purpose

This repository documents the prediction workflow, prompting strategy, model
configuration, and supporting evidence used for the SRK AI Lab submission to
the **StanceEval-2026 shared task on Arabic stance detection**.

The Track 2 system uses a **few-shot multi-LLM framework** with three
independently prompted large language models:

- **GPT-5.5 Instant**
- **Gemini 3.6 Flash with Extended Thinking**
- **Qwen 3.7 Plus**

The system classifies Arabic target-post pairs into the three StanceEval
labels:

- `Favor`
- `Against`
- `None`

Each LLM independently predicts the stance toward the specified target using
task-specific instructions and class-balanced few-shot demonstrations. The
three predictions are then compared and examined through a GPT-5.5 audit
stage. The final label is determined through **evidence-based adjudication**
using the model predictions, audit observations, and the Target–Post
evidence.

Track 2 focuses on **cross-target generalization**, requiring the system to
handle targets that are unseen during the relevant training setting.

The official Track 2 result reported in the system description paper is an
**Favg2 of 0.9518, ranking 3rd**.

---

## 2. Task and Dataset

### 2.1 Unseen-Target Stance Detection

Track 2 evaluates the ability of a stance detection system to generalize
across targets that are not represented in the corresponding training
setting.

For each instance, the system receives:

- a Target;
- an Arabic social-media post; and
- the required stance-label space.

The system predicts one of:

```text
Favor
Against
None
```

The prediction is made with respect to the **specific Target** associated
with the post.

The system therefore does not treat the task as general sentiment
classification. Instead, it determines whether the author supports,
opposes, or takes no clear stance toward the given target.

---

### 2.2 Mawqif-v2 Benchmark

The system uses the **Mawqif-v2** benchmark for Arabic target-specific stance
detection.

StanceEval-2026 evaluates both:

- **Track 1 — Seen Targets**
- **Track 2 — Unseen Targets**

This README focuses specifically on **Track 2**.

The Track 2 training data cover:

1. **COVID-19 Vaccine**
2. **Digital Transformation**

The Track 2 training set contains **2,721 instances**, while the development
set contains **1,400 instances**.

The Track 2 training distribution is:

| Target | Training Instances | Favor (%) | Against (%) | None (%) |
|---|---:|---:|---:|---:|
| COVID-19 Vaccine | 1,373 | 43.63 | 43.55 | 12.82 |
| Digital Transformation | 1,348 | 76.78 | 12.39 | 10.83 |

The Track 2 development and evaluation settings contain targets that are
unseen during training, making cross-target generalization a central
requirement of the task.

---

## 3. Few-Shot LLM Prediction

The final Track 2 system uses **few-shot in-context prompting** with three
LLMs.

Each model receives:

1. task instructions;
2. stance-label definitions;
3. class-balanced demonstrations;
4. the Target;
5. the Arabic post; and
6. instructions for producing the stance prediction.

The three models act as **independent prediction sources**.

### 3.1 GPT-5.5 Instant

GPT-5.5 Instant uses **24 class-balanced demonstrations**:

```text
Favor   : 8 examples
Against : 8 examples
None    : 8 examples
Total   : 24 examples
```

The Track 2 prompt emphasizes:

- target-specific stance;
- distinguishing stance from sentiment;
- implicit stance;
- negation;
- sarcasm;
- rhetorical questions;
- irony;
- target-specific interpretation; and
- avoiding `None` when Favor or Against can reasonably be inferred.

The model is instructed to return predictions in the format:

```text
ID,Label
```

For example:

```text
182,Favor
```

---

### 3.2 Gemini 3.6 Flash with Extended Thinking

Gemini 3.6 Flash with Extended Thinking uses **24 class-balanced
demonstrations**:

```text
Favor   : 8 examples
Against : 8 examples
None    : 8 examples
Total   : 24 examples
```

The Track 2 prompt emphasizes author-level stance and target-specific
interpretation.

It gives particular attention to phenomena including:

- sarcasm;
- irony;
- mockery;
- satire;
- self-deprecating jokes;
- hyperbole;
- rhetorical questions;
- emoji meaning;
- cultural expressions;
- quoted speech;
- repeated hashtags; and
- hashtag reversal.

The prompt also emphasizes that positive wording does not necessarily mean
`Favor`, while negative wording does not necessarily mean `Against`.

Gemini's Track 2 prompt requests:

```text
Label
Confidence
Difficulty
Reason
```

for the relevant analysis.

---

### 3.3 Qwen 3.7 Plus

Qwen 3.7 Plus uses **150 class-balanced demonstrations**:

```text
Favor   : 50 examples
Against : 50 examples
None    : 50 examples
Total   : 150 examples
```

The Track 2 Qwen prompt emphasizes:

- author-level stance;
- target-specific interpretation;
- distinction between stance and sentiment;
- sarcasm;
- irony;
- rhetorical questions;
- implicit meaning;
- contextual interpretation; and
- generalization to targets not encountered during training.

The prompt explicitly encourages the model to infer the stance from the
meaning of the complete Target–Post pair rather than memorizing target-specific
patterns.

The expected output format is:

```text
ID,Label
```

For example:

```text
101,Favor
102,None
103,Against
```

---

### 3.4 Demonstration Configuration

The final Track 2 few-shot configurations are:

| Model | Total Demonstrations | Favor | Against | None |
|---|---:|---:|---:|---:|
| GPT-5.5 Instant | 24 | 8 | 8 | 8 |
| Gemini 3.6 Flash with Extended Thinking | 24 | 8 | 8 | 8 |
| Qwen 3.7 Plus | 150 | 50 | 50 | 50 |

The demonstrations are class-balanced for each model.

The demonstration count is independent of the evaluation inference chunk
size.

---

## 4. Input Preparation and Preprocessing

For few-shot demonstrations, the training instances were formatted as
Target–Post–Stance examples while preserving the original textual content.

The system intentionally retains textual cues that may contribute to stance
interpretation, including:

- hashtags;
- emojis;
- punctuation; and
- other contextual textual signals.

Evaluation instances were used as provided, without additional preprocessing.

This preserves potentially useful information for interpreting:

- implicit stance;
- sarcasm;
- irony;
- rhetorical expressions; and
- other pragmatic signals.

The system therefore does not apply a conventional preprocessing pipeline
that removes hashtags, emojis, punctuation, or other potentially informative
textual cues.

---

## 5. Prompting Strategy

### 5.1 Zero-Shot Development

Zero-shot prompting was considered during system development.

The zero-shot setting provided:

- the task definition;
- the target;
- the Arabic post; and
- the stance-label descriptions.

No demonstrations were included in the zero-shot configuration.

---

### 5.2 Few-Shot Prompting

The final Track 2 system uses few-shot prompting.

The few-shot prompts additionally provide class-balanced demonstrations for:

- `Favor`;
- `Against`; and
- `None`.

The prompts use English task instructions and label descriptions while the
Target and Arabic Post remain in Arabic.

The purpose of the demonstrations is to communicate the task and provide
examples of how the three stance labels are distinguished.

Because Track 2 evaluates unseen targets, the models must use the semantic
relationship between the Target and Post rather than relying on
target-specific memorization.

---

### 5.3 Target-Specific Interpretation

A central principle of the Track 2 prompts is:

```text
Determine the author's stance toward the given TARGET,
not the general sentiment of the POST.
```

The model therefore needs to determine:

1. what the post is discussing;
2. how that content relates to the specified target;
3. whose opinion is being expressed; and
4. whether that opinion supports, opposes, or does not clearly take a
   position toward the target.

This target-aware interpretation is particularly important in the
unseen-target setting.

---

### 5.4 Generalization to Unseen Targets

The Track 2 prompts explicitly require the models to generalize from the
semantic meaning of the Target–Post pair.

The system does not assume that the target itself appeared in the training
examples.

Instead, the model is expected to use:

- the meaning of the target;
- the meaning of the Arabic post;
- contextual relationships; and
- the stance expressed by the author.

This allows the few-shot examples to communicate the task structure while
the model performs target-specific interpretation for unseen targets.

---

## 6. Cross-Model Comparison

After the three independent LLM prediction stages, the outputs of:

- GPT-5.5 Instant;
- Gemini 3.6 Flash with Extended Thinking; and
- Qwen 3.7 Plus

are compared.

The comparison identifies:

- agreement between models; and
- disagreement or difficult cases.

### 6.1 Agreement

When the models agree on a stance label, the agreement provides supporting
evidence for that prediction.

However, agreement is not treated as an unconditional guarantee of
correctness.

The Target–Post relationship remains central to the final decision.

---

### 6.2 Disagreement or Difficult Cases

When the models disagree, the case receives additional examination.

Particular attention is given to:

- target scope;
- stance-holder identification;
- implicit stance;
- negation;
- sarcasm;
- irony;
- rhetorical questions;
- contextual meaning;
- quoted speech;
- cultural expressions; and
- posts that discuss a topic without expressing a stance toward the
  specified target.

Cross-model disagreement therefore acts as a useful signal for identifying
cases requiring closer target-aware interpretation.

---

## 7. GPT-5.5 Audit

A dedicated **GPT-5.5 Instant audit stage** is used after the initial
multi-LLM predictions.

For Track 2, the audit procedure uses the relevant Target–Post information
and the predictions produced by:

- GPT-5.5 Instant;
- Qwen 3.7 Plus; and
- Gemini 3.6 Flash with Extended Thinking.

The audit examines the relationship between the Target and Post and considers
linguistic evidence relevant to the stance decision.

The GPT-5.5 audit is used as an **additional reasoning and quality-control
stage**.

It is not treated as an infallible source of ground truth.

The audit output is also not used as an unconditional automatic
label-replacement rule.

Instead, the audit can:

- support an existing prediction;
- identify a potentially problematic interpretation;
- provide additional linguistic evidence; or
- help resolve a difficult or conflicting case.

GPT-5.5 Instant therefore serves two roles in the Track 2 workflow:

1. independent few-shot prediction model; and
2. audit model after prediction comparison.

It is **not treated as a fourth independent LLM**.

---

## 8. Evidence-Based Adjudication

The final Track 2 prediction is determined through **evidence-based
adjudication**.

The decision considers:

- the GPT-5.5 prediction;
- the Gemini prediction;
- the Qwen prediction;
- GPT-5.5 audit observations; and
- the linguistic evidence in the Target–Post pair.

The final decision is therefore not based solely on the GPT-5.5 audit.

The workflow does not use a rule such as:

```text
Always follow the GPT-5.5 Audit.
```

Instead, the final label is selected after considering the available model
predictions, audit observations, and the actual Target–Post relationship.

This is particularly important for Track 2 because the targets are unseen
relative to the training setting and require semantic cross-target
generalization.

---

## 9. Final Prediction Pipeline

The complete Track 2 workflow can be summarized as:

```text
                    Target + Arabic Post
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
   GPT-5.5 Instant    Gemini 3.6 Flash    Qwen 3.7 Plus
      Few-Shot        Extended Thinking      Few-Shot
          |                 |                 |
          +-----------------+-----------------+
                            |
                            v
                  Independent Predictions
                            |
                            v
                   Cross-Model Comparison
                            |
                            v
                      GPT-5.5 Audit
                            |
                            v
                 Target–Post Examination
                            |
                            v
                Evidence-Based Adjudication
                            |
                            v
                    Final Stance Label
                    Favor / Against / None
```

The final prediction is therefore produced from the complete multi-stage
workflow rather than by directly selecting the output of one individual
model.

---

## 10. LLM Access and Inference

The Track 2 LLM predictions were generated through direct interaction with
the respective chat interfaces:

- GPT-5.5 Instant;
- Gemini 3.6 Flash with Extended Thinking; and
- Qwen 3.7 Plus.

No API-based or SDK-based automated inference pipeline was used for the LLM
interactions.

### 10.1 Track 2 Inference Chunking

The Track 2 evaluation instances were processed in chunks.

The final configuration used:

| Model | Track 2 Chunk Size |
|---|---:|
| GPT-5.5 Instant | 50 |
| Gemini 3.6 Flash with Extended Thinking | 30 |
| Qwen 3.7 Plus | 50 |

Gemini used a smaller chunk size of 30 for Track 2 evaluation processing.

The chunk size is independent of the number of few-shot demonstrations.

For example:

```text
GPT-5.5:
24 demonstrations + evaluation chunks of 50

Gemini:
24 demonstrations + evaluation chunks of 30

Qwen:
150 demonstrations + evaluation chunks of 50
```

Prompts and outputs were recorded as part of the experimental and
verification materials.

---

### 10.2 Model Training

The final system does not use task-specific fine-tuning.

No:

- task-specific parameter updates;
- trained stance classifier;
- parameter-level model fusion; or
- fine-tuned Arabic stance checkpoint

was used in the final multi-LLM system.

The Track 2 system relies on few-shot prompting, independent predictions,
GPT-5.5 auditing, and evidence-based adjudication.

---

## 11. Evaluation

### 11.1 Primary Metric

The primary StanceEval-2026 ranking metric is **Favg2**.

Favg2 is the macro-average of F1 for:

- `Favor`
- `Against`

The `None` class is not included in Favg2.

---

### 11.2 Favg3

**Favg3** is the macro-average of F1 across all three stance classes:

- `Favor`;
- `Against`; and
- `None`.

---

### 11.3 Accuracy

Accuracy is additionally reported as an overall classification measure.

Official leaderboard results are kept distinct from retained development and
post-submission analyses.

---

## 12. Official Track 2 Results

The final SRK AI Lab multi-LLM system achieved the following official
StanceEval-2026 Track 2 result:

| Metric | Track 2 |
|---|---:|
| **Favg2** | **0.9518** |
| **Favg3** | **0.9053** |
| **Accuracy** | **0.9332** |
| **Rank** | **3rd** |

The official Track 2 **Favg2 of 0.9518** is the primary leaderboard result.

The final system combines the three independently prompted LLMs with GPT-5.5
auditing and evidence-based adjudication.

---

### 12.1 Retained Development Results

The paper reports retained individual prompting results separately from the
official final multi-LLM system.

The reported Track 2 retained results are:

| Track | Model | Configuration | Favg2 | Favg3 | Accuracy |
|---|---|---|---:|---:|---:|
| Track 2 | Gemini 3.6 Flash | Zero-shot | 0.8758 | 0.7886 | 0.8354 |
| Track 2 | Gemini 3.6 Flash | Few-shot (24) | 0.9185 | 0.8410 | 0.8929 |
| Track 2 | Final Multi-LLM System | Multi-LLM + Audit + Evidence-Based Adjudication | **0.9518** | **0.9053** | **0.9332** |

These retained results provide development context for the final system.

They are not presented as a complete component-wise ablation because the
individual numerical contributions of cross-model comparison, GPT-5.5
auditing, and evidence-based adjudication were not independently evaluated.

---

## 13. Error Analysis and Difficult Cases

Representative validation cases with available gold labels were examined
qualitatively.

Because gold labels were not available for the final test data, the error
analysis uses validation cases rather than estimating test-set error
frequencies.

The analysis focuses on cases where stance depends on:

- target scope;
- stance-holder identification;
- implicit meaning;
- contextual interpretation;
- rhetorical language; and
- other pragmatic cues.

Representative difficult cases included:

| Track | Error Type | Gold | GPT | Qwen | Gemini |
|---|---|---|---|---|---|
| Track 1 | Sarcasm / reversal | Favor | Against | Against | Against |
| Track 1 | Speaker attribution | None | Against | Favor | Favor |
| Track 2 | Target scope | Favor | Favor | Favor | Against |
| Track 2 | Rhetorical ambiguity | None | Against | Against | Against |

For Track 2, the analysis highlights difficulties where criticism or
discussion surrounding a target can be interpreted as opposition to the
target itself.

For example, criticism related to gender representation may be mistaken for
opposition to **Women Empowerment**, while rhetorical questions may be
interpreted as explicit opposition despite a `None` stance.

These cases demonstrate that unseen-target stance detection requires
attention to the target, stance-holder, and intended meaning rather than
surface-level sentiment alone.

---

## 14. Evidence Included

The Track 2 supporting materials may include the following structure:

```text
README.md

Final_Submission/
    Track2_Final_Predictions.txt

Prompts/
    Track2_GPT-5.5_Instant_Prompt.txt
    Track2_Gemini_3.6_Flash_Prompt.txt
    Track2_Qwen_3.7_Plus_Prompt.txt
    Track2_GPT-5.5_Audit_Prompt.txt

Demonstrations/
    Track2_GPT-5.5_24_Demonstrations/
    Track2_Gemini_24_Demonstrations/
    Track2_Qwen_150_Demonstrations/

LLM_Outputs/
    GPT-5.5/
    Gemini/
    Qwen/

Audit/
    Track2_GPT-5.5_Audit/

Comparison/
    Track2_Prediction_Comparison/

Screenshots/
    GPT-5.5/
    Gemini/
    Qwen/

Code/
    Prediction_Comparison/
    Submission_Preparation/
```

Directory names may be adjusted to match the actual released repository
structure.

The evidence package is intended to document, where available:

- few-shot prompts;
- few-shot demonstration examples;
- GPT-5.5 outputs;
- Gemini outputs;
- Qwen outputs;
- screenshots or conversation records;
- GPT-5.5 audit materials;
- prediction comparison;
- final prediction file; and
- supporting scripts used for comparison or submission preparation.

Only materials actually included or used should be represented as system
evidence.

No API implementation should be claimed where none was used.

---

## 15. Reproducibility

The LLMs were accessed through interactive chat interfaces rather than
through a unified API-based inference pipeline.

Consequently, exact programmatic reproduction of every individual interactive
response cannot be guaranteed.

The repository nevertheless documents the main components of the Track 2
prediction workflow, including:

- task instructions;
- few-shot prompts;
- demonstration configurations;
- model names;
- inference chunk sizes;
- model outputs where available;
- cross-model comparison;
- GPT-5.5 audit procedure;
- evidence-based adjudication; and
- the final Track 2 prediction file.

The available prompts and demonstration materials provide a record of the
few-shot configurations used during Track 2 prediction.

The repository therefore aims to provide transparent documentation of the
actual prompting, prediction, comparison, auditing, and evidence-based
review process without claiming an automated API inference pipeline that was
not used.

---

## 16. Limitations

The final system is based on prompted LLM inference rather than
task-specific fine-tuning.

Limited GPU resources, time, and inference constraints restricted the
exploration of fine-tuned Arabic stance models during the project.

The interactive nature of the LLM interfaces also limits exact programmatic
reproduction of every individual response.

Track 2 additionally presents the challenge of **cross-target
generalization**, since the evaluation setting includes targets that are
unseen during training.

Difficult cases can arise when stance depends on:

- target scope;
- stance-holder identification;
- sarcasm;
- irony;
- rhetorical language;
- implicit meaning;
- contextual interpretation; or
- cultural and pragmatic cues.

The multi-LLM and GPT-5.5 audit workflow provides additional evidence for
such cases, but it does not guarantee correct interpretation of every
ambiguous Target–Post pair.

Future work will investigate fine-tuned Arabic LLMs and improved prompt
optimization.

---

## 17. Documentation Integrity

This README distinguishes between:

1. individual LLM predictions;
2. cross-model comparison;
3. GPT-5.5 audit analysis;
4. evidence-based adjudication; and
5. the final Track 2 prediction.

The three independently prompted LLMs are:

```text
GPT-5.5 Instant
Gemini 3.6 Flash with Extended Thinking
Qwen 3.7 Plus
```

GPT-5.5 Instant also performs the subsequent audit stage. It is therefore
not treated as a fourth independent model.

The GPT-5.5 audit provides supporting analysis and does not automatically
determine the final label.

The final prediction is based on the available model predictions, audit
observations, and Target–Post linguistic evidence.

The official Track 2 leaderboard result is kept separate from retained
individual-model development results.

No unsupported claims are made regarding:

- API-based inference;
- automated API querying;
- task-specific fine-tuning;
- parameter-level model fusion;
- an unconditional audit-based label replacement rule; or
- component-wise performance improvements that were not independently
  evaluated.

The purpose of this documentation is to accurately describe the Track 2
system that generated the submitted predictions.

---

## 18. Repository Contents

A recommended Track 2 repository organization is:

```text
SRK_AI_Lab_StanceEval_2026/
│
├── README.md
│
├── Final_Submission/
│   └── Track2_Final_Predictions.txt
│
├── Prompts/
│   ├── Track2_GPT-5.5_Instant_Prompt.txt
│   ├── Track2_Gemini_3.6_Flash_Prompt.txt
│   ├── Track2_Qwen_3.7_Plus_Prompt.txt
│   └── Track2_GPT-5.5_Audit_Prompt.txt
│
├── Demonstrations/
│   ├── Track2_GPT-5.5_24_Demonstrations/
│   ├── Track2_Gemini_24_Demonstrations/
│   └── Track2_Qwen_150_Demonstrations/
│
├── LLM_Outputs/
│   ├── GPT-5.5/
│   ├── Gemini/
│   └── Qwen/
│
├── Audit/
│   └── Track2_GPT-5.5_Audit/
│
├── Comparison/
│   └── Track2_Prediction_Comparison/
│
├── Screenshots/
│   ├── GPT-5.5/
│   ├── Gemini/
│   └── Qwen/
│
└── Code/
    ├── Prediction_Comparison/
    └── Submission_Preparation/
```

Directory names may be adjusted to match the actual repository contents.

---

## 19. Track 2 System Summary

The complete Track 2 system can be summarized as:

```text
Mawqif-v2 Track 2
Unseen Targets
        |
        v
Target + Arabic Post
        |
        +----------------------+----------------------+
        |                      |                      |
        v                      v                      v
 GPT-5.5 Instant        Gemini 3.6 Flash        Qwen 3.7 Plus
 24 Few-Shot Examples   24 Few-Shot Examples    150 Few-Shot Examples
        |                      |                      |
        +----------------------+----------------------+
                               |
                               v
                     Independent Predictions
                               |
                               v
                    Cross-Model Comparison
                               |
                               v
                         GPT-5.5 Audit
                               |
                               v
                  Target–Post Evidence Review
                               |
                               v
                  Evidence-Based Adjudication
                               |
                               v
                    Final Stance Prediction
                    Favor / Against / None
```

The final Track 2 system achieved:

```text
Favg2   : 0.9518
Favg3   : 0.9053
Accuracy: 0.9332
Rank    : 3rd
```

---

## 20. Acknowledgment

The authors thank the **StanceEval-2026 organizers** and **Mawqif-v2
contributors** for their support.

---

## 21. Citation

If you use this system, repository, prompts, or associated materials, please
cite the corresponding StanceEval-2026 system description paper and the
relevant StanceEval-2026 and Mawqif-v2 resources.

### StanceEval-2026 Shared Task

```bibtex
@inproceedings{Albalawi-etal-2026-stanceeval,
    title = {StanceEval-2026: The Second Stance Detection Shared Task},
    author = {Albalawi, Rasha
        and Albadi, Nuha
        and Luqman, Hamzah
        and Ezzini, Saad
        and Kurdi, Maram
        and Yamani, Asma
        and Ashraf, Ahmed
        and Al-Shaibani, Maged
        and Alturayeif, Nora},
    booktitle = {Proceedings of the Fourth Arabic Natural Language Processing Conference (ArabicNLP 2026)},
    year = {2026},
    address = {Budapest, Hungary},
    publisher = {Association for Computational Linguistics},
}
```

### Mawqif-v2

```bibtex
@article{Albalawi-etal-2026-mawqif,
    title = {Mawqif-v2: An Arabic Benchmark Dataset for Cross-Target Stance Detection},
    author = {Albalawi, Rasha
        and Albadi, Nuha
        and Luqman, Hamzah
        and Kurdi, Maram
        and Ezzini, Saad
        and Yamani, Asma
        and Ashraf, Ahmed},
    year = {2026},
    journal = {arXiv preprint arXiv:2608.09539},
}
```

---

## 22. Final System Statement

The SRK AI Lab Track 2 system is a **few-shot multi-LLM framework for
cross-target Arabic stance detection**.

GPT-5.5 Instant, Gemini 3.6 Flash with Extended Thinking, and Qwen 3.7 Plus
independently predict the stance of each Target–Post pair using
class-balanced demonstrations. Their predictions are compared and examined
through a GPT-5.5 audit, after which the final label is selected through
evidence-based adjudication using the model predictions, audit observations,
and Target–Post linguistic evidence.

The system is designed to generalize beyond the targets represented in its
training demonstrations by relying on the semantic relationship between the
given target and Arabic post.

The official Track 2 result is:

**Favg2 = 0.9518 — Rank: 3rd.**