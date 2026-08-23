# SRK AI Lab at StanceEval-2026: Multi-LLM Few-Shot Prompting with Evidence-Guided Auditing for Arabic Stance Detection

## LLM-Based Prediction System Documentation

**Team:** SRK AI Lab  
**Track:** Track 1 — Seen Targets  
**Task:** StanceEval-2026 Arabic Stance Detection  
**Benchmark:** Mawqif-v2

---

## 1. Purpose

This repository documents the prediction workflow, prompting strategy, model
configuration, and supporting evidence used for the SRK AI Lab submission to
the **StanceEval-2026 shared task on Arabic stance detection**.

The system is based on a **few-shot multi-LLM framework** using three
independently prompted large language models:

- **GPT-5.5 Instant**
- **Gemini 3.6 Flash with Extended Thinking**
- **Qwen 3.7 Plus**

The system classifies Arabic target-post pairs into the three StanceEval
labels:

- `Favor`
- `Against`
- `None`

Each LLM independently predicts the stance of the post toward the specified
target using task-specific instructions and class-balanced few-shot
demonstrations. The predictions are subsequently compared and examined
through a GPT-5.5 audit stage. The final label is determined through
**evidence-based adjudication** using the model predictions, audit
observations, and the target-post evidence.

The same overall framework was developed for both seen-target and
unseen-target settings. This README focuses specifically on the **Track 1
seen-target system**.

The corresponding system description paper is:

> **SRK AI Lab at StanceEval-2026: Multi-LLM Few-Shot Prompting with
> Evidence-Guided Auditing for Arabic Stance Detection**

The paper reports an official **Track 1 Favg2 of 0.8863, ranking 4th**.

---

## 2. Task and Dataset

### 2.1 Stance Detection Task

Stance detection determines whether a text expresses support, opposition, or
no clear position toward a **specific target**.

This differs from general sentiment classification because the same post can
have different interpretations depending on the target being considered.

For StanceEval-2026, the system receives:

- a target;
- an Arabic social-media post; and
- the required stance-label space.

The system predicts one of:

```text
Favor
Against
None
```

The prediction must be made with respect to the **given target only**.

The system therefore does not simply classify the overall positive or
negative sentiment of the post. It determines the author's stance toward the
specified target.

---

### 2.2 Mawqif-v2 Benchmark

The system was evaluated using the **Mawqif-v2** benchmark for Arabic
target-specific stance detection.

StanceEval-2026 contains both:

- **Track 1 — Seen Targets**
- **Track 2 — Unseen Targets**

This repository documents **Track 1**.

Track 1 contains three targets that are represented in the training data:

1. **COVID-19 Vaccine**
2. **Digital Transformation**
3. **Women Empowerment**

The Track 1 training set contains **3,502 instances**, while the development
set contains **619 instances**.

The Track 1 training distribution by target is:

| Target | Training Instances | Favor (%) | Against (%) | None (%) |
|---|---:|---:|---:|---:|
| COVID-19 Vaccine | 1,167 | 43.62 | 43.53 | 12.85 |
| Digital Transformation | 1,145 | 76.77 | 12.40 | 10.83 |
| Women Empowerment | 1,190 | 63.87 | 31.18 | 4.96 |

The training data were used to construct few-shot demonstrations. Development
instances remained separate and were not used as demonstrations.

---

## 3. Few-Shot LLM Prediction

The final system uses **few-shot in-context prompting** with three LLMs.

Each model receives:

1. task instructions;
2. stance-label definitions;
3. class-balanced demonstrations;
4. the target;
5. the Arabic post; and
6. instructions for returning the predicted stance label.

The three models operate as **independent prediction sources**.

### 3.1 GPT-5.5 Instant

GPT-5.5 Instant uses **24 class-balanced demonstrations**:

```text
Favor   : 8 examples
Against : 8 examples
None    : 8 examples
Total   : 24 examples
```

The prompt emphasizes:

- target-specific stance;
- distinction between stance and sentiment;
- implicit stance;
- negation;
- sarcasm;
- rhetorical questions;
- irony;
- target-specific interpretation; and
- avoiding `None` when Favor or Against is reasonably supported.

The model is instructed to output predictions in the format:

```text
ID,Label
```

For example:

```text
182,Favor
```

The prompt does not require the model to return explanations for the normal
prediction stage.

---

### 3.2 Gemini 3.6 Flash with Extended Thinking

Gemini 3.6 Flash with Extended Thinking also uses **24 class-balanced
demonstrations**:

