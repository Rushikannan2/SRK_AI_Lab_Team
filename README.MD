# SRK AI Lab — StanceEval-2026

## Multi-LLM Few-Shot Prompting with Evidence-Guided Auditing for Arabic Stance Detection

**Team:** SRK AI Lab  
**Shared Task:** StanceEval-2026 — ArabicNLP 2026 Shared Task on Stance Detection  
**Tracks:** Track 1 — Seen Targets and Track 2 — Unseen Targets

This repository contains the prediction systems, prompts, raw LLM outputs, screenshots, GPT-based audit materials, post-processing notebooks, and final submission files developed by **SRK AI Lab** for the StanceEval-2026 shared task.

## System Overview

The submitted systems use a few-shot multi-LLM framework for Arabic target-specific stance detection. Three independent large language models — **GPT-5.5 Instant, Gemini 3.6 Flash with Extended Thinking, and Qwen 3.7 Plus** — are used to classify Target–Post pairs into the three task labels: `Favor`, `Against`, and `None`.

The prediction workflow consists of three independent LLM predictions, followed by **GPT-5.5 auditing** and **cross-model comparison**. GPT-5.5 examines the three predictions against the original Target–Post pair and relevant linguistic and stance evidence. The audit is applied to both agreement and disagreement cases. The final decision considers the three model predictions, GPT-5.5 audit observations, and the Target–Post evidence to select the most supported stance label. :contentReference[oaicite:2]{index=2} :contentReference[oaicite:3]{index=3}

The few-shot demonstrations were drawn from the training data, fixed before evaluation, and kept unchanged during inference. No development or evaluation instances were used as demonstrations. :contentReference[oaicite:4]{index=4}

## Official Results

| Track | Setting | Favg2 | Rank |
|---|---|---:|---:|
| Track 1 | Seen Targets | **0.8420** | **8th** |
| Track 2 | Unseen Targets | **0.9192** | **3rd** |

These are the official StanceEval-2026 results of the final multi-LLM system. :contentReference[oaicite:5]{index=5}

## Task

StanceEval-2026 is an Arabic stance detection shared task based on the **Mawqif-XT** benchmark. For each instance, the system receives a target and an Arabic social-media post and must determine the stance expressed toward that specific target.

The required labels are:

- `Favor`
- `Against`
- `None`

The task is target-aware: the prediction should reflect the relationship between the post and the specified target rather than general sentiment. :contentReference[oaicite:6]{index=6}

## Track 1 — Seen Targets

Track 1 evaluates stance detection for targets seen during training. It covers three targets:

- COVID-19 Vaccine
- Digital Transformation
- Women Empowerment

The Track 1 system uses fixed few-shot demonstrations from the training data. GPT-5.5 Instant, Gemini 3.6 Flash with Extended Thinking, and Qwen 3.7 Plus independently generate stance predictions. These predictions are then examined by GPT-5.5 and incorporated into cross-model comparison for the final decision. :contentReference[oaicite:7]{index=7} :contentReference[oaicite:8]{index=8}

**Official Track 1 result:**

- **Favg2:** 0.8420
- **Rank:** 8th

The complete Track 1 materials are available in:

```text
SRK_AI_Lab_Track_1/

Track 2 — Unseen Targets

Track 2 evaluates cross-target generalization, where the evaluation targets are unseen during training. The system therefore relies on the semantic relationship between the target and the Arabic post rather than target-specific memorization.

The same three-model few-shot workflow is used for Track 2, followed by GPT-5.5 auditing and cross-model comparison.

Official Track 2 result:

Favg2: 0.9192
Rank: 3rd

The complete Track 2 materials are available in:

SRK_AI_Lab_Track_2/

## System

The prediction workflow consists of several stages.

```text
                         Target + Arabic Post
                                  |
                 +----------------+----------------+
                 |                |                |
                 v                v                v
          GPT-5.5 Instant   Gemini 3.6 Flash   Qwen 3.7 Plus
             (Few-shot)     with Extended       (Few-shot)
                              Thinking
                 |                |                |
                 v                v                v
            Prediction 1     Prediction 2     Prediction 3
          (Favor / Against / None)
                 |                |                |
                 +----------------+----------------+
                                  |
                                  v
                       GPT-5.5 Audit Analysis
                     (All Three Predictions)
                                  |
                                  v
                     Target–Post Examination
                  + Linguistic/Stance Evidence
                                  |
                                  v
                       Cross-Model Comparison
                  + Audit Observations
                  + Agreement/Disagreement
                                  |
                                  v
                          Final Decision
              Most-Supported Stance Label Selected
               (Gemini > GPT-5.5 > Qwen for
                unresolved ambiguity only)
                                  |
                                  v
                         Final Stance Label
                         Favor / Against / None
