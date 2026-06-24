# TakeMeter: r/nba Discourse Classifier

## Project Overview

TakeMeter is a fine-tuned text classification project that evaluates the type of discourse in public r/nba comments. The goal is to classify each comment into one of three labels: `analysis`, `hot_take`, or `reaction`.

I chose r/nba because NBA discussion naturally includes a wide range of comment styles. Some users write detailed basketball analysis with reasoning or statistics, some post bold unsupported opinions, and others react emotionally to games, highlights, trades, and draft events.

This project compares a fine-tuned `distilbert-base-uncased` classifier against a zero-shot Groq `llama-3.3-70b-versatile` baseline on the same test set.

## Label Taxonomy

| Label | Definition |
|---|---|
| `analysis` | A basketball-related comment that makes an argument using reasoning, evidence, statistics, tactics, roster construction, player comparison, or historical context. |
| `hot_take` | A bold or confident basketball opinion with little or no real support. The claim may be interesting or even true, but the comment asserts more than it explains. |
| `reaction` | An emotional, humorous, hype, frustration, meme-like, sarcastic, or immediate response with little or no argument. |

### Label Examples

#### analysis

- “Jokic creates open looks because defenses have to help when he catches the ball near the elbow.”
- “The issue is not only minutes played, it is travel, recovery time, sleep schedule, and intensity of movement.”

#### hot_take

- “This team is cooked and should blow it up immediately.”
- “This player is never going to be a real first option on a championship team.”

#### reaction

- “NO WAY HE HIT THAT.”
- “This game is absolute chaos lmao.”

## Dataset

The dataset contains 200 public r/nba comments collected from multiple thread types, including game threads, highlight threads, player comparison discussions, AMA threads, and league-policy discussion threads.

The final dataset was saved as `takemeter_dataset.csv` with the following columns:

| Column | Description |
|---|---|
| `text` | The r/nba comment text |
| `label` | One of `analysis`, `hot_take`, or `reaction` |
| `notes` | Short annotation note explaining the label |
| `source_thread` | The Reddit thread where the comment came from |

### Label Distribution

| Label | Count | Percentage |
|---|---:|---:|
| `reaction` | 75 | 37.5% |
| `analysis` | 71 | 35.5% |
| `hot_take` | 54 | 27.0% |
| **Total** | **200** | **100%** |

The dataset is not perfectly balanced, but no single label dominates the dataset. The largest class is `reaction`, which represents 37.5% of the examples.

## Data Collection and Annotation Process

I collected public r/nba comments from several types of threads to capture different discourse styles:

- Game and draft threads for live reactions
- Highlight threads for emotional and hype responses
- Player-comparison threads for analysis and hot takes
- League-policy discussion threads for more detailed reasoning

Each example was manually reviewed and assigned exactly one label. I used the label definitions from `planning.md` while annotating. For difficult examples, I used decision rules based on whether the comment actually explained a claim or simply asserted it.

## Difficult Annotation Examples

| Comment | Possible Labels | Final Label | Reason |
|---|---|---|---|
| “Giannis is the much better 2 way player.” | `analysis`, `hot_take` | `hot_take` | It makes a basketball claim, but it does not explain why or provide evidence. |
| “Sir, what? Ray Allen hit a 3 down 3 with 5 seconds remaining, down 3-2 in the series lol.” | `reaction`, `analysis` | `analysis` | Even though the tone is casual, it uses a historical comparison to challenge a claim. |
| “The report doesn’t insist that load management doesn’t work. There needs to be a balance between rest and recovery.” | `analysis`, `hot_take` | `analysis` | It clarifies the study’s meaning and gives a more nuanced interpretation. |

## Model and Training Approach

The fine-tuned model started from `distilbert-base-uncased`, a pre-trained transformer model from Hugging Face.

The dataset was split into:

| Split | Examples |
|---|---:|
| Train | 140 |
| Validation | 30 |
| Test | 30 |

The model was fine-tuned using the starter CodePath Colab notebook. I kept the default training setup because the dataset was small and the goal was to evaluate whether a lightweight fine-tuned classifier could learn this subjective classification task.

Key hyperparameters:

| Hyperparameter | Value |
|---|---:|
| Base model | `distilbert-base-uncased` |
| Epochs | 3 |
| Learning rate | 2e-5 |
| Batch size | 16 |

## Zero-Shot Baseline

For the baseline, I used Groq `llama-3.3-70b-versatile` in a zero-shot setting. The model was given the label definitions and asked to classify each test example into exactly one of the three labels.

The baseline was evaluated on the same 30-example test set as the fine-tuned model.

Baseline prompt rules included:

- Output exactly one label
- Valid labels are `analysis`, `hot_take`, and `reaction`
- Choose `analysis` only when reasoning clearly supports the claim
- Choose `reaction` for jokes, hype, sarcasm, or emotional responses
- Choose `hot_take` for strong claims with little support

## Evaluation Results

### Fine-Tuned Model Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True Label | Predicted `analysis` | Predicted `hot_take` | Predicted `reaction` |
|---|---:|---:|---:|
| `analysis` | 11 | 0 | 0 |
| `hot_take` | 8 | 0 | 0 |
| `reaction` | 9 | 0 | 2 |

The confusion matrix shows that the fine-tuned model heavily over-predicted `analysis`. It correctly classified all 11 true `analysis` examples, but it misclassified all 8 `hot_take` examples as `analysis`. It also misclassified 9 of 11 `reaction` examples as `analysis`.

This suggests the model learned to associate NBA-related vocabulary with `analysis`, even when the comment was actually an unsupported opinion or emotional reaction. The model did not learn a strong boundary between a reasoned basketball argument and a surface-level basketball claim.

### Wrong Prediction Analysis

The fine-tuned model made 17 wrong predictions out of 30 test examples. Most of the errors followed the same pattern: the model predicted `analysis` for comments that were actually `hot_take` or `reaction`.

| Example | True Label | Predicted Label | Why It Failed |
|---|---|---|---|
| “Last years Rockets with Curry and no Harden might not even make the playoffs” | `hot_take` | `analysis` | The model likely treated the mention of specific teams and players as analytical. However, the comment is a hypothetical claim with no explanation or evidence, so it fits `hot_take`. |
| “Dang wtf that was such a short interview and for some commercials lmfao, god I hate espn” | `reaction` | `analysis` | The model may have focused on the criticism of broadcast coverage, but the comment is mostly emotional frustration and live-event reaction. |
| “Yet he bleeds points on the other end so those 4 extra assist is a wash.” | `hot_take` | `analysis` | This sounds basketball-specific because it mentions defense and assists, but it does not explain or support the claim enough to count as analysis. |
| “Mike Brown drew up some shit not seen since Picasso on that play” | `reaction` | `analysis` | The phrase refers to a play design, which may have pushed the model toward `analysis`, but the actual comment is a joke/reaction. |
| “Rob Pelinka doesn't know the salary cap.” | `hot_take` | `analysis` | The model likely associated salary-cap vocabulary with analysis, but the comment is a blunt unsupported claim. |

The main error pattern is that the model relied too much on surface-level NBA vocabulary. Mentions of teams, players, coaches, assists, defense, or salary-cap terms often caused the model to predict `analysis`, even when the comment did not provide real reasoning. This shows that the model did not fully learn the distinction between “basketball-related language” and “reasoned basketball argument.”

### Sample Classifications

| Comment | Predicted Label | Confidence | Notes |
|---|---|---:|---|
| “Giannis averaged 30/12/6 on 60 fg 62.5 TS” | `analysis` | 0.37 | This prediction is reasonable because the comment uses specific statistics to compare a player. |
| “Last years Rockets with Curry and no Harden might not even make the playoffs” | `analysis` | 0.37 | This prediction was wrong. The comment mentions teams and players, but it is still an unsupported hypothetical claim, so the true label is `hot_take`. |
| “Dang wtf that was such a short interview and for some commercials lmfao, god I hate espn” | `analysis` | 0.37 | This prediction was wrong. The comment is mainly emotional frustration, so the true label is `reaction`. |
| “Yet he bleeds points on the other end so those 4 extra assist is a wash.” | `analysis` | 0.39 | This prediction was wrong. It uses basketball language, but the claim is not explained enough to be `analysis`. |


### Overall Accuracy

| Model | Accuracy |
|---|---:|
| Groq zero-shot baseline | 0.767 |
| Fine-tuned DistilBERT | 0.433 |

The zero-shot Groq baseline performed better than the fine-tuned DistilBERT model on the test set.

### Baseline Per-Class Metrics

| Label | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| `analysis` | 0.89 | 0.73 | 0.80 | 11 |
| `hot_take` | 0.62 | 0.62 | 0.62 | 8 |
| `reaction` | 0.77 | 0.91 | 0.83 | 11 |
| **Accuracy** |  |  | **0.77** | 30 |
| **Macro avg** | 0.76 | 0.75 | 0.75 | 30 |
| **Weighted avg** | 0.77 | 0.77 | 0.77 | 30 |

### Fine-Tuned DistilBERT Per-Class Metrics

| Label | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| `analysis` | 0.39 | 1.00 | 0.56 | 11 |
| `hot_take` | 0.00 | 0.00 | 0.00 | 8 |
| `reaction` | 1.00 | 0.18 | 0.31 | 11 |
| **Accuracy** |  |  | **0.43** | 30 |
| **Macro avg** | 0.46 | 0.39 | 0.29 | 30 |
| **Weighted avg** | 0.51 | 0.43 | 0.32 | 30 |

## Initial Reflection

The fine-tuned model did not outperform the zero-shot baseline. Instead, it over-predicted `analysis`, correctly catching all true `analysis` examples but failing completely on `hot_take`. This suggests that the model learned surface-level basketball language as a signal for analysis, rather than learning the deeper boundary between a supported argument and an unsupported claim.

This is an important failure mode because `analysis` and `hot_take` often use similar NBA vocabulary. A short unsupported claim like “Giannis is the much better two-way player” contains basketball reasoning words, but it does not actually explain the claim. The model likely needed more training examples that explicitly contrast unsupported basketball claims with genuinely reasoned comments.


## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the difference between three types of r/nba discourse: reasoned basketball analysis, unsupported hot takes, and emotional reactions. The ideal model would not simply look for basketball vocabulary, but would judge whether the comment actually supports its claim.

The fine-tuned model did not fully learn that distinction. It seemed to learn that comments mentioning players, teams, coaches, stats, assists, defense, or salary-cap concepts were usually `analysis`. Because of that, it over-predicted `analysis` and failed to identify many `hot_take` and `reaction` examples.

The biggest gap was between `analysis` and `hot_take`. A comment like “Rob Pelinka doesn't know the salary cap” contains basketball/front-office vocabulary, but it is still an unsupported claim. The model treated that type of language as analytical even when there was no reasoning behind it.

If I improved this project, I would collect more borderline examples that contrast unsupported basketball claims with genuinely reasoned comments. I would also collect more short `analysis` examples and more basketball-specific `hot_take` examples so the model learns that topic-specific vocabulary alone is not enough.

## Spec Reflection

The planning spec helped guide the project by forcing me to define the labels before collecting the full dataset. This made annotation more consistent because I had a clear rule: `analysis` required actual reasoning, while `hot_take` was a strong claim without enough support.

One way the implementation diverged from the spec is that the fine-tuned model did not reach the original success goal of around 70% accuracy. Instead, the zero-shot Groq baseline performed much better than the fine-tuned DistilBERT model. I kept the result because it honestly revealed an important limitation: with only 200 examples, a small fine-tuned model struggled to learn a subjective discourse boundary that a larger instruction-following model handled better.

## AI Usage

I used AI assistance in several parts of this project.

First, I used AI to help stress-test and refine the label definitions. The AI helped generate and discuss borderline cases, especially examples that could be confused between `analysis` and `hot_take`. I revised the rules so that `analysis` required actual reasoning or evidence, not just basketball vocabulary.

Second, I used AI to help organize and label public r/nba examples into the dataset format. I reviewed the labels and kept the annotation rules consistent with my planning document. The labels were not treated as automatic ground truth; I checked them against the definitions while building the dataset.

Third, I used AI to help analyze the model’s wrong predictions. After the fine-tuned model over-predicted `analysis`, I used the wrong-prediction list to identify the main pattern: the model was relying too heavily on surface-level NBA vocabulary instead of detecting whether a comment actually explained a claim.