```text
Favor   : 8 examples
Against : 8 examples
None    : 8 examples
Total   : 24 examples
```

The prompt emphasizes author-level stance and target-specific interpretation.

Particular attention is given to:

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

The prompt explicitly warns that positive wording does not necessarily imply
`Favor`, and negative wording does not necessarily imply `Against`.

Gemini's prompt also requests additional information during the relevant
analysis workflow, including:

```text
Label
Confidence
Difficulty
Reason
```

This allows difficult or ambiguous cases to be examined more carefully.

---

### 3.3 Qwen 3.7 Plus

Qwen 3.7 Plus uses a larger few-shot demonstration set containing:

```text
Favor   : 50 examples
Against : 50 examples
None    : 50 examples
Total   : 150 examples
```

The Qwen prompt emphasizes:

- author-level stance;
- target-specific interpretation;
- distinction between sentiment and stance;
- sarcasm;
- irony;
- rhetorical questions;
- implicit meaning;
- complete contextual interpretation; and
- selecting `None` only when the stance is genuinely unclear.

The prompt also instructs the model not to rely on isolated keywords and to
consider the complete Target–Post relationship.

The expected prediction format is:

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

### 3.4 Demonstration Construction

The few-shot demonstrations were constructed from the available
StanceEval-2026 training data.

The demonstrations are class-balanced for each model, but the number of
demonstrations differs between models:

| Model | Total | Favor | Against | None |
|---|---:|---:|---:|---:|
| GPT-5.5 Instant | 24 | 8 | 8 | 8 |
| Gemini 3.6 Flash with Extended Thinking | 24 | 8 | 8 | 8 |
| Qwen 3.7 Plus | 150 | 50 | 50 | 50 |

Development/evaluation instances were kept separate from the demonstration
examples.

The number of demonstrations is independent of the inference chunk size.

---

## 4. Input Preparation and Preprocessing

The system intentionally preserves the original textual information used in
the Arabic posts.

For few-shot demonstrations, training instances were formatted as:

```text
Target – Post – Stance
```

The original textual content was preserved.

In particular, the preprocessing stage did **not** remove or normalize away
potentially useful contextual signals such as:

- hashtags;
- emojis;
- punctuation; and
- other textual cues.

These elements may carry important information for interpreting stance,
especially in informal Arabic social-media text.

Evaluation instances were used as provided, without additional preprocessing.

Therefore, the system does not perform a conventional normalization pipeline
that removes hashtags, emojis, punctuation, or other contextual information.

This design was motivated by the possibility that such cues may contribute to
the interpretation of implicit stance, sarcasm, irony, or pragmatic meaning.

---

## 5. Prompting Strategy

### 5.1 Zero-Shot Development

Zero-shot prompting was considered during system development.

The zero-shot configuration provided:

- the task definition;
- the target;
- the Arabic post; and
- the stance-label descriptions.

No demonstrations were provided in the zero-shot setting.

---

### 5.2 Few-Shot Development

The final system uses few-shot prompting.

Few-shot prompts additionally provide class-balanced examples of:

- `Favor`
- `Against`
- `None`

The prompts use English instructions and label descriptions, while the
target and Arabic post remain in Arabic.

The prompting strategy is based on target-specific stance interpretation
rather than sentiment classification.

The models are explicitly instructed to consider implicit stance and
linguistic phenomena such as:

- negation;
- sarcasm;
- irony;
- rhetorical questions;
- implicit support;
- implicit opposition; and
- contextual meaning.

---

### 5.3 Target-Specific Reasoning

A central principle of the prompts is:

```text
Determine the stance toward the TARGET,
not the general sentiment of the POST.
```

For example, a post can contain positive language while criticizing the
target, or contain negative language while criticizing an opponent of the
target.

Therefore:

```text
Positive wording ≠ automatically Favor
Negative wording ≠ automatically Against
```

The models are instructed to identify the author's actual stance toward the
specified target.

---

### 5.4 Handling `None`

The `None` label is used when a Favor or Against stance cannot reasonably be
inferred.

Examples include posts that are:

- unrelated to the target;
- purely factual;
- neutral questions;
- discussions of both sides without taking a position;
- ambiguous with no sufficiently supported stance; or
- merely mentioning the target without expressing support or opposition.

The prompts emphasize that `None` should not be selected simply because the
stance is implicit. If the text provides sufficient evidence for Favor or
Against, the implicit stance should be considered.

---

## 6. Multi-LLM Prediction and Comparison

After the three independent few-shot prediction stages, the outputs of:

