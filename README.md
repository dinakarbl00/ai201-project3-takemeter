# TakeMeter

TakeMeter is a text classifier for public professional basketball Reddit discourse. It classifies posts and comments into four labels: `analysis`, `hot_take`, `reaction`, and `low_effort_noise`.

The goal is not to decide whether a basketball opinion is right or wrong. The goal is to classify the style and quality of the take: whether it uses reasoning, makes an unsupported claim, reacts emotionally, or contributes very little substance.

## Community Choice

I chose public professional basketball Reddit discussion, mainly NBA related communities such as `r/nba` and `r/nbadiscussion`, with one WNBA post game thread included for additional post game discourse. This community is a good fit for TakeMeter because basketball Reddit has a wide range of discourse quality. Some comments use stats, matchups, coaching decisions, and roster context, while others are emotional reactions, bold unsupported claims, or meme like comments.

These distinctions matter because basketball fans often argue not only about teams and players, but also about the quality of the discourse itself. A regular participant can usually tell the difference between a detailed basketball argument, a hot take, a live reaction, and low effort noise.

## Label Taxonomy

### `analysis`

A post should be labeled `analysis` when it makes a clear basketball argument using specific reasoning, evidence, stats, tactical observations, historical comparison, or cause and effect explanation.

Examples:

1. "The Spurs sagged off LeBron and went under screens and dared him to shoot, and he made them pay in Game 7."
2. "Tony Parker averaged 15.7 points per game in the 2013 Finals on 41, 28, and 72 shooting splits."

Decision rule: If a post gives specific basketball reasoning, I label it `analysis` even if the tone is opinionated.

### `hot_take`

A post should be labeled `hot_take` when it makes a bold, confident, or controversial basketball claim without enough evidence or reasoning to support it.

Examples:

1. "The NBA has too many trophies."
2. "Prime LeBron is as good as any basketball player ever."

Decision rule: A post can include some basketball language and still be a `hot_take` if the claim is broad and not supported with enough evidence.

### `reaction`

A post should be labeled `reaction` when it mainly expresses an immediate emotional response to a game, play, trade, injury, or news event, without trying to make a broader argument.

Examples:

1. "Amazing game."
2. "That block by LeBron on Iggy at the end was unbelievable."

Decision rule: A short post is not automatically low effort. If it is clearly an emotional response to a game moment, I label it `reaction`.

### `low_effort_noise`

A post should be labeled `low_effort_noise` when it is too short, meme like, repetitive, sarcastic, or empty to contribute a meaningful basketball take.

Examples:

1. "The LLLarriors."
2. "No NVP tho."

Decision rule: I use this label when the comment is mostly a joke, chant, meme, or throwaway line instead of a meaningful take.

## Dataset

The final dataset contains 200 labeled examples in a single CSV file with three columns:

| Column  | Description                                                    |
| ------- | -------------------------------------------------------------- |
| `text`  | The cleaned Reddit post or comment text                        |
| `label` | One of the four taxonomy labels                                |
| `notes` | Optional annotation notes for difficult or borderline examples |

The data came from public Reddit basketball discussion spaces. Sources included `r/nba`, `r/nbadiscussion`, `r/lakers`, and one `r/wnba` post game thread. I collected comments from post game threads, player discussion threads, historical highlight threads, and broader discussion threads.

I cleaned the raw comments by removing usernames, vote counts, timestamps, Reddit interface text, and repeated reply markers. I also skipped comments that were only questions, deleted comments, pure factual explanations without a take, or comments that depended too much on missing context.

### Label Distribution

| Label            | Count |
| ---------------- | ----: |
| analysis         |    60 |
| hot_take         |    48 |
| reaction         |    52 |
| low_effort_noise |    40 |
| Total            |   200 |

No single label accounts for more than 70 percent of the dataset. The largest label, `analysis`, accounts for 30 percent of the data.

## Labeling Process

I labeled each example using the definitions in the taxonomy. I tried to keep the labels mutually exclusive by focusing on what the comment was mainly doing.

If a comment gave specific reasoning, stats, matchup details, or cause and effect explanation, I labeled it `analysis`. If it made a strong claim without enough support, I labeled it `hot_take`. If it mainly expressed emotion about a game or moment, I labeled it `reaction`. If it was mostly a meme, joke, chant, or throwaway line, I labeled it `low_effort_noise`.

For borderline examples, I added notes explaining the possible labels and why I chose the final label.

### Difficult Labeling Examples

#### Difficult Example 1

Text:
"Shooting Stars is a meme, but it's a great haul for the King of NY."

Possible labels: `reaction`, `hot_take`
Final label: `hot_take`

I chose `hot_take` because the comment makes an evaluative claim about which trophies matter and how impressive the haul is. It has a celebratory tone, but it is more of a judgment than a pure emotional reaction.

#### Difficult Example 2

Text:
"LeBron from 2012 to 2018 is the greatest basketball player in NBA history. A standard LeBron line in the playoffs could be something like 33, 9, and 8 while guarding 1 through 5. The perfect player."

Possible labels: `analysis`, `hot_take`
Final label: `analysis`

This was difficult because the opening sentence is a huge all time claim. I chose `analysis` because the comment gives specific support: playoff production, rebounding, assists, and defensive versatility across positions.

#### Difficult Example 3

Text:
"Amazing game."

Possible labels: `reaction`, `low_effort_noise`
Final label: `reaction`

This was difficult because the comment is extremely short. I chose `reaction` because it directly responds emotionally to the game result. If the same comment appeared without post game context, it might look like low effort noise.

#### Difficult Example 4

Text:
"He owes the team a discount for keeping his son around."

Possible labels: `hot_take`, `low_effort_noise`
Final label: `low_effort_noise`

This was difficult because it implies a take about LeBron's contract value, but the comment is mostly a joke or jab. I labeled it `low_effort_noise` because it does not explain a real basketball or roster argument.

## Fine Tuning Approach

The base model was `distilbert-base-uncased`, fine tuned for four class text classification. I used the provided Google Colab starter notebook with a T4 GPU runtime. The notebook handled the train, validation, and test split using a 70 percent, 15 percent, and 15 percent split.

The final split was:

| Split      | Count |
| ---------- | ----: |
| Train      |   140 |
| Validation |    30 |
| Test       |    30 |

I first tried the starter notebook defaults: 3 epochs, batch size 16, learning rate 2e-5, and 50 warmup steps. That run underfit the task. The training split only produced 27 total update steps, so 50 warmup steps meant the model spent the whole run warming up. It did not learn to predict `reaction` or `low_effort_noise`.

I changed the training setup to:

| Hyperparameter   | Final Value |
| ---------------- | ----------: |
| Epochs           |           8 |
| Train batch size |           8 |
| Eval batch size  |          32 |
| Learning rate    |        3e-5 |
| Warmup steps     |          14 |
| Weight decay     |        0.01 |

I chose these settings because the first run clearly underfit. The smaller batch size and higher epoch count gave the model more update steps, while 14 warmup steps kept the warmup around 10 percent of training instead of longer than the entire training run.

## Baseline Approach

The baseline model was Groq `llama-3.3-70b-versatile` used as a zero shot classifier. I used the same 30 example test set that was used for the fine tuned model.

The prompt included the four label definitions and decision rules. The model was instructed to output only one of the valid label names.

Prompt summary:

```text
You are classifying public professional basketball Reddit posts and comments.
Assign each post to exactly one label: analysis, hot_take, reaction, or low_effort_noise.

analysis: specific basketball reasoning, evidence, stats, tactics, historical comparison, or cause and effect.
hot_take: bold or controversial claim without enough evidence.
reaction: immediate emotional response to a game, play, trade, injury, or news event.
low_effort_noise: short, meme like, repetitive, or empty comment with little meaningful substance.

Choose exactly one label.
Output only the label name.
```

I collected the baseline predictions by running the Groq classification cell in the Colab notebook after the dataset was split. All 30 test examples produced parseable responses.


