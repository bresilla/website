+++
type = "post"
status = "published"
slug = "balthasar-evidence-chain"
title = "The Balthasar Evidence Chain"
description = "How Balthasar decides which parts of a conversation may shape the next session, and keeps the evidence behind every decision."
date = "2026-09-03"
readingtime = 9
thumbnail = "balthasar-hero.jpg"
foot = "Memory should return with its receipts."
tags = ["balthasar", "memory", "agents", "architecture"]
categories = ["software"]
series = ["EVA-01"]
part = "1"
punchline = "Saving text is easy. Balthasar's real job is deciding which claims have earned the right to shape the next session."
tldr = "Keep the full history, promote a claim only when its evidence clears the threshold, and never return memory without showing where it came from."
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

**THE IDEA**

# Memory that can explain itself

Balthasar is a memory layer for agents whose work lasts longer than one context window. Its first job is to keep “somebody said this” separate from “the agent may rely on this.”

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

Store broadly. Believe carefully. Those are different operations, and Balthasar keeps them separate.

**THE JOURNEY**

# What happens to one remembered idea

A sentence does not jump straight from chat into durable memory. Most statements stay in the historical record. A few earn promotion.

```faqe:timeline
[[items]]
title = "A person or tool says something"
meta = "the original moment is preserved"
body = "Balthasar first keeps the statement as part of the session history. Preservation does not mean belief."
tone = "positive"

[[items]]
title = "The idea enters working memory"
meta = "useful inside the current session"
body = "The agent can use the idea while the current task continues, but the idea has not yet earned a place in future sessions."
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

These states can look alike in a log, but they mean different things: **it happened**, **it may matter**, and **we have enough evidence to assert it**.

**THE EVIDENCE**

# Eight ways a memory earns trust

Each signal is a witness. They are alternative sources of evidence, not a checklist every memory has to complete.

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
Weaker evidence has to wait. A claim that just left the context window may be useful, but age does not make it true. If the lesson turns up independently in a later session, the two events may clear the promotion floor together.

One source has a cap. Repeating the same sentence ten times in one conversation still gives Balthasar one source, not ten.
"""
]
```

**THE SOURCES**

# Where durable memories come from

There is no magic extraction pass that decides what matters. Ordinary events feed evidence into the same gate.

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
body = "Another identified participant contributed evidence, but did not gain the power to declare truth."
badge = "waits"
tone = "warning"

[[items]]
eyebrow = "deliberate review"
title = "A person promoted it"
body = "Someone reviewed the session and deliberately selected a lesson for later work."
badge = "strong evidence"
tone = "accent"
```

> **Many sources, one gate.** The source changes; the admission rules do not.

**THE MEMORY LOOP**

# The whole loop

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

The imbalance is deliberate. History is broad; durable memory is narrow.

**THE NUMBER**

# Confidence is derived, never assigned

Nobody gets to type in a confidence score. Balthasar recomputes it from the evidence attached to the claim.

> **confidence = combined independent evidence × freshness × scope relevance**

```faqe:grid
columns = 4
variant = "metrics"

[[items]]
eyebrow = "below 0.10"
title = "history only"
body = "The statement remains part of the record but does not participate in normal recall."
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

Change the evidence and the number changes too. The score cannot wander away from the reasons behind it.

**RECALL**

# Finding is not believing

Balthasar searches more than it trusts. That matters because a transcript contains guesses, questions, abandoned plans, and text copied from elsewhere.

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

The agent can inspect more of its past without pretending that every old sentence is still true.

**WHY IT MATTERS**

# When the context window runs out

If the useful history still fits in context, keep it there. Durable memory starts earning its cost only when the work outlives the window.

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

There is no prize for adding memory to a short task. Use it when preferences, corrections, or expensive lessons would otherwise scroll away.

**THE TRADE-OFF**

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
cells = ["Should repetition create certainty?", "Not by itself. Evidence from one source is capped."]

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
eyebrow = "what it does not pretend"
title = "Memory is not omniscience"
bullets = [
  "An implied idea may remain unnoticed without optional inference.",
  "A weak candidate can wait without ever becoming a fact.",
  "Old evidence can lose relevance as the project changes.",
  "A short task may be better served by its context window alone."
]
tone = "warning"
```

> Saving a sentence is cheap. The hard question is whether that sentence has earned the right to shape the next session.
