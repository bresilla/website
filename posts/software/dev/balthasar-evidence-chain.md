+++
type = "post"
status = "published"
slug = "balthasar-evidence-chain"
title = "The Balthasar Evidence Chain"
description = "How Balthasar decides which parts of a conversation may shape the next session, and keeps the evidence behind every decision."
date = "2026-09-03"
readingtime = 9
thumbnail = "balthasar-hero.jpg"
foot = "Evidence, confidence, provenance, and lifecycle state."
tags = ["balthasar", "memory", "agents", "architecture"]
categories = ["software"]
series = ["EVA-01"]
part = "1"
punchline = "Balthasar records the evidence used to admit a claim into durable memory."
tldr = "Balthasar retains complete history and promotes a claim when its independent evidence reaches the configured threshold. Recall includes provenance and lifecycle state."
credits = [
    "https://github.com/ai-nerv/balthasar",
    "https://unsplash.com/photos/f2krpIYBrJc",
]

[style]
accent = "#d71920"
chromatic = ["#d71920", "#000000"]
background = "#ffffff"
foreground = "#0a0a0a"
theme = "light"
+++

# Evidence-backed memory in Balthasar

Balthasar is a memory layer for agents whose work lasts longer than one context window. It stores historical statements separately from claims admitted into durable memory.

```faqe:prose
columns = [
"""
Most assistant memory starts as a pile of saved snippets. A message gets selected, shortened, and stored. When it turns up again, the agent rarely knows why it was saved or whether it still holds.

Balthasar asks a fussier question: **what evidence does this claim have?** The answer includes the original events, the contexts in which the claim appeared, and the confidence those events produce together.
""",
"""
The complete conversation stays in history. Durable memory is much smaller: only claims that crossed an evidence threshold. Balthasar can therefore find an old statement without quietly upgrading it to fact.

That is what “memory with receipts” means here. When a claim returns, Balthasar can show its source, the later evidence that strengthened it, and any correction that replaced it.
"""
]
```

Balthasar stores the complete history and applies a higher threshold before a claim enters durable memory.

# What happens to one remembered idea

A sentence first enters the historical record. Promotion into durable memory happens later and only for a small subset of statements.

```faqe:timeline
[[items]]
title = "A person or tool says something"
meta = "the original moment is preserved"
body = "Balthasar first keeps the statement as part of session history and records where it came from."
tone = "positive"

[[items]]
title = "The idea enters working memory"
meta = "useful inside the current session"
body = "The agent can use the idea during the current task. Future sessions receive it only after promotion."
tone = "accent"

[[items]]
title = "Evidence attaches"
meta = "request · correction · repetition · consequence"
body = "Different events carry different weight. A direct request is stronger than an idea inferred from a passing remark."
tone = "positive"

[[items]]
title = "Weak evidence waits"
meta = "hold floor · 0.30"
body = "A plausible idea stays as a candidate. It may gain support from another, independent event later."
tone = "warning"

[[items]]
title = "Enough evidence promotes the idea"
meta = "promotion floor · 0.50"
body = "Once the combined evidence clears the floor, the idea becomes durable project memory."
tone = "positive"

[[items]]
title = "New evidence changes confidence"
meta = "confidence is always derived"
body = "Agreement reinforces a memory. A correction supersedes it without erasing the earlier history."
tone = "accent"

[[items]]
title = "Recall returns the idea with context"
meta = "memory plus its supporting history"
body = "The next session gets the claim together with enough provenance to see why it is there."
tone = "positive"
```

The lifecycle distinguishes historical observations, held candidates, and durable claims that may be asserted during recall.

# Eight evidence types

Each signal is a separate source of evidence. A claim may use one strong source or several weaker independent sources.

```faqe:progress
max = 1.0

[[items]]
label = "SAID"
value = 1.0
display = "1.00"
text = "the person explicitly asked for it to be remembered"
tone = "positive"

[[items]]
label = "FIX"
value = 0.8
display = "0.80"
text = "the person corrected something previously believed"
tone = "positive"

[[items]]
label = "SCAR"
value = 0.5
display = "0.50"
text = "the lesson came from an expensive failure or repair"
tone = "positive"

[[items]]
label = "MANUAL"
value = 0.4
display = "0.40"
text = "a trusted peer deliberately proposed the memory"
tone = "warning"

[[items]]
label = "INFERRED"
value = 0.35
display = "0.35"
text = "an optional model found an implied claim"
tone = "warning"

[[items]]
label = "TIDE"
value = 0.3
display = "0.30"
text = "the idea left the active context but still looked useful"
tone = "warning"

[[items]]
label = "CALLUS"
value = 0.25
display = "0.25"
text = "the same lesson returned in an unrelated session"
tone = "warning"

[[items]]
label = "SLEEP"
value = 0.2
display = "0.20"
text = "a later review discovered a recurring pattern"
tone = "warning"

[[markers]]
label = "hold 0.30"
value = 0.3
tone = "warning"

[[markers]]
label = "promote 0.50"
value = 0.5
tone = "positive"
```

