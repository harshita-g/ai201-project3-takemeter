# TakeMeter Planning

## Community

I chose r/nba because it has active basketball discussion with a wide range of discourse styles. Some comments include detailed basketball analysis, some are bold unsupported opinions, and many are emotional reactions to games, highlights, trades, or player news.

This community is a good fit for a classification task because the differences between analysis, hot takes, and reactions are meaningful to NBA fans. A regular r/nba user would understand that a detailed breakdown of spacing or defense is different from a quick “this team is cooked” comment or a hype reaction during a game.

## Label Taxonomy

### analysis

A comment is labeled `analysis` if it makes a basketball-related argument using reasoning, evidence, statistics, tactics, roster construction, player comparison, or historical context.

Examples:
- "The Nuggets create so many open looks because Jokic forces help from the elbow and cutters punish the weak side rotation."
- "The Lakers' spacing gets worse when two non-shooters share the floor because defenses can help off the corners."

### hot_take

A comment is labeled `hot_take` if it makes a bold or confident basketball opinion with little or no real support. The claim may be interesting or even true, but the comment asserts more than it explains.

Examples:
- "This team is cooked and should blow it up immediately."
- "He is the most overrated superstar in the league."

### reaction

A comment is labeled `reaction` if it is mainly an emotional, humorous, hype, frustration, meme-like, or immediate response with little or no argument.

Examples:
- "NO WAY HE HIT THAT."
- "This game is absolute chaos lmao."

## Hard Edge Cases

Some comments combine a strong opinion with one statistic or one piece of evidence. These can be ambiguous between `analysis` and `hot_take`.

Decision rule: I will label a comment as `analysis` only if the evidence is used to support a clear basketball argument. If the statistic or evidence is mostly used as a punchline, insult, or exaggerated claim, I will label it as `hot_take`.

Example:
- "LeBron is overrated because his record against top-seeded teams is terrible."

Possible labels: `analysis` or `hot_take`.

Decision: `hot_take`, because the comment uses a stat-like claim to support a broad opinion but does not explain the context, comparison, or reasoning enough to count as analysis.

Another difficult case is sarcasm. If a sarcastic comment is mainly a joke or emotional response, I will label it as `reaction`. If it makes a clear unsupported basketball claim, I will label it as `hot_take`.

## Data Collection Plan

I will collect at least 200 public posts or comments from r/nba. I will use public Reddit threads such as game threads, post-game threads, highlight threads, trade/news threads, and player discussion threads.

I will aim for a balanced dataset:

| Label | Target Count |
|---|---:|
| analysis | 65–70 |
| hot_take | 65–70 |
| reaction | 65–70 |

If one label is underrepresented after collecting 200 examples, I will collect more examples from thread types where that label is more common. For example, if I need more `analysis`, I will look at serious player/team discussion threads. If I need more `reaction`, I will look at game threads or highlight threads.

## Evaluation Metrics

I will evaluate the model using overall accuracy, per-class precision, recall, F1-score, and a confusion matrix.

Accuracy is useful for understanding total performance, but it is not enough because the model could get high accuracy by mostly predicting the most common label. Per-class F1 is important because I want to know whether the model learns all three discourse types, especially the harder boundary between `analysis` and `hot_take`.

The confusion matrix will help show which labels the model confuses. For this task, the most important confusion is likely `analysis` vs. `hot_take`, because both can involve opinions but differ in how much reasoning they provide.

## Definition of Success

I will consider the classifier successful if the fine-tuned model performs better than the zero-shot Groq baseline on the same test set.

For a useful prototype, I want the fine-tuned model to reach around 70% overall accuracy and avoid having any label with extremely poor F1-score. I expect `reaction` to be easier to classify, while `analysis` and `hot_take` may be harder because they sometimes overlap.

## AI Tool Plan

### Label stress-testing

I will use an AI tool to generate borderline r/nba-style comments between `analysis`, `hot_take`, and `reaction`. If the generated examples are difficult to classify using my definitions, I will tighten my decision rules before labeling the full dataset.

### Annotation assistance

I may use an AI tool to suggest labels for some examples, but I will manually review every label myself. I will not accept labels without reading the comment and checking it against my definitions.

### Failure analysis

After training and evaluation, I will use an AI tool to review the model’s wrong predictions and identify possible error patterns. I will verify those patterns myself by rereading the examples before including them in the README.