- GPT-5.5 Instant;
- Gemini 3.6 Flash with Extended Thinking; and
- Qwen 3.7 Plus

are compared.

The comparison identifies:

- cases where all models agree; and
- cases where the models disagree.

### 6.1 Agreement

When all three models produce the same label, the agreed prediction provides
strong supporting evidence.

However, agreement is not treated as an unconditional guarantee of
correctness.

The target-post relationship remains important for the final decision.

---

### 6.2 Disagreement

When the models disagree, the case is treated as more difficult and receives
additional examination.

The review focuses on linguistic and contextual phenomena such as:

- target scope;
- stance-holder identification;
- sarcasm;
- reversal;
- rhetorical ambiguity;
- implicit meaning;
- negation;
- irony; and
- contextual interpretation.

The purpose of the comparison stage is therefore not simply to perform
majority voting. Instead, model disagreement helps identify cases where
target-aware linguistic interpretation is particularly important.

---

## 7. GPT-5.5 Audit and Evidence-Based Adjudication

### 7.1 GPT-5.5 Audit

Following the initial predictions and cross-model comparison, the relevant
prediction information is examined through a **GPT-5.5 Instant audit stage**.

For Track 1, the audit receives:

- the original target;
- the original Arabic tweet/post; and
- predictions from GPT-5.5 Instant, Qwen 3.7 Plus, and Gemini 3.6 Flash
  with Extended Thinking.

The audit considers the Target–Post relationship and relevant linguistic
evidence.

The audit is an additional reasoning and review stage.

It is **not treated as an independent fourth LLM prediction source**.

GPT-5.5 Instant therefore has two roles:

1. initial few-shot stance prediction; and
2. subsequent audit of the prediction information.

---

### 7.2 Evidence-Based Adjudication

The audit does not directly determine the final label.

Instead, the final label is determined through **evidence-based
adjudication** using:

- the three model predictions;
- the GPT-5.5 audit observations; and
- the evidence contained in the Target–Post pair.

The workflow therefore does not apply a rule such as:

```text
Always follow the GPT-5.5 audit.
```

Instead, the audit provides additional evidence and analysis that is
considered alongside the original predictions and the actual linguistic
relationship between the target and post.

The final decision is consequently target-aware and evidence-guided.

---

## 8. Final Prediction Pipeline

The complete Track 1 workflow is:

```text
                 Target + Arabic Post
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
 GPT-5.5 Instant     Gemini 3.6 Flash   Qwen 3.7 Plus
    Few-Shot        Extended Thinking      Few-Shot
        |                |                |
        +----------------+----------------+
                         |
                         v
                Model Predictions
                         |
                         v
              Cross-Model Comparison
                         |
             +-----------+-----------+
             |                       |
          Agreement             Disagreement /
             |                   Difficult Case
             |                       |
             +-----------+-----------+
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

The same conceptual workflow was used for the final system, while the
specific demonstration configurations differed between models.

---

## 9. LLM Access and Inference

The LLMs were accessed through their respective **chat interfaces**.

The final Track 1 system used:

- GPT-5.5 Instant;
- Gemini 3.6 Flash with Extended Thinking; and
- Qwen 3.7 Plus.

No API-based or SDK-based automated inference pipeline was used for the LLM
interactions.

### 9.1 Track 1 Inference Chunking

For Track 1, evaluation instances were processed in chunks of **50
instances** for each model.

| Model | Track 1 Chunk Size |
|---|---:|
| GPT-5.5 Instant | 50 |
| Gemini 3.6 Flash with Extended Thinking | 50 |
| Qwen 3.7 Plus | 50 |

The chunk size is separate from the few-shot demonstration count.

For example:

```text
GPT-5.5:
24 demonstrations + evaluation chunks of 50

Gemini:
24 demonstrations + evaluation chunks of 50

