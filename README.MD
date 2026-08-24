# SRK AI Lab — StanceEval-2026

## Multi-LLM Few-Shot Prompting with Evidence-Guided Auditing for Arabic Stance Detection

**Team:** SRK AI Lab  
**Shared Task:** StanceEval-2026 — ArabicNLP 2026 Shared Task on Stance Detection  
**Tracks:** Track 1 — Seen Targets and Track 2 — Unseen Targets

This repository contains the prediction systems, prompts, raw LLM outputs, screenshots, GPT-based audit materials, post-processing notebooks, and final submission files developed by **SRK AI Lab** for the StanceEval-2026 shared task.

The submitted systems use a few-shot multi-LLM framework for Arabic stance detection. Three independent large language models — **GPT-5.5 Instant, Gemini 3.6 Flash with Extended Thinking, and Qwen 3.7 Plus** — were used to classify Target–Tweet pairs into the three task labels: `Favor`, `Against`, and `None`.

The prediction workflow combines independent LLM predictions with cross-model comparison, GPT-based auditing for difficult or disagreement cases, and evidence-guided adjudication based on the Target–Tweet relationship.

The system achieved the following official results:

| Track | Setting | Favg2 | Rank |
|---|---|---:|---:|
| Track 1 | Seen Targets | **0.8863** | **4th** |
| Track 2 | Unseen Targets | **0.9518** | **3rd** |

## Task

StanceEval-2026 is an Arabic stance detection shared task based on the Mawqif-v2 benchmark.

For each instance, the system receives a target and an Arabic tweet/post and must determine the stance expressed toward that specific target.

The required labels are:

- `Favor`
- `Against`
- `None`

The task is target-aware: a text should not be classified simply according to its general sentiment. The prediction must reflect whether the text expresses support, opposition, or no clear stance toward the specified target.

## Track 1 — Seen Targets

Track 1 evaluates stance detection for seen targets.

The SRK AI Lab Track 1 system uses few-shot examples together with the target and Arabic tweet to guide the three LLMs in making independent stance predictions.

The final Track 1 system combines the three model predictions with additional GPT-based auditing and evidence-guided adjudication.

**Official Track 1 result:**

- **Favg2:** 0.8863
- **Rank:** 4th

The complete Track 1 materials are available in:

```text
SRK_AI_Lab_Track_1/
````

## Track 2 — Unseen Targets

Track 2 evaluates stance detection for unseen targets.

The same general multi-LLM framework is used, but the prediction requires greater reliance on the semantic relationship between the target and tweet because the target is unseen relative to the relevant training setting.

The system therefore focuses on interpreting the Target–Tweet relationship rather than relying on target-specific memorization.

**Official Track 2 result:**

- **Favg2:** 0.9518
- **Rank:** 3rd

The complete Track 2 materials are available in:

```text
SRK_AI_Lab_Track_2/
```

## System

The prediction workflow consists of several stages.

```text
                         Target + Arabic Post
                                  |
                 +----------------+----------------+
                 |                |                |
                 v                v                v
          GPT-5.5 Instant   Gemini 3.6 Flash   Qwen 3.7 Plus
             Few-Shot       Extended Thinking      Few-Shot
                 |                |                |
                 +----------------+----------------+
                                  |
                                  v
                         Model Predictions
                                  |
                                  v
                       Cross-Model Comparison
                                  |
                    +-------------+-------------+
                    |                           |
                 Agreement              Disagreement /
                    |                    Difficult Case
                    |                           |
                    +-------------+-------------+
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

The system begins by providing each LLM with a few-shot prompt containing representative examples and the task instructions.

GPT, Gemini, and Qwen independently interpret each Target–Tweet pair and produce a stance prediction.

The resulting predictions are then compared. Agreement between models provides supporting evidence, while disagreement or difficult cases receive additional review.

A GPT-based audit is used to examine difficult cases and provide additional reasoning about the Target–Tweet relationship.

The audit is used as a quality-control and adjudication aid rather than as an unconditional source of ground truth.

The final decision considers the available model predictions, GPT audit analysis, and the linguistic and contextual evidence in the Target–Tweet pair.

## Target-Aware Interpretation

The system treats stance detection as a target-specific classification problem rather than ordinary sentiment classification.

During prediction and review, attention is given to factors such as explicit and implicit stance, negation, sarcasm, irony, contextual meaning, pragmatic interpretation, hashtags, wording, and cases where a tweet discusses a topic without actually expressing Favor or Against toward the specified target.

This target-aware interpretation is especially important for difficult examples and for Track 2 unseen-target generalization.

## Few-Shot Prompting

Few-shot prompting is used to communicate the task format, stance labels, and representative examples to each LLM.

The prediction models receive the relevant Target–Tweet input together with the few-shot task context.

The prompts used for the three models are preserved separately:

```text
Prompts/
├── Gemini/
├── GPT/
└── Qwen/
```

The corresponding prompt materials are available separately inside each track directory.

## LLM Prediction Sources

The three independent prediction sources are:

- **GPT-5.5 Instant**
- **Gemini 3.6 Flash with Extended Thinking**
- **Qwen 3.7 Plus**

The models were accessed through their respective interactive chat interfaces.

The LLM prediction stage was not implemented as a unified API-based automated inference pipeline.

The repository therefore preserves the actual evidence available from the interactive prediction process, including prompts, raw outputs, screenshots, audit records, and post-processing resources.

## Cross-Model Comparison

After the three models produce their predictions, the outputs are compared.

When all or multiple models agree, their agreement provides supporting evidence for the prediction.

Agreement is not treated as an automatic guarantee of correctness.

When predictions disagree, or when an example appears linguistically or contextually difficult, the case receives additional review.

The comparison process helps identify cases where different models interpret the Target–Tweet relationship differently.

## GPT Audit

A dedicated GPT-based audit workflow is included for difficult and disagreement cases.

The audit considers the relevant target, tweet, model predictions, and available evidence and provides an additional analysis of the stance relationship.

The audit is intended to support quality control and evidence-based decision making.

It is not treated as infallible and is not used through a rule such as:

```text
Always follow the GPT Audit.
```

Instead, the audit can support a prediction, identify a potentially problematic prediction, or provide additional context for the final decision.

The audit materials are preserved in:

```text
GPT_Audit/
```

for each track.

## Evidence-Guided Adjudication

The final prediction is determined after considering the available evidence from:

- GPT;
- Gemini;
- Qwen;
- cross-model comparison;
- GPT-based auditing; and
- direct examination of the Target–Tweet relationship.

The final label is therefore not claimed to be the unmodified output of any single LLM.

The purpose of the adjudication stage is to select the strongest Target–Tweet-supported interpretation using the available model and audit evidence.

## LLM Access and Reproducibility

The GPT, Gemini, and Qwen predictions were generated through direct interaction with their respective chat interfaces.

No API-based automated querying pipeline was used for the interactive LLM prediction stage.

Accordingly, the repository does not claim an API implementation that was not actually used.

Because interactive LLM responses may not be deterministically reproducible through the same interface at a later time, exact programmatic reproduction of every individual interactive response cannot be guaranteed.

The repository instead provides the available evidence and records of the original workflow, including the prompts, raw outputs, screenshots, audit materials, post-processing notebooks, and final prediction files.

Where local notebooks or scripts were used for parsing, comparison, organization, post-processing, or preparing final prediction files, those materials are included in the corresponding track directories.

## Evidence Included

The repository preserves the available evidence for both tracks, including:

- few-shot prompts;
- raw GPT outputs;
- raw Gemini outputs;
- raw Qwen outputs;
- screenshots of interactive LLM sessions;
- GPT audit prompts;
- GPT audit outputs;
- post-processing notebooks;
- final prediction files; and
- submission archives.

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

The Mawqif-v2 dataset is not redistributed in this repository.

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

- the Mawqif-v2 dataset;
- third-party LLMs and services;
- third-party content; and
- other materials whose rights belong to their respective owners.

Users are responsible for complying with the applicable terms and conditions when using third-party materials.

## Acknowledgment

We thank the **StanceEval-2026 organizers** and **Mawqif-v2 contributors**
for providing the shared-task framework, dataset, and evaluation resources.