```faqe:prose
columns = [
"""
Some evidence is strong enough on its own. An explicit request to remember a preference leaves little doubt about intent. A confirmed correction also stands alone because it tells Balthasar that the old belief is wrong or stale.

A costly failure can also cross alone. Once an agent has found and repaired the cause, making another session pay for the same mistake would be wasteful.
""",
"""
Weaker evidence stays in the holding state. A claim leaving the context window receives a low weight. An independent recurrence in a later session can raise the combined confidence above the promotion floor.

Evidence from one source is capped, so repeated statements in one conversation cannot increase confidence without limit.
"""
]
```

# Where durable memories come from

Evidence enters through direct requests, corrections, failures, context pressure, recurrence, review, inference, and trusted collaboration.

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "direct intent"
title = "Somebody asked"
body = "A person deliberately marked an idea for future sessions."
badge = "crosses alone"
tone = "positive"

[[items]]
eyebrow = "correction"
title = "The world changed"
body = "A new statement replaced something the project previously believed."
badge = "crosses alone"
tone = "positive"

[[items]]
eyebrow = "consequence"
title = "A failure taught a lesson"
body = "The agent found the cause, repaired it, and now has a lesson worth reusing."
badge = "crosses alone"
tone = "positive"

[[items]]
eyebrow = "context pressure"
title = "An idea left the window"
body = "The session moved on, but the idea remained useful enough to retain as a candidate."
badge = "waits"
tone = "warning"

[[items]]
eyebrow = "independent recurrence"
title = "Another session agreed"
body = "The same lesson appeared again in a different situation."
badge = "waits"
tone = "warning"

[[items]]
eyebrow = "later review"
title = "A pattern became visible"
body = "Looking across sessions revealed a habit or repeated outcome."
badge = "waits"
tone = "warning"

[[items]]
eyebrow = "optional inference"
title = "A model read between the lines"
body = "A configured model proposed an implied claim. The normal evidence gate still decides whether it survives."
badge = "waits"
tone = "warning"

[[items]]
eyebrow = "trusted collaboration"
title = "A peer proposed it"
body = "Another identified participant contributed evidence under the same promotion rules."
badge = "waits"
tone = "warning"

[[items]]
eyebrow = "deliberate review"
title = "A person promoted it"
body = "Someone reviewed the session and deliberately selected a lesson for later work."
badge = "strong evidence"
tone = "accent"
```

All evidence types pass through the same promotion rules.

# Memory lifecycle

One run produces history and evidence. Some of that evidence changes durable memory, which may return during the next run.

```faqe:graph
title = "The Balthasar memory loop"
description = "A session produces history and evidence. Evidence promotes durable memories. Recall combines those memories with relevant historical moments for the next session."
columns = 4
rows = 4

[[groups]]
label = "CURRENT SESSION"
column = 1
row = 1
width = 2
height = 2

[[groups]]
label = "DURABLE KNOWLEDGE"
column = 3
row = 1
width = 2
height = 2

[[groups]]
label = "BETWEEN SESSIONS"
column = 2
row = 3
width = 2
height = 2

[[nodes]]
id = "experience"
title = "conversation and action"
subtitle = "what happened now"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "working"
title = "working memory"
subtitle = "useful for this session"
column = 2
row = 1
tone = "neutral"

[[nodes]]
id = "history"
title = "preserved history"
subtitle = "what was actually observed"
column = 1
row = 2
tone = "neutral"

[[nodes]]
id = "evidence"
title = "evidence"
subtitle = "why an idea may matter"
column = 2
row = 2
tone = "warning"

[[nodes]]
id = "memory"
title = "durable memory"
subtitle = "facts · habits · episodes"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "receipts"
title = "receipts"
subtitle = "the supporting moments"
column = 4
row = 1
tone = "positive"

[[nodes]]
id = "recall"
title = "recall"
subtitle = "what is relevant now"
column = 4
row = 2
tone = "accent"

[[nodes]]
id = "review"
title = "review and consolidation"
subtitle = "patterns across sessions"
column = 2
row = 3
tone = "warning"

[[nodes]]
id = "next"
title = "the next session"
subtitle = "memory returns with context"
column = 3
row = 4
tone = "accent"

[[edges]]
from = "experience"
to = "working"
label = "use now"

[[edges]]
from = "experience"
to = "history"
label = "preserve"

