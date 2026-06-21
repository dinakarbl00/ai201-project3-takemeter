# TakeMeter Planning

## Community Choice

I chose NBA discussion posts and comments from public basketball communities such as r/nba, r/nbadiscussion, and team subreddits. This community is a good fit for TakeMeter because basketball discourse has a wide range of take quality. Some posts make detailed arguments using stats, matchups, or tactical observations, while others are bold unsupported opinions, emotional reactions, or low effort comments.

This makes the community useful for a classification task because the labels reflect real distinctions that basketball fans already care about. A regular participant would understand the difference between a detailed basketball argument, a hot take, a live reaction, and a low effort comment.

## Initial Label Taxonomy

### analysis

A post should be labeled `analysis` when it makes a clear basketball argument using specific reasoning, evidence, stats, tactical observations, historical comparison, or cause and effect explanation.

Examples:
1. "The Nuggets struggled late because Minnesota kept putting two defenders near Jokic at the elbow and forced the weak side shooters to create off the dribble."
2. "SGA’s efficiency is not just free throws. He gets to his spots in the midrange, keeps turnovers low, and creates clean corner looks when the defense collapses."

### hot_take

A post should be labeled `hot_take` when it makes a bold, confident, or controversial basketball claim without enough evidence or reasoning to support it.

Examples:
1. "Tatum is never winning as the best player on a championship team."
2. "The Suns are cooked. That team has no real future."

### reaction

A post should be labeled `reaction` when it mainly expresses an immediate emotional response to a game, play, trade, injury, or news event, without trying to make a broader argument.

Examples:
1. "That block was insane. I still cannot believe he got there in time."
2. "I am sick. We really blew a 15 point lead in the fourth quarter."

### low_effort_noise

A post should be labeled `low_effort_noise` when it is too short, meme like, repetitive, or empty to contribute a meaningful basketball take.

Examples:
1. "L team lol"
2. "Refs sold."

## Hardest Anticipated Edge Case

The hardest edge case will be posts that sound like hot takes but include one statistic or one piece of basketball evidence.

Example:
"Embiid is a playoff fraud. His scoring always drops when the second round gets physical."

This could be interpreted as `hot_take` because it makes a bold negative claim with aggressive framing. It could also be interpreted as `analysis` because it mentions a basketball pattern. My decision rule is that one vague or cherry picked point is not enough for `analysis`. I will label it `hot_take` unless the post gives enough specific evidence, such as multiple examples, stats, matchups, or a clear explanation of why the pattern happens.

## Data Collection Plan

I will collect public NBA related posts and comments from Reddit communities such as r/nba, r/nbadiscussion, and possibly team specific subreddits if I need more examples for a label. I will only use public text that is visible without joining a private community or using private messages.

My goal is to collect at least 200 total examples. I will aim for roughly balanced labels so the model does not learn to predict one category too often. My target distribution is:

| Label            | Target Count |
| ---------------- | -----------: |
| analysis         |           50 |
| hot_take         |           50 |
| reaction         |           50 |
| low_effort_noise |           50 |

I will save the dataset as a single CSV file with at least these columns:

| Column | Purpose                                             |
| ------ | --------------------------------------------------- |
| text   | The Reddit post or comment text                     |
| label  | One of the four labels                              |
| notes  | Optional notes for difficult or borderline examples |

I will not split the dataset manually because the starter Colab notebook handles the train, validation, and test split automatically. The notebook uses a 70 percent training, 15 percent validation, and 15 percent test split.

Before labeling all 200 examples, I will review around 30 to 40 real examples to check whether the labels apply cleanly. If one label is too rare or too broad, I will adjust the definitions before finishing the full annotation.

If one label is underrepresented after 200 examples, I will collect more examples for that label instead of accepting a heavily imbalanced dataset. I will also make sure no single label is more than 70 percent of the dataset.

## Evaluation Metrics

I will evaluate both the fine tuned DistilBERT model and the Groq zero shot baseline on the same test set. This matters because the comparison is only fair if both models are tested on the exact same examples.

I will report overall accuracy because it gives a simple view of how often the model is correct. However, accuracy alone is not enough for this project because the labels are subjective and some labels may be harder than others.

I will also report per class precision, recall, and F1 score. These metrics help show whether the model is learning each label or only doing well on the easier categories.

Precision will help answer: when the model predicts a label, how often is it right?

Recall will help answer: out of all true examples of a label, how many did the model catch?

F1 score will help balance precision and recall for each label. I expect F1 to be especially useful for labels like `analysis` and `hot_take`, because those two may be confused when a post includes both strong opinion and some evidence.

I will also include a confusion matrix for the fine tuned model. This will show which label pairs the model confuses most often. For example, if many true `analysis` examples are predicted as `hot_take`, that would show the model struggles to separate real reasoning from bold opinion.

## Definition of Success

My minimum definition of success is that the fine tuned model should perform better than the Groq zero shot baseline on the same test set, or at least perform similarly while showing clearer behavior on specific labels.

For a real community tool, I would consider the model good enough if it reaches at least 70 percent overall accuracy and at least 0.65 F1 score for every label. I chose this threshold because the task is subjective, the dataset is small, and I do not expect perfect agreement on messy Reddit comments.

A stronger result would be 75 percent or higher overall accuracy with no label below 0.70 F1. If one label performs much worse than the others, I will treat that as a sign that either the label boundary needs improvement or the dataset does not have enough clear examples for that label.

The model should not be used as an automatic moderation tool. A better realistic use would be as a lightweight sorting or analysis tool that helps identify what kind of discourse is happening in a community.

## AI Tool Plan

### Label Stress Testing

Before annotating the full dataset, I will use an AI tool to stress test my label definitions. I will give the AI my four label definitions and ask it to generate borderline NBA comments that could fit more than one label. I will then classify those examples myself.

If the AI generates examples that are hard to classify, I will update the decision rules in this planning document before labeling the full dataset. This will help make the labels more consistent.

### Annotation Assistance

I may use an AI tool to pre label a small batch of examples, but I will manually review every label before adding it to the final dataset. If I use AI pre labeling, I will disclose it in the README and explain what I changed or overrode.

My final labels will be my own decisions, not unchecked AI output. For difficult cases, I will write notes explaining which labels were possible and why I chose the final label.

### Failure Pattern Analysis

After fine tuning, I will use an AI tool to help review the model's wrong predictions. I will give the AI a list of misclassified examples and ask it to look for patterns, such as confusion between `analysis` and `hot_take`, sarcastic comments, very short posts, or comments that depend heavily on NBA context.

I will not accept the AI's analysis automatically. I will re read the examples myself and only include patterns in the README if they are supported by the actual error set.

## Planned Stretch Features

I plan to attempt all four stretch features if time allows.

### Inter Annotator Reliability

I will ask another person to independently label at least 30 examples using my label definitions. I will compare their labels with mine and report either percentage agreement or Cohen's kappa. I will also discuss the disagreements, especially if they happen around `analysis` vs `hot_take` or `reaction` vs `low_effort_noise`.

### Confidence Calibration

I will check whether the fine tuned model's confidence scores are meaningful. For example, I will compare high confidence predictions with lower confidence predictions and see whether higher confidence actually corresponds to higher accuracy.

### Error Pattern Analysis

Beyond three individual wrong predictions, I will identify a systematic pattern in the errors. I expect the model may struggle with short sarcastic comments or posts that combine a bold opinion with one piece of evidence.

### Deployed Interface

I will build or use a simple interface that accepts a new NBA post as input and displays the predicted label and confidence score. This can be a lightweight Streamlit app or a simple Colab demo cell, depending on what is easiest to show clearly in the demo video.