Qwen:
150 demonstrations + evaluation chunks of 50
```

Prompts and model outputs were recorded as part of the experimental and
verification materials.

---

### 9.2 Model Training

The final system did **not** use task-specific fine-tuning.

No:

- task-specific parameter updates;
- trained stance classifier;
- parameter-level model fusion; or
- fine-tuned Arabic stance checkpoint

was used in the final multi-LLM system.

The system is based on prompting and evidence-guided prediction review.

---

## 10. Evaluation

### 10.1 Primary Metric

The primary StanceEval-2026 ranking metric is **Favg2**.

Favg2 is the macro-average of F1 for:

- `Favor`
- `Against`

The `None` class is not included in Favg2.

---

### 10.2 Favg3

**Favg3** is the macro-average of F1 across all three stance classes:

- `Favor`
- `Against`
- `None`

---

### 10.3 Accuracy

Accuracy is additionally reported as an overall classification measure.

Official submission metrics are distinguished from metrics obtained during
post-submission or development analyses.

---

## 11. Official Track 1 Results

The final SRK AI Lab multi-LLM system achieved the following official
StanceEval-2026 Track 1 result:

| Metric | Track 1 |
|---|---:|
| **Favg2** | **0.8863** |
| **Favg3** | **0.7111** |
| **Accuracy** | **0.8409** |
| **Rank** | **4th** |

The official Track 1 Favg2 of **0.8863** is the primary leaderboard result.

The final system combines the three LLM predictions with GPT-5.5 auditing
and evidence-based adjudication.

---

### 11.1 Retained Development Results

The paper also reports retained individual-model development results.

These results are presented separately from the official final multi-LLM
submission:

| Track | Model/System | Configuration | Favg2 | Favg3 | Accuracy |
|---|---|---|---:|---:|---:|
| Track 1 | Qwen 3.7 Plus | Zero-shot | 0.7991 | 0.6148 | 0.7472 |
| Track 1 | Qwen 3.7 Plus | Few-shot (150) | 0.8174 | 0.6168 | 0.7614 |
| Track 1 | Gemini 3.6 Flash | Zero-shot | 0.8260 | 0.6519 | 0.7670 |
| Track 1 | Gemini 3.6 Flash | Few-shot (24) | 0.8366 | 0.6494 | 0.7727 |
| Track 1 | Final Multi-LLM System | Multi-LLM + Audit + Evidence-Based Adjudication | **0.8863** | **0.7111** | **0.8409** |

These retained development results are provided for context.

They should **not** be interpreted as a complete component-wise ablation of
every stage of the final system, because the individual contributions of
cross-model comparison, GPT-5.5 auditing, and evidence-based adjudication
were not independently evaluated.

---

## 12. Error Analysis and Difficult Cases

Representative validation/development cases were examined qualitatively
because gold labels were not available for the final test data.

The analysis focused on cases where stance depends on:

- the target;
- the stance-holder;
- the author's intended meaning;
- contextual interpretation;
- implicit cues; or
- pragmatic phenomena.

Representative error categories included:

| Track | Error Type | Gold | GPT | Qwen | Gemini |
|---|---|---|---|---|---|
| Track 1 | Sarcasm / reversal | Favor | Against | Against | Against |
| Track 1 | Speaker attribution | None | Against | Favor | Favor |
| Track 2 | Target scope | Favor | Favor | Favor | Against |
| Track 2 | Rhetorical ambiguity | None | Against | Against | Against |

For Track 1, the analysis showed that models can confuse criticism of
anti-vaccination claims with opposition to vaccination itself.

Another challenge is **speaker attribution**, where a post may express or
quote the position of another person rather than the author's own stance.

These cases demonstrate why target-specific interpretation is important and
why surface sentiment alone is insufficient for stance detection.

---

## 13. Evidence Included

The Track 1 supporting materials may include the following structure:

```text
README.md

Final_Submission/
    Track1_Final_Predictions.txt

Prompts/
    Track1_GPT-5.5_Prompt.txt
    Track1_Gemini_Prompt.txt
    Track1_Qwen_Prompt.txt
    Track1_GPT-5.5_Audit_Prompt.txt

Demonstrations/
    Track1_GPT-5.5_Demonstrations/
    Track1_Gemini_Demonstrations/
    Track1_Qwen_Demonstrations/

LLM_Outputs/
    GPT-5.5/
    Gemini/
    Qwen/

Audit/
    Track1_GPT-5.5_Audit/

Comparison/
    Track1_Prediction_Comparison/

Screenshots/
    GPT-5.5/
    Gemini/
    Qwen/

Code/
    Prediction_Comparison/
    Submission_Preparation/