[[edges]]
from = "experience"
to = "evidence"
label = "witness"
tone = "warning"

[[edges]]
from = "evidence"
to = "memory"
label = "promote"
tone = "positive"

[[edges]]
from = "memory"
to = "receipts"
label = "explain"
tone = "positive"

[[edges]]
from = "history"
to = "review"
dashed = true
tone = "warning"

[[edges]]
from = "review"
to = "evidence"
label = "reinforce"
dashed = true
tone = "warning"

[[edges]]
from = "memory"
to = "recall"
label = "find"
tone = "positive"

[[edges]]
from = "history"
to = "recall"
label = "quote"
dashed = true
tone = "accent"

[[edges]]
from = "recall"
to = "next"
label = "context"
tone = "accent"
```

History contains all recorded events. Durable memory contains the subset of claims that passed promotion.

# Confidence calculation

Balthasar computes confidence from the evidence attached to each claim.

`confidence = combined independent evidence × freshness × scope relevance`

```faqe:grid
columns = 4
variant = "metrics"

[[items]]
eyebrow = "below 0.10"
title = "history only"
body = "The statement remains in history and is excluded from normal recall."
tone = "muted"

[[items]]
eyebrow = "from 0.10"
title = "findable"
body = "Recall may surface it as a historical clue, clearly separated from established memory."
tone = "neutral"

[[items]]
eyebrow = "from 0.30"
title = "waiting"
body = "The candidate has enough evidence to remain available for reinforcement."
tone = "warning"

[[items]]
eyebrow = "from 0.50"
title = "durable"
body = "The claim has crossed the promotion floor and may support future work."
tone = "positive"
```

Adding, expiring, or superseding evidence triggers confidence recomputation.

# Historical search and durable recall

Balthasar searches historical records and durable claims through separate recall paths. Historical results may include guesses, questions, abandoned plans, and copied text.

```faqe:table
title = "What recall is allowed to do"
variant = "comparison"
columns = ["Kind", "Meaning", "How it returns"]

[[rows]]
cells = ["Historical statement", "Something was said or observed", "As a quote or clue, never automatic truth"]
tones = ["neutral", "neutral", "warning"]

[[rows]]
cells = ["Waiting candidate", "Some evidence exists but it has not crossed", "With uncertainty and provenance"]
tones = ["neutral", "warning", "warning"]

[[rows]]
cells = ["Durable memory", "Independent evidence cleared the floor", "As usable context with its receipts"]
tones = ["neutral", "positive", "positive"]

[[rows]]
cells = ["Superseded memory", "It used to be true or was once believed", "Only when its history explains a change"]
tones = ["neutral", "negative", "neutral"]
```

Historical results return as sourced records and remain excluded from the current-claim lane.

# Work beyond the context window

Short tasks can use the active context directly. Durable memory applies to work whose relevant history exceeds that window.

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "short work"
title = "The window is enough"
body = "Recent instructions and decisions are already present. Adding a memory system may provide no advantage."
badge = "prefer simplicity"
tone = "accent"

[[items]]
eyebrow = "long-running work"
title = "Evidence survives the scroll"
body = "Preferences, corrections, and hard-won lessons remain available after the conversation that produced them is gone from view."
badge = "memory earns its cost"
tone = "positive"
```

The repository evaluation therefore includes both short-history and long-history control arms.

# What Balthasar chooses

```faqe:table
title = "Design choices"
variant = "striped"
columns = ["Question", "Balthasar's answer"]

[[rows]]
cells = ["Should every message become memory?", "No. Every message may remain history; durable claims need evidence."]

[[rows]]
cells = ["Should a model decide what is true?", "No. A model may propose a claim, but promotion remains evidence-driven."]

[[rows]]
cells = ["Should corrections erase the past?", "No. New truth supersedes old belief while preserving the change."]

[[rows]]
cells = ["Should repetition create certainty?", "Evidence from one source is capped, including repeated statements."]

[[rows]]
cells = ["Should recall hide uncertainty?", "No. Findable history and assertable memory remain visibly different."]
```

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "what it optimizes for"
title = "Trust through provenance"
bullets = [
  "Every durable idea retains its supporting moments.",
  "Corrections preserve history instead of deleting it.",
  "Independent evidence matters more than repetition.",
  "The system remains useful without mandatory model calls."
]
tone = "positive"

[[items]]
eyebrow = "known limits"
title = "Memory is not omniscience"
bullets = [
  "An implied idea may remain unnoticed without optional inference.",
  "A weak candidate can wait without ever becoming a fact.",
  "Old evidence can lose relevance as the project changes.",
  "A short task may be better served by its context window alone."
]
tone = "warning"
```

Balthasar admits a sentence into durable memory only when the attached evidence satisfies the configured promotion rules.
