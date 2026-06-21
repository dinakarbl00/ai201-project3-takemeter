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