```

The exact directories included in the repository should correspond only to
the materials actually released with the project.

The evidence package is intended to document:

- few-shot prompts;
- demonstration examples;
- LLM prediction outputs;
- GPT-5.5 audit materials;
- prediction comparison;
- final prediction file; and
- supporting scripts used for comparison or submission preparation.

No API implementation should be claimed if it was not used.

---

## 14. Reproducibility

The system uses interactive chat interfaces rather than a unified API-based
inference pipeline.

Consequently, exact programmatic reproduction of every interactive model
response cannot be guaranteed.

Nevertheless, the repository documents the major components of the
prediction process, including:

- task instructions;
- few-shot prompts;
- demonstration configurations;
- model names;
- prediction outputs where available;
- inference chunking;
- GPT-5.5 audit procedure;
- cross-model comparison;
- evidence-based adjudication; and
- the final Track 1 prediction file.

The prompts and demonstration materials used for the Track 1 configurations
are provided as supporting evidence where available.

The repository therefore provides a transparent record of the actual
prediction workflow without claiming an API or automated inference system
that was not used.

---

## 15. Limitations

The final system was based on prompted LLM inference rather than
task-specific fine-tuning.

Limited GPU resources, time, and inference constraints restricted the
exploration of fine-tuned Arabic stance models during the project.

The interactive nature of the LLM interfaces also limits exact
programmatic reproducibility of every individual response.

In addition, stance detection remains difficult when the intended stance is
implicit or depends on:

- target scope;
- stance-holder identification;
- sarcasm;
- rhetorical language;
- irony;
- contextual meaning; or
- other pragmatic cues.

The final multi-LLM workflow was designed to address some of these
challenges through independent predictions and GPT-5.5 evidence-guided
auditing, but these mechanisms do not guarantee correct interpretation of
every ambiguous case.

Future work will investigate fine-tuned Arabic LLMs and improved prompt
optimization.

---

## 16. Documentation Integrity

This README distinguishes clearly between:

1. individual LLM predictions;
2. cross-model comparison;
3. GPT-5.5 auditing;
4. evidence-based adjudication; and
5. the final Track 1 prediction.

The three LLMs are independent prediction sources:

```text
GPT-5.5 Instant
Gemini 3.6 Flash with Extended Thinking
Qwen 3.7 Plus
```

GPT-5.5 Instant is also used in the subsequent audit stage. It is therefore
not treated as a fourth independent model.

The GPT-5.5 audit provides supporting analysis and does not by itself
determine the final label.

The final label is determined from the available model predictions, audit
observations, and Target–Post evidence.

The official leaderboard result is kept separate from retained individual
model development results.

No unsupported claims are made regarding:

- API-based inference;
- automated API querying;
- task-specific fine-tuning;
- parameter-level fusion;
- majority voting as the final decision rule; or
- component-wise performance improvements that were not independently
  evaluated.

The purpose of this documentation is to accurately describe the system that
generated the submitted Track 1 predictions.

---

## 17. Repository Contents

A recommended repository organization is:

```text
SRK_AI_Lab_StanceEval_2026/
│
├── README.md
│
├── Final_Submission/
│   └── Track1_Final_Predictions.txt
│
├── Prompts/
│   ├── Track1_GPT-5.5_Instant_Prompt.txt
│   ├── Track1_Gemini_3.6_Flash_Prompt.txt
│   ├── Track1_Qwen_3.7_Plus_Prompt.txt
│   └── Track1_GPT-5.5_Audit_Prompt.txt
│
├── Demonstrations/
│   ├── Track1_GPT-5.5_24_Demonstrations/
│   ├── Track1_Gemini_24_Demonstrations/
│   └── Track1_Qwen_150_Demonstrations/
│
├── LLM_Outputs/
│   ├── GPT-5.5/
│   ├── Gemini/
│   └── Qwen/
│
├── Audit/
│   └── Track1_GPT-5.5_Audit/
│
├── Comparison/
│   └── Track1_Prediction_Comparison/
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

Directory names may be adjusted to match the actual released repository
structure.

---

## 18. Track 1 System Summary

The final Track 1 system can be summarized as follows:

```text
Mawqif-v2 Track 1
Seen Targets
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

The final Track 1 system achieved:

```text
Favg2   : 0.8863
Favg3   : 0.7111
Accuracy: 0.8409
Rank    : 4th
```

---

## 19. Acknowledgment

The authors thank the **StanceEval-2026 organizers** and **Mawqif-v2
contributors** for their support.

---

## 20. Citation

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

## 21. Final System Statement

The SRK AI Lab Track 1 system is a **few-shot multi-LLM framework for
target-specific Arabic stance detection**.

GPT-5.5 Instant, Gemini 3.6 Flash with Extended Thinking, and Qwen 3.7 Plus
independently predict the stance of each Target–Post pair using
class-balanced demonstrations. Their predictions are compared, examined
through a GPT-5.5 audit, and resolved through evidence-based adjudication
that considers the model predictions, audit observations, and the
Target–Post relationship.

The final system achieved an official:

**Track 1 Favg2 = 0.8863 (4th place).**