```

The system begins by providing each LLM with a few-shot prompt containing representative examples and task instructions.

GPT-5.5 Instant, Gemini 3.6 Flash with Extended Thinking, and Qwen 3.7 Plus independently interpret each Target–Post pair and produce a stance prediction.

The three predictions are then compared using their agreement and disagreement patterns.

GPT-5.5 performs an audit of all three model predictions by examining the original Target–Post pair and relevant linguistic and stance evidence.

The audit provides an additional model-based assessment of the predictions, considering factors such as target–post relation, negation, sarcasm, irony, rhetorical questions, and implicit meaning.

The final decision considers the three model predictions, GPT-5.5 audit observations, and the linguistic and contextual evidence in the Target–Post pair to select the most supported stance label.



## Target-Aware Interpretation

The system treats stance detection as a target-specific classification problem rather than ordinary sentiment classification.

During prediction and GPT-5.5 auditing, attention is given to factors such as explicit and implicit stance, negation, sarcasm, irony, contextual meaning, pragmatic interpretation, hashtags, wording, and cases where a post discusses a topic without expressing Favor or Against toward the specified target.

This target-aware interpretation is important for both seen-target classification in Track 1 and unseen-target generalization in Track 2.


## Few-Shot Prompting

Few-shot prompting is used to communicate the task definition, stance labels, decision guidelines, and representative examples to each prediction model.

The prediction models receive the relevant Target–Post input together with the corresponding few-shot task context. The few-shot demonstrations are drawn from the training data, fixed before evaluation, and kept unchanged during inference. No development or evaluation instances are used as demonstrations.

The prompt materials are preserved separately:

```text
Prompts/
├── Gemini/
├── GPT/
└── Qwen/


The corresponding prompt materials are available separately inside each track directory.

## LLM Prediction Sources

The three independent prediction sources are:

GPT-5.5 Instant
Gemini 3.6 Flash with Extended Thinking
Qwen 3.7 Plus

The models were accessed through their respective interactive chat interfaces.

The LLM prediction stage was not implemented as a unified API- or SDK-based automated inference pipeline.

The repository therefore preserves the available records of the interactive prediction process, including prompts, raw outputs, screenshots, audit materials, and post-processing resources.

## Cross-Model Comparison

After the three models independently produce their stance predictions, the predictions are examined together with the GPT-5.5 audit observations.

The comparison considers model agreement and disagreement patterns across the three predictions.

Agreement between models provides supporting evidence, but agreement is not treated as an automatic guarantee of correctness.

When predictions differ, the disagreement is examined using the GPT-5.5 audit observations and the original Target–Post evidence.

When all three models agree, the agreement is also checked against the original Target–Post evidence rather than being accepted automatically.

The cross-model comparison therefore considers both agreement and disagreement when determining the most supported stance label.

## GPT Audit

A dedicated GPT-5.5 audit workflow is used to examine the predictions generated by GPT-5.5 Instant, Gemini 3.6 Flash with Extended Thinking, and Qwen 3.7 Plus.

The audit is applied to all prediction cases, including both agreement and disagreement cases.

For each case, GPT-5.5 examines the original target and Arabic post together with the three model predictions and evaluates the target–post relationship and relevant linguistic or contextual evidence.

The audit pays particular attention to phenomena such as:

negation;
sarcasm;
irony;
rhetorical questions;
mockery;
implicit stance;
target/opponent relationships; and
cases where sentiment and stance may differ.

The GPT-5.5 audit is not treated as a fourth independent prediction. Its observations provide additional model-based evidence for the cross-model comparison and final decision.

The audit does not automatically override the other model predictions. The original Target–Post pair remains the primary evidence, while the model predictions and GPT-5.5 audit observations are considered together.

The audit materials are preserved in:

GPT_Audit/

for each track.


Final Decision

The final stance label is selected after considering:

GPT-5.5 prediction;
Gemini 3.6 Flash prediction;
Qwen 3.7 Plus prediction;
GPT-5.5 audit observations;
cross-model agreement/disagreement patterns; and
the linguistic and contextual evidence in the original Target–Post pair.

The final decision selects the most supported label among Favor, Against, and None.

For unresolved ambiguity, the fixed secondary priority is Gemini 3.6 Flash with Extended Thinking, followed by GPT-5.5 Instant and Qwen 3.7 Plus. This priority is used only as a secondary criterion and does not automatically override the available evidence.



LLM Access and Reproducibility

The GPT, Gemini, and Qwen predictions were generated through direct interaction with their respective chat interfaces.

No API- or SDK-based automated inference pipeline was used for the interactive LLM prediction stage.

Accordingly, the repository does not claim an API implementation that was not actually used.

Because interactive LLM responses may not be deterministically reproducible through the same interface at a later time, exact programmatic reproduction of every individual interactive response cannot be guaranteed.

The repository instead provides the available records of the original workflow, including prompts, raw outputs, screenshots, GPT-5.5 audit materials, post-processing notebooks, and final prediction files.

Where local notebooks or scripts were used for parsing, comparison, organization, post-processing, or preparation of final prediction files, those materials are included in the corresponding track directories.

Evidence Included

The repository preserves the available evidence for both tracks, including:

few-shot prompts;
raw GPT outputs;
raw Gemini outputs;
raw Qwen outputs;
screenshots of interactive LLM sessions;
GPT-5.5 audit prompts;
GPT-5.5 audit outputs;
post-processing notebooks;
final prediction files; and
submission archives.

The materials are organized independently for Track 1 and Track 2.

## Repository Structure

```text
SRK_AI_Lab_Team/
│
├── README.md
├── LICENSE
│
├── SRK_AI_Lab_Track_1/
│   │
│   ├── README.md
│   │
│   ├── Final_Submission/
│   │   ├── 880809_prediction.txt
│   │   ├── Final_Gemini_Few_Shot_09_Final.txt
│   │   └── Final_Gemini_Few_Shot_09_final.zip
│   │
│   ├── GPT_Audit/
│   │   ├── GPT_Audit_Track-1_Prompt.txt
│   │   ├── GPT_Audit_Track-1_Raw_Outputs.txt
│   │   └── LLM_Screenshots/
│   │
│   ├── LLM_Raw_Outputs/
│   │   ├── Gemini/
│   │   ├── GPT/
│   │   └── Qwen/
│   │
│   ├── LLM_Screenshots/
│   │   ├── Gemini/
│   │   ├── GPT/
│   │   └── Qwen/
│   │
│   ├── Parsing_Postprocessing/
│   │   └── Track-1_Post_Processing.ipynb
│   │
│   └── Prompts/
│       ├── Gemini/
│       ├── GPT/
│       └── Qwen/
│
└── SRK_AI_Lab_Track_2/
    │
    ├── README.md
    │
    ├── Final_Submission/
    │   ├── 880023_prediction.txt
    │   ├── Gemini_Few_Shot_17_labels_only.txt
    │   └── Gemini_Few_Shot_17_labels_only.zip
    │
    ├── GPT_Audit/
    │   ├── GPT_Audit_Track-2_Prompt.txt
    │   ├── GPT_Audit_Track-2_Raw_Outputs.txt
    │   └── LLM_Screenshots/
    │
    ├── LLM_Raw_Outputs/
    │   ├── Gemini/
    │   ├── GPT/
    │   └── Qwen/
    │
    ├── LLM_Screenshots/
    │   ├── Gemini/
    │   ├── GPT/
    │   └── Qwen/
    │
    ├── Parsing_Postprocessing/
    │   └── Track-2_Post_Processing.ipynb
    │
    └── Prompts/
        ├── Gemini/
        ├── GPT/
        └── Qwen/
```

## Track 1 Materials

The Track 1 directory contains all materials specific to the seen-target
submission.

### Final Submission

```text
SRK_AI_Lab_Track_1/Final_Submission/
├── 880809_prediction.txt
├── Final_Gemini_Few_Shot_09_Final.txt
└── Final_Gemini_Few_Shot_09_final.zip
```

These files contain the final Track 1 prediction and associated submission
materials.

### GPT Audit

```text
SRK_AI_Lab_Track_1/GPT_Audit/
├── GPT_Audit_Track-1_Prompt.txt
├── GPT_Audit_Track-1_Raw_Outputs.txt
└── LLM_Screenshots/
```

This directory contains the Track 1 GPT audit prompt, raw audit outputs, and
associated screenshots.

### Raw LLM Outputs

```text
SRK_AI_Lab_Track_1/LLM_Raw_Outputs/
├── Gemini/
├── GPT/
└── Qwen/
```

These directories contain the recorded outputs from the three prediction
sources.

### LLM Screenshots

```text
SRK_AI_Lab_Track_1/LLM_Screenshots/
├── Gemini/
├── GPT/
└── Qwen/
```

These directories contain screenshots associated with the interactive LLM
prediction process.