## Evaluation Report

### Test Set

The dataset had 200 total labeled examples. The starter notebook split the dataset into 140 training examples, 30 validation examples, and 30 test examples.

The test set label distribution was:

| Label            | Count |
| ---------------- | ----: |
| analysis         |     9 |
| hot_take         |     7 |
| reaction         |     8 |
| low_effort_noise |     6 |
| Total            |    30 |

### Baseline Model

The baseline was a zero shot Groq model using `llama-3.3-70b-versatile`. I gave the model the four label definitions from my taxonomy and asked it to output exactly one label name for each test example. I collected the baseline predictions on the same 30 example test set used for the fine tuned model.

Baseline accuracy:

| Model                   | Accuracy |
| ----------------------- | -------: |
| Zero shot Groq baseline |    0.633 |

Baseline per class metrics:

| Label            | Precision | Recall |   F1 | Support |
| ---------------- | --------: | -----: | ---: | ------: |
| analysis         |      0.89 |   0.89 | 0.89 |       9 |
| hot_take         |      0.67 |   0.29 | 0.40 |       7 |
| reaction         |      0.50 |   0.88 | 0.64 |       8 |
| low_effort_noise |      0.50 |   0.33 | 0.40 |       6 |

The zero shot baseline was strongest on `analysis`, likely because analysis posts contain explicit reasoning, stats, or cause and effect language. It struggled more with `hot_take` and `low_effort_noise`, where short, sarcastic, emotional, or meme like comments can overlap.

### Fine Tuned Model

The fine tuned model used `distilbert-base-uncased`.

Fine tuned accuracy:

| Model                 | Accuracy |
| --------------------- | -------: |
| Fine tuned DistilBERT |    0.533 |

Fine tuned per class metrics:

| Label            | Precision | Recall |   F1 | Support |
| ---------------- | --------: | -----: | ---: | ------: |
| analysis         |      0.78 |   0.78 | 0.78 |       9 |
| hot_take         |      0.50 |   0.29 | 0.36 |       7 |
| reaction         |      0.50 |   0.62 | 0.56 |       8 |
| low_effort_noise |      0.29 |   0.33 | 0.31 |       6 |

### Baseline vs Fine Tuned Comparison

| Model                   | Accuracy |
| ----------------------- | -------: |
| Zero shot Groq baseline |    0.633 |
| Fine tuned DistilBERT   |    0.533 |

The fine tuned DistilBERT model performed 0.100 below the Groq baseline. I do not interpret this as a failure of the project, but as evidence that this subjective discourse task is difficult for a small model trained on only 200 examples. Groq’s larger general language model had a stronger prior understanding of tone, sarcasm, and basketball discussion. The fine tuned model still learned useful distinctions, especially for `analysis`, but it struggled with the shorter and more ambiguous labels.

### Fine Tuned Confusion Matrix

| True Label       | Predicted analysis | Predicted hot_take | Predicted reaction | Predicted low_effort_noise |
| ---------------- | -----------------: | -----------------: | -----------------: | -------------------------: |
| analysis         |                  7 |                  2 |                  0 |                          0 |
| hot_take         |                  1 |                  2 |                  1 |                          3 |
| reaction         |                  1 |                  0 |                  5 |                          2 |
| low_effort_noise |                  0 |                  0 |                  4 |                          2 |

The strongest class was `analysis`, where the model correctly predicted 7 out of 9 examples. The weakest class was `low_effort_noise`, where it only predicted 2 out of 6 examples correctly. The biggest confusion pattern was `low_effort_noise` being predicted as `reaction`, which happened 4 times. The model also confused `hot_take` with `low_effort_noise` 3 times.

### Specific Wrong Predictions

#### Wrong Prediction 1

Text:
"The reason people are being so much more harsh to SAS than OKC is because SAS lost more recently. People already had these conversations about OKC, but moved on because the Finals started."

True label: `analysis`
Predicted label: `hot_take`
Confidence: 0.57