### Post-Processing

```text
SRK_AI_Lab_Track_1/Parsing_Postprocessing/
└── Track-1_Post_Processing.ipynb
```

This notebook contains the Track 1 parsing and post-processing operations
used in preparing the prediction outputs.

### Prompts

```text
SRK_AI_Lab_Track_1/Prompts/
├── Gemini/
├── GPT/
└── Qwen/
```

These directories contain the Track 1 few-shot prompt materials.

## Track 2 Materials

The Track 2 directory contains all materials specific to the unseen-target
submission.

### Final Submission

```text
SRK_AI_Lab_Track_2/Final_Submission/
├── 880023_prediction.txt
├── Gemini_Few_Shot_17_labels_only.txt
└── Gemini_Few_Shot_17_labels_only.zip
```

These files contain the final Track 2 prediction and associated submission
materials.

### GPT Audit

```text
SRK_AI_Lab_Track_2/GPT_Audit/
├── GPT_Audit_Track-2_Prompt.txt
├── GPT_Audit_Track-2_Raw_Outputs.txt
└── LLM_Screenshots/
```

This directory contains the Track 2 GPT audit prompt, raw audit outputs, and
associated screenshots.

### Raw LLM Outputs

```text
SRK_AI_Lab_Track_2/LLM_Raw_Outputs/
├── Gemini/
├── GPT/
└── Qwen/
```

These directories contain the recorded outputs from the three prediction
sources.

### LLM Screenshots

```text
SRK_AI_Lab_Track_2/LLM_Screenshots/
├── Gemini/
├── GPT/
└── Qwen/
```

These directories contain screenshots associated with the interactive LLM
prediction process.

### Post-Processing

```text
SRK_AI_Lab_Track_2/Parsing_Postprocessing/
└── Track-2_Post_Processing.ipynb
```

This notebook contains the Track 2 parsing and post-processing operations
used in preparing the prediction outputs.

### Prompts

```text
SRK_AI_Lab_Track_2/Prompts/
├── Gemini/
├── GPT/
└── Qwen/
```

These directories contain the Track 2 few-shot prompt materials.

## Final Submission Records

The repository preserves the final prediction files associated with both
submitted tracks.

For Track 1:

```text
SRK_AI_Lab_Track_1/Final_Submission/
```

For Track 2:

```text
SRK_AI_Lab_Track_2/Final_Submission/
```

The files in these directories correspond to the prediction/submission
materials prepared for the respective StanceEval-2026 tracks.

## Post-Processing Resources

The repository contains separate notebooks for the two tracks:

```text
Track 1:
SRK_AI_Lab_Track_1/Parsing_Postprocessing/Track-1_Post_Processing.ipynb

Track 2:
SRK_AI_Lab_Track_2/Parsing_Postprocessing/Track-2_Post_Processing.ipynb
```

These notebooks document the relevant parsing and post-processing performed
after the interactive LLM prediction stage.

## Dataset

The Mawqif-XT dataset is not redistributed in this repository.

This repository contains the system documentation and prediction-related
materials associated with the StanceEval-2026 task, but does not provide a
copy of the task dataset.

Users who wish to reproduce the work should obtain the dataset through its
official distribution channel and comply with the applicable dataset terms
and conditions.

The repository does not claim ownership of the Mawqif-v2 dataset.

## Third-Party Models and Services

The system uses third-party LLMs and services, including GPT, Gemini, and
Qwen.

This repository does not claim ownership of the underlying third-party
models, services, or technologies.

The recorded model outputs and screenshots are included as documentation and
evidence of the prediction workflow.

Use of third-party models and services remains subject to their respective
terms, policies, and licenses.

## Screenshots and Recorded Outputs

The repository contains screenshots and recorded outputs generated during the
interactive LLM prediction process.

These materials are included to document the system workflow and preserve the
available evidence associated with the submitted predictions.

They should not be interpreted as redistribution of the underlying
third-party model or service.

## License

This repository includes a root-level `LICENSE` file containing the **MIT License**.

The MIT License applies to original SRK AI Lab materials in this repository for which the team holds the relevant rights.

The MIT License does not override or replace the terms applicable to third-party materials, including:

- the Mawqif-XT dataset;
- third-party LLMs and services;
- third-party content; and
- other materials whose rights belong to their respective owners.

Users are responsible for complying with the applicable terms and conditions when using third-party materials.

## Acknowledgment

We thank the **StanceEval-2026 organizers** and **Mawqif-XT contributors**
for providing the shared-task framework, dataset, and evaluation resources.