This example was labeled `analysis` because it explains a discourse pattern using timing and media attention. The model likely predicted `hot_take` because the wording is about people being harsh and narratives moving quickly, which sounds like opinionated sports discourse. The mistake shows that the model sometimes focuses on tone instead of recognizing causal explanation.

#### Wrong Prediction 2

Text:
"Amazing game."

True label: `reaction`
Predicted label: `low_effort_noise`
Confidence: 0.64

This is a difficult boundary case. The comment is short, but it is still a direct emotional response to the game, so I labeled it `reaction`. The model predicted `low_effort_noise` because the text is only two words and has no specific basketball detail. This shows a weakness in the label boundary: short reactions and low effort comments are hard to separate without thread context.

#### Wrong Prediction 3

Text:
"The Thunder had bad breaks with injuries, that is why they lost. The Spurs just blew a golden opportunity."

True label: `hot_take`
Predicted label: `analysis`
Confidence: 0.93

This was labeled `hot_take` because it makes a strong comparative claim about why two teams lost, but does not provide enough evidence to fully support it. The model predicted `analysis` with high confidence, probably because the sentence contains cause and effect language like "that is why they lost." This shows that the model learned to associate explanatory wording with `analysis`, even when the explanation is too broad or unsupported.

#### Wrong Prediction 4

Text:
"He owes the team a discount for keeping his son around."

True label: `low_effort_noise`
Predicted label: `reaction`
Confidence: 0.38

This was labeled `low_effort_noise` because it is more of a joke or jab than a meaningful basketball take. The model predicted `reaction`, probably because it recognized negative sentiment but did not understand the meme like context. The low confidence suggests the model was uncertain, which makes sense for a short sarcastic comment.

### Error Pattern Analysis

The main failure pattern is that the fine tuned model struggles with short posts where the difference between `reaction`, `hot_take`, and `low_effort_noise` depends on context and tone. For longer posts, especially `analysis`, the model can use explicit signals such as stats, matchups, injuries, rotations, and cause and effect language. For shorter posts, the model often has to infer whether a comment is a real emotional response, a bold unsupported claim, or just a joke.

This pattern appears in several mistakes. "Amazing game" was predicted as `low_effort_noise` even though it was a reaction. "Should've been the DPOY over Gasol" was predicted as `low_effort_noise` even though it was a hot take. "As a Cavs fan since 17 days ago, you have no idea how much this means to me" was predicted as `reaction` even though it was labeled `low_effort_noise` because it was a joke about bandwagon fandom.

To improve this model, I would add more short examples with clearer notes around the boundary between `reaction` and `low_effort_noise`. I would also add more examples of short `hot_take` comments so the model can learn that a short comment can still be a claim if it makes a strong judgment.

### What the Model Captured vs What I Intended

I intended the model to learn discourse quality based on how a post makes its point: evidence based reasoning, unsupported claim, emotional response, or low effort noise. The model partially captured this. It learned that longer posts with stats, matchup details, or explanatory structure usually belong to `analysis`.

However, the model also learned shortcuts. It often treated cause and effect language as `analysis`, even when the claim was unsupported. It also treated very short text as either `reaction` or `low_effort_noise` without reliably understanding the difference. This means the model learned surface level signals like length, emotional wording, and explanatory phrases, but did not fully capture the intended boundary between meaningful reaction, unsupported take, and empty joke.

### Sample Classifications

| Text                                                                                                                                                  | True Label       | Predicted Label  | Confidence | Correct? |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ---------------- | ---------: | -------- |
| "Tony Parker averaged 15.7 points per game in the 2013 Finals on 41, 28, and 72 shooting splits."                                                     | analysis         | analysis         |       0.87 | Yes      |
| "I was so shocked when I saw they were up in overtime."                                                                                               | reaction         | reaction         |       0.80 | Yes      |
| "The Fever reliably find a way to lose after a great beginning. They make it infuriating to remain a fan. Congrats to the Dream for wanting it more." | hot_take         | hot_take         |       0.67 | Yes      |
| "Amazing game."                                                                                                                                       | reaction         | low_effort_noise |       0.64 | No       |
| "He owes the team a discount for keeping his son around."                                                                                             | low_effort_noise | reaction         |       0.38 | No       |

The first example is a reasonable correct prediction because it includes a specific stat line and shooting splits, which matches the `analysis` definition. The model correctly recognized that this post was using evidence rather than only expressing emotion.

The incorrect examples show the main weakness of the model. "Amazing game" is a reaction, but it is so short that the model treated it like low effort noise. "He owes the team a discount for keeping his son around" is a joke or jab, but the model treated the negative sentiment as a reaction.

### Confidence Calibration

I also checked whether the model's confidence scores were meaningful.

| Confidence Range | Count | Accuracy | Average Confidence |
| ---------------- | ----: | -------: | -----------------: |
| 0.00 to 0.40     |     3 |     0.00 |               0.38 |
| 0.40 to 0.60     |    10 |     0.30 |               0.50 |
| 0.60 to 0.80     |     8 |     0.75 |               0.68 |
| 0.80 to 1.00     |     9 |     0.78 |               0.89 |

The confidence scores were somewhat meaningful because higher confidence predictions were generally more accurate. Predictions between 0.60 and 1.00 were much more accurate than predictions below 0.60. However, the model was not perfectly calibrated. One wrong prediction had 0.93 confidence, which shows that the model can still be very confident when it learns the wrong shortcut.


## AI Usage

I used AI tools during the project, but I reviewed and revised the outputs myself.

First, I used ChatGPT to stress test the label taxonomy. I asked it to help identify ambiguous cases between labels such as `analysis` vs `hot_take` and `reaction` vs `low_effort_noise`. I revised the decision rules based on this, especially the rule that a short comment can still be a `reaction` if it clearly responds emotionally to a game moment.

Second, I used ChatGPT to help clean raw Reddit comments into CSV rows and suggest initial labels. I reviewed the rows before keeping them, removed comments that were not useful for the task, and used notes for difficult cases. This means the final dataset was AI assisted, but the labels were still reviewed against my taxonomy.

Third, I used ChatGPT to help analyze the model's wrong predictions. I gave it the confusion matrix and wrong prediction list, then used the output to identify a pattern: the fine tuned model struggled most with short posts where `reaction`, `hot_take`, and `low_effort_noise` overlap. I verified this by rereading the actual wrong predictions before writing the final analysis.

## Spec Reflection

One way the spec helped guide the work was by forcing the label taxonomy to be precise before data collection. The requirement for complete sentence definitions, examples, and hard edge cases made me think carefully about what each label meant. This helped avoid vague labels like good or bad, which would have been hard to annotate consistently.

One way my implementation diverged from the original plan was that I included one WNBA post game thread in addition to NBA sources. I originally planned to use only NBA communities, but I needed more post game reaction and low effort examples to balance the dataset. Since the task is about public professional basketball discourse, I kept the example but described the community honestly in the README.

Another implementation change was the fine tuning setup. I initially used the starter notebook defaults, but the first fine tuned model underfit and did not predict two of the four labels at all. I changed the number of epochs, batch size, learning rate, and warmup steps after seeing that the default setup was not learning the full task.

## Files in This Repository

| File                                      | Purpose                                                                                           |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `planning.md`                             | Initial project plan, label taxonomy, edge cases, metrics, AI tool plan, and stretch feature plan |
| `data/takemeter_labeled.csv`              | Final labeled dataset with 200 examples                                                           |
| `outputs/evaluation_results.json`         | Exported metric summary from Colab                                                                |
| `outputs/confusion_matrix.png`            | Confusion matrix image from the fine tuned model                                                  |
| `outputs/fine_tuned_test_predictions.csv` | Fine tuned model predictions on the test set                                                      |
| `outputs/wrong_predictions.csv`           | Wrong predictions used for error analysis                                                         |
| `README.md`                               | Final project report                                                                              |
