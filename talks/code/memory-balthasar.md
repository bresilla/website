---
title: "Memory, **then Balthasar**"
sub_title: How human memory works, why retrieval matters, and what that taught us about agent memory
author: Trim Bresilla
date: 2026-09-03
theme:
  name: light
  override:
    default:
      colors:
        foreground: "0a0a0a"
        background: "ffffff"
    palette:
      colors:
        accent: "d71920"
        chromatic: "000000"
    slide_title:
      colors:
        foreground: "d71920"
options:
  incremental_lists: false
---

Functional definition of memory
====================

Memory is a **change in a system** that lets the past alter what happens next.

<!-- pause -->

Three things have to work:

1. **Encoding:** something changes now.
2. **Persistence:** that change survives.
3. **Retrieval:** a later cue makes the change useful.

If any one fails, we experience forgetting.

<!-- speaker_note: Start here because “memory” is often reduced to storage. Storage is only the middle step; the past has to become useful again later. -->

<!-- end_slide -->

Major memory systems
=========================

```faqe:grid
columns = 4
variant = "cards"

[[items]]
eyebrow = "seconds"
title = "Working"
body = "Keeps a small amount active while reasoning, speaking, or acting."
badge = "active state"
tone = "accent"

[[items]]
eyebrow = "events"
title = "Episodic"
body = "Remembers what happened, where, when, and from whose perspective."
badge = "experienced"
tone = "positive"

[[items]]
eyebrow = "knowledge"
title = "Semantic"
body = "Carries facts and concepts without requiring the original episode."
badge = "known"
tone = "neutral"

[[items]]
eyebrow = "skills"
title = "Procedural"
body = "Changes performance through practice, often without verbal recall."
badge = "enacted"
tone = "warning"
```

They work together, but each one develops and fails differently.

<!-- speaker_note: The taxonomy is functional, not four sealed boxes. Working memory supports the current task. Episodic memory represents lived events. Semantic memory abstracts knowledge. Procedural memory changes skilled behavior. -->

<!-- end_slide -->

Encoding, consolidation, and retrieval
=============================

```faqe:graph
title = "From experience to later action"
description = "Attention keeps part of an event. Consolidation stabilizes and reorganizes the trace. Later, a cue helps reconstruct something useful from it."
columns = 5
rows = 2

[[nodes]]
id = "event"
title = "event"
subtitle = "more arrives than can be kept"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "attention"
title = "attention"
subtitle = "selection and meaning"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "trace"
title = "distributed trace"
subtitle = "activity and plasticity"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "consolidate"
title = "consolidation"
subtitle = "stabilize and reorganize"
column = 4
row = 1
tone = "positive"

[[nodes]]
id = "cue"
title = "retrieval cue"
subtitle = "partial present input"
column = 4
row = 2
tone = "accent"

[[nodes]]
id = "reconstruct"
title = "reconstruction"
subtitle = "usable, not verbatim"
column = 5
row = 1
tone = "positive"

[[nodes]]
id = "update"
title = "update"
subtitle = "the trace may change again"
column = 5
row = 2
tone = "warning"

[[edges]]
from = "event"
to = "attention"
label = "select"

[[edges]]
from = "attention"
to = "trace"
label = "encode"
tone = "accent"

[[edges]]
from = "trace"
to = "consolidate"
label = "persist"
tone = "positive"

[[edges]]
from = "cue"
to = "reconstruct"
label = "reactivate"
tone = "accent"

[[edges]]
from = "consolidate"
to = "reconstruct"
label = "support"
tone = "positive"

[[edges]]
from = "reconstruct"
to = "update"
label = "reconsolidate"
tone = "warning"
```

<!-- speaker_note: Keep coming back to this loop. We select, encode, consolidate, retrieve, and sometimes change the memory by retrieving it. Nothing here behaves like moving a file between folders. -->

<!-- end_slide -->

Encoding constraints
========================

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "signal"
title = "Attention"
body = "Competition decides what receives enough processing to leave a useful trace."
tone = "accent"

[[items]]
eyebrow = "structure"
title = "Meaning"
body = "Relating new material to existing knowledge creates more routes back to it."
tone = "positive"

[[items]]
eyebrow = "conditions"
title = "State"
body = "Stress, sleep, emotion, goals, and context shape what is encoded and how."
tone = "warning"

[[items]]
eyebrow = "failure"
title = "Never encoded"
body = "Some apparent forgetting happened during encoding because the information received little attention."
tone = "negative"

[[items]]
eyebrow = "support"
title = "Elaboration"
body = "Examples and explanations give us more ways to find the material later."
tone = "positive"

[[items]]
eyebrow = "boundary"
title = "No perfect copy"
body = "What is stored already reflects selection, interpretation, and prior knowledge."
tone = "neutral"
```

<!-- speaker_note: Memory begins by discarding most of the input. Better encoding comes from structure and prior knowledge, not from trying to absorb everything. -->

<!-- end_slide -->

Distributed engrams
=========================

An **engram** is the physical ensemble changed by learning and capable of contributing to later recall.

<!-- pause -->

```faqe:graph
title = "One episode, many components"
columns = 4
rows = 2

[[nodes]]
id = "episode"
title = "episode"
subtitle = "the remembered event"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "sensory"
title = "sensory detail"
subtitle = "sights · sounds"
column = 1
row = 2
tone = "neutral"

[[nodes]]
id = "space"
title = "place and time"
subtitle = "context"
column = 2
row = 2
tone = "positive"

[[nodes]]
id = "meaning"
title = "meaning"
subtitle = "conceptual relations"
column = 3
row = 2
tone = "warning"

[[nodes]]
id = "emotion"
title = "value"
subtitle = "emotion and goals"
column = 4
row = 2
tone = "accent"

[[edges]]
from = "episode"
to = "sensory"

[[edges]]
from = "episode"
to = "space"

[[edges]]
from = "episode"
to = "meaning"

[[edges]]
from = "episode"
to = "emotion"
```

We can still identify the memory even while the participating cells and connection strengths change.

<!-- speaker_note: Modern engram research describes sparse, distributed ensembles. The participating cells and connection strengths can change while the memory remains identifiable. -->

<!-- end_slide -->

Consolidation timescales
===================================

```faqe:table
title = "Stabilization is not one process"
variant = "comparison"
columns = ["Scale", "Timescale", "What changes", "Why it matters"]

[[rows]]
cells = ["Synaptic / cellular", "minutes to hours", "Gene expression, protein synthesis, and synaptic strength", "Makes a newly encoded trace less fragile"]
tones = ["accent", "neutral", "neutral", "positive"]

[[rows]]
cells = ["Systems", "days to years", "Coordination among hippocampal and cortical networks", "Reorganizes how a memory can be retrieved"]
tones = ["accent", "neutral", "neutral", "positive"]

[[rows]]
cells = ["Reactivation", "repeated offline moments", "Parts of recent activity patterns recur", "Strengthens, integrates, and sometimes transforms"]
tones = ["accent", "neutral", "neutral", "warning"]
```

Consolidation can make a memory last, but the version that lasts may keep the gist and lose incidental detail.

<!-- speaker_note: Resist the computer metaphor. A memory is not copied from a temporary folder into a permanent one; several biological processes reshape it over different timescales. -->

<!-- end_slide -->

Offline consolidation
==============================

```faqe:timeline
[[items]]
title = "Experience creates a fresh pattern"
meta = "awake · task engaged"
body = "A recent event establishes an initially fragile set of relationships."
tone = "accent"

[[items]]
title = "Quiet rest reduces competition"
meta = "minutes after learning"
body = "New input slows, allowing recent activity to recur and interact with existing knowledge."
tone = "neutral"

[[items]]
title = "Sleep coordinates replay"
meta = "offline reactivation"
body = "Hippocampal and cortical rhythms support reactivation and redistribution."
tone = "positive"

[[items]]
title = "The next retrieval is changed"
meta = "later behavior"
body = "Recall can become faster, more abstract, or less dependent on the original context."
tone = "warning"
```

<!-- speaker_note: Sleep is not a save button. Offline reactivation helps consolidate a memory, and may transform it along the way. -->

<!-- end_slide -->

Retrieval types
==========================

Retrieval is the use of a present cue to **reactivate enough of a past pattern** to guide thought or action.

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "produce"
title = "Free recall"
body = "Generate the memory with minimal external structure."
tone = "accent"

[[items]]
eyebrow = "prompt"
title = "Cued recall"
body = "A partial clue narrows the search and completes a pattern."
tone = "positive"

[[items]]
eyebrow = "choose"
title = "Recognition"
body = "Judge whether a currently presented item was encountered before."
tone = "neutral"

[[items]]
eyebrow = "order"
title = "Serial recall"
body = "Recover items together with their learned sequence."
tone = "warning"

[[items]]
eyebrow = "future"
title = "Prospective"
body = "Remember to perform an intended action when the right situation arrives."
tone = "positive"

[[items]]
eyebrow = "behavior"
title = "Implicit expression"
body = "Past experience changes performance without conscious recollection."
tone = "neutral"
```

<!-- speaker_note: These tasks impose different cue conditions. Recognition is usually easier than free recall because the target itself is present. Prospective memory reverses the direction: a future cue must trigger an intended act. -->

<!-- end_slide -->

Reconstructive retrieval
===========================

```faqe:graph
title = "Retrieval reconstructs from partial evidence"
columns = 5
rows = 2

[[nodes]]
id = "cue"
title = "present cue"
subtitle = "question · smell · place"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "context"
title = "current context"
subtitle = "goals and expectations"
column = 2
row = 2
tone = "warning"

[[nodes]]
id = "trace"
title = "surviving trace"
subtitle = "partial distributed pattern"
column = 3
row = 1
tone = "neutral"

[[nodes]]
id = "knowledge"
title = "prior knowledge"
subtitle = "schemas and language"
column = 3
row = 2
tone = "positive"

[[nodes]]
id = "memory"
title = "remembered account"
subtitle = "coherent reconstruction"
column = 5
row = 1
tone = "accent"

[[edges]]
from = "cue"
to = "trace"
label = "search"
tone = "accent"

[[edges]]
from = "context"
to = "memory"
label = "shape"
tone = "warning"

[[edges]]
from = "trace"
to = "memory"
label = "reactivate"

[[edges]]
from = "knowledge"
to = "memory"
label = "complete"
tone = "positive"
```

The reconstructed account can be useful and sincerely held without being a literal replay.

<!-- speaker_note: Reconstruction is why memory is useful and why it can mislead us. Prior knowledge fills gaps, current goals pull some details forward, and confidence does not measure historical accuracy. -->

<!-- end_slide -->

Context-dependent recall
==============================

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "encoding specificity"
title = "Match helps"
body = "A cue works when it overlaps with how the target was represented during learning."
bullets = ["location", "emotional state", "wording", "task", "related concepts"]
tone = "positive"

[[items]]
eyebrow = "transfer"
title = "Variety helps more"
body = "Practicing under varied cues makes a memory less dependent on one fragile route."
bullets = ["different questions", "new examples", "mixed contexts", "explain to others", "apply in a task"]
tone = "accent"
```

Recall becomes more reliable when practice creates several usable cues.

<!-- speaker_note: Matching the original context helps, but relying on one cue makes recall brittle. Practice with different questions, examples, and settings. -->

<!-- end_slide -->

Retrieval practice
==================================

```faqe:timeline
[[items]]
title = "Attempt retrieval"
meta = "before looking"
body = "The learner tries to reconstruct an answer from available cues."
tone = "accent"

[[items]]
title = "Expose the gap"
meta = "success or failure"
body = "The attempt measures what the learner can produce from memory; rereading mainly measures familiarity."
tone = "warning"

[[items]]
title = "Receive feedback"
meta = "correct and complete"
body = "Feedback repairs errors before they become the next practiced response."
tone = "positive"

[[items]]
title = "Retrieve again later"
meta = "spaced and varied"
body = "A later successful reconstruction strengthens access under a new temporal cue."
tone = "positive"
```

Rereading changes familiarity. Retrieval practice changes the ability to produce and use knowledge.

<!-- speaker_note: Testing changes learning; it does more than measure it. Feedback matters most after a failed or incorrect attempt. -->

<!-- end_slide -->

Reconsolidation
===========================

```faqe:graph
title = "Retrieval and reconsolidation"
columns = 5
rows = 2

[[nodes]]
id = "stable"
title = "stored memory"
subtitle = "currently consolidated"
column = 1
row = 1
tone = "positive"

[[nodes]]
id = "retrieve"
title = "retrieval"
subtitle = "reactivation"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "labile"
title = "labile window"
subtitle = "temporarily modifiable"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "new"
title = "new information"
subtitle = "correction or context"
column = 3
row = 2
tone = "accent"

[[nodes]]
id = "updated"
title = "updated memory"
subtitle = "reconsolidated"
column = 5
row = 1
tone = "positive"

[[edges]]
from = "stable"
to = "retrieve"

[[edges]]
from = "retrieve"
to = "labile"
label = "destabilize"
tone = "warning"

[[edges]]
from = "new"
to = "labile"
label = "integrate"
tone = "accent"

[[edges]]
from = "labile"
to = "updated"
label = "restabilize"
tone = "positive"
```

Remembering can strengthen, weaken, distort, or update what will be remembered next.

<!-- speaker_note: Reconsolidation does not follow every retrieval. Under the right conditions, though, retrieval makes a memory changeable before it settles again. -->

<!-- end_slide -->

Forgetting mechanisms
==========================================

```faqe:table
title = "Why access fails"
variant = "striped"
columns = ["Mechanism", "What happened", "Useful side", "Risk"]

[[rows]]
cells = ["Decay / drift", "The trace or its accessibility changed with time", "Reduces obsolete detail", "Useful knowledge disappears"]

[[rows]]
cells = ["Interference", "Similar memories compete", "Supports abstraction", "Old and new answers collide"]

[[rows]]
cells = ["Cue failure", "Available cues fail to reactivate the trace", "Keeps irrelevant traces quiet", "Relevant information remains inaccessible"]

[[rows]]
cells = ["Inhibition", "Competing content is suppressed", "Protects current goals", "Suppressed content becomes hard to recover"]
```

Perfect retention would increase interference between old details and currently relevant information.

<!-- speaker_note: Forgetting can be useful filtering. A workable memory system controls stale and competing material instead of maximizing the number of stored items. -->

<!-- end_slide -->

History of memory techniques
=============================

```faqe:timeline
[[items]]
title = "Oral pattern"
meta = "before widespread writing"
body = "Rhythm, meter, formula, story, and repetition make knowledge reproducible."
tone = "neutral"

[[items]]
title = "Method of loci"
meta = "classical antiquity"
body = "Imagined material is placed along a stable spatial route and retrieved by walking it."
tone = "accent"

[[items]]
title = "External notebooks"
meta = "manuscript to print cultures"
body = "Commonplace books move selected quotations and ideas into a personal index."
tone = "positive"

[[items]]
title = "Experimental memory"
meta = "Ebbinghaus · 1885"
body = "Controlled study measures forgetting, relearning, and the benefit of spacing."
tone = "warning"

[[items]]
title = "Scheduled review"
meta = "Leitner · SM-2 · flashcards"
body = "Recall success determines when an item should return."
tone = "positive"

[[items]]
title = "Networked retrieval"
meta = "search · embeddings · graphs"
body = "External systems find related records at scales that unaided recall cannot scan."
tone = "accent"

[[items]]
title = "Agent memory"
meta = "2023 onward"
body = "Systems must decide what to retain, update, retrieve, trust, and forget across trajectories."
tone = "negative"
```

<!-- speaker_note: The tools change, but the pressure stays familiar: make recall more reliable. Once machines join the loop, selection and provenance become harder than raw storage. -->

<!-- end_slide -->

Method of loci
==============

```faqe:graph
title = "A familiar route becomes a retrieval scaffold"
columns = 5
rows = 2

[[nodes]]
id = "door"
title = "front door"
subtitle = "first idea"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "hall"
title = "hallway"
subtitle = "second idea"
column = 2
row = 2
tone = "positive"

[[nodes]]
id = "table"
title = "table"
subtitle = "third idea"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "stairs"
title = "stairs"
subtitle = "fourth idea"
column = 4
row = 2
tone = "accent"

[[nodes]]
id = "window"
title = "window"
subtitle = "fifth idea"
column = 5
row = 1
tone = "positive"

[[edges]]
from = "door"
to = "hall"
label = "walk"

[[edges]]
from = "hall"
to = "table"
label = "walk"

[[edges]]
from = "table"
to = "stairs"
label = "walk"

[[edges]]
from = "stairs"
to = "window"
label = "walk"
```

The method binds unfamiliar items to a route you already know well. Distinctive images help. Studies report strong serial-recall gains, though the size of the effect varies by study and learner.

<!-- speaker_note: The route supplies order; the strange images make each stop distinctive. The recent meta-analysis found immediate serial-recall benefits, with substantial variation and possible publication bias. -->

<!-- end_slide -->

Spaced practice
=================================

```faqe:graph
title = "A conceptual forgetting curve"
description = "Without successful retrieval, accessibility drops quickly and then more slowly. Each effortful retrieval raises accessibility and changes the next curve."
columns = 6
rows = 3

[[nodes]]
id = "learn"
title = "learn"
subtitle = "high accessibility"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "drop1"
title = "forget"
subtitle = "fast early loss"
column = 2
row = 2
tone = "negative"

[[nodes]]
id = "review1"
title = "retrieve"
subtitle = "effort + feedback"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "drop2"
title = "forget"
subtitle = "slower decline"
column = 4
row = 2
tone = "warning"

[[nodes]]
id = "review2"
title = "retrieve"
subtitle = "longer interval"
column = 5
row = 1
tone = "positive"

[[nodes]]
id = "durable"
title = "durable access"
subtitle = "varied future cues"
column = 6
row = 1
tone = "accent"

[[edges]]
from = "learn"
to = "drop1"
tone = "negative"

[[edges]]
from = "drop1"
to = "review1"
label = "test"
tone = "positive"

[[edges]]
from = "review1"
to = "drop2"
tone = "warning"

[[edges]]
from = "drop2"
to = "review2"
label = "test"
tone = "positive"

[[edges]]
from = "review2"
to = "durable"
label = "transfer"
tone = "accent"
```

There is no universal spacing interval. It depends on how well you know the material and how long you need to keep it.

<!-- speaker_note: This is a concept sketch, not Ebbinghaus data. The next review should arrive after recall has become harder, but before it becomes hopeless. -->

<!-- end_slide -->

Study sequence
=========================

```faqe:progress
max = 1.0

[[items]]
label = "reread"
value = 0.25
display = "familiar"
text = "easy exposure can inflate confidence"
tone = "neutral"

[[items]]
label = "recall"
value = 0.7
display = "diagnostic"
text = "producing an answer exposes access"
tone = "accent"

[[items]]
label = "feedback"
value = 0.85
display = "corrective"
text = "repair the response and fill omissions"
tone = "warning"

[[items]]
label = "spacing"
value = 1.0
display = "durable"
text = "return after enough time for effort"
tone = "positive"
```

The values are illustrative. The labels describe the function of each study step.

<!-- speaker_note: Do not read fake precision into these bars. Rereading gives exposure, recall tests access, feedback repairs mistakes, and spacing chooses when to try again. -->

<!-- end_slide -->

External memory systems
===================================

```faqe:table
title = "From remembering content to remembering how to find it"
variant = "comparison"
columns = ["System", "Strength", "New burden"]

[[rows]]
cells = ["Commonplace book", "Curated excerpts and personal categories", "Selection and a usable index"]

[[rows]]
cells = ["Zettelkasten", "Atomic notes linked into a growing idea network", "Stable identifiers and meaningful links"]

[[rows]]
cells = ["Search engine", "Large-scale lexical retrieval", "Query formulation and source judgment"]

[[rows]]
cells = ["Vector retrieval", "Finds semantic similarity across wording", "Similarity is not truth or task relevance"]

[[rows]]
cells = ["Agent memory", "Past can shape autonomous future action", "Authority, provenance, correction, and forgetting"]
```

<!-- speaker_note: External memory saves us from retaining everything ourselves, then creates a new problem: who chose the material, and why should we trust it? -->

<!-- end_slide -->

Memory locations in agent systems
=====================================

```faqe:grid
columns = 4
variant = "cards"

[[items]]
eyebrow = "trained"
title = "Parameters"
body = "Slow, compressed capabilities and associations learned before the current interaction."
badge = "broad · opaque"
tone = "neutral"

[[items]]
eyebrow = "active"
title = "Context"
body = "Immediate instructions, observations, and intermediate work available to the model now."
badge = "precise · bounded"
tone = "accent"

[[items]]
eyebrow = "stored"
title = "External records"
body = "Transcripts, databases, notes, artifacts, and evidence retained outside the model."
badge = "large · searchable"
tone = "positive"

[[items]]
eyebrow = "compiled"
title = "Skills and policy"
body = "Reusable procedures and constraints that shape how an agent acts."
badge = "actionable · governed"
tone = "warning"
```

Boundary policy specifies which information may move between these stores, its authority, and its supporting evidence.

<!-- speaker_note: “Agent memory” is not another name for a vector database. Parameters, live context, external records, and learned procedures change at different speeds and carry different risks. -->

<!-- end_slide -->

Recent agent-memory systems
============================

```faqe:timeline
[[items]]
title = "MemGPT"
meta = "2023 · memory tiers"
body = "Applies an operating-system and virtual-memory analogy to move information beyond a fixed context window."
tone = "accent"

[[items]]
title = "HippoRAG"
meta = "2024 · graph retrieval"
body = "Combines hippocampal-indexing inspiration, knowledge graphs, and Personalized PageRank for multi-hop recall."
tone = "positive"

[[items]]
title = "LongMemEval"
meta = "2024–2025 · evaluation"
body = "Tests extraction, multi-session reasoning, temporal reasoning, updates, and abstention across long histories."
tone = "warning"

[[items]]
title = "A-MEM and Mem0"
meta = "2025 · dynamic organization"
body = "Explore evolving linked notes and selective extraction, consolidation, retrieval, and graph memory."
tone = "accent"

[[items]]
title = "MemoryOS and MemOS"
meta = "2025 · systems layer"
body = "Treat memory as managed tiers or resources with lifecycle, provenance, and versioning concerns."
tone = "positive"

[[items]]
title = "LongMemEval-V2 and LeanMem"
meta = "2026 · scale and control"
body = "Push evaluation into huge multimodal trajectories and adapt memory types and budgets to each query."
tone = "negative"
```

<!-- speaker_note: Read this as a map, not a ranking. Recent work spends less time asking how to extend context and more time on lifecycle, evaluation, provenance, and control. -->

<!-- end_slide -->

Long-term memory requirements
=============================

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "LongMemEval"
title = "Extract"
body = "Find a needed detail buried in a long interaction history."
tone = "neutral"

[[items]]
eyebrow = "LongMemEval"
title = "Reason across sessions"
body = "Combine evidence that never appeared in one place or one conversation."
tone = "accent"

[[items]]
eyebrow = "LongMemEval"
title = "Respect time"
body = "Understand order, duration, recency, and time-sensitive constraints."
tone = "positive"

[[items]]
eyebrow = "LongMemEval"
title = "Update knowledge"
body = "Prefer the latest valid state while preserving how it changed."
tone = "warning"

[[items]]
eyebrow = "LongMemEval"
title = "Abstain"
body = "Abstain when the available history cannot support a reliable answer."
tone = "negative"

[[items]]
eyebrow = "real systems"
title = "Control influence"
body = "Keep untrusted text from silently becoming durable instruction."
tone = "accent"
```

<!-- speaker_note: LongMemEval defines the first five abilities. The sixth card adds authority and influence control for agents that can act on recalled information. -->

<!-- end_slide -->

Comparison of recent systems
===================================

```faqe:table
title = "Recent agent-memory approaches"
variant = "comparison"
columns = ["System", "Core bet", "Retrieval unit", "Open pressure"]

[[rows]]
cells = ["MemGPT", "Tiered memory management", "managed records", "what should move between tiers?"]

[[rows]]
cells = ["HippoRAG", "Associative graph navigation", "entities and passages", "does relevance imply correctness?"]

[[rows]]
cells = ["A-MEM", "Notes evolve and link dynamically", "linked memories", "how are bad links repaired?"]

[[rows]]
cells = ["Mem0", "Selective consolidation with graph relations", "facts and relations", "how is provenance preserved?"]

[[rows]]
cells = ["MemOS", "Memory as a managed resource", "versioned MemCubes", "who controls lifecycle policy?"]

[[rows]]
cells = ["LeanMem", "Choose memory type and budget per query", "profile, event, or source", "can selection remain trustworthy?"]
```

<!-- speaker_note: Each system picks a storage unit, an update rule, and a retrieval method. None escapes the awkward question: a fluent, relevant result may still be wrong. -->

<!-- end_slide -->

LongMemEval results
=====================

```faqe:grid
columns = 3
variant = "metrics"

[[items]]
eyebrow = "LongMemEval"
title = "500"
body = "questions spanning sustained multi-session interaction"
tone = "accent"

[[items]]
eyebrow = "reported baseline gap"
title = "~30%"
body = "accuracy drop attributed to long-term memory challenges"
tone = "warning"

[[items]]
eyebrow = "LongMemEval-V2"
title = "451"
body = "questions grounded in web-agent trajectories"
tone = "positive"

[[items]]
eyebrow = "maximum history"
title = "500"
body = "trajectories for one user history"
tone = "negative"

[[items]]
eyebrow = "maximum scale"
title = "115M"
body = "tokens in multimodal interaction history"
tone = "accent"

[[items]]
eyebrow = "real question"
title = "experience"
body = "does an agent improve because it has lived through prior work?"
tone = "positive"
```

LongMemEval-V2 evaluates retrieval across histories containing as many as 500 trajectories and 115 million multimodal tokens.

<!-- speaker_note: These are the benchmark authors' figures. Long context will improve, but replaying an entire working life on every query is still a poor systems plan. -->

<!-- end_slide -->

Retrieval and evidential status
======================

```faqe:graph
title = "The missing gate"
columns = 5
rows = 2

[[nodes]]
id = "record"
title = "stored record"
subtitle = "something was written"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "similar"
title = "similar result"
subtitle = "the query matched"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "source"
title = "source and scope"
subtitle = "who · where · when"
column = 3
row = 2
tone = "warning"

[[nodes]]
id = "evidence"
title = "evidence gate"
subtitle = "why should it influence?"
column = 4
row = 1
tone = "negative"

[[nodes]]
id = "context"
title = "usable context"
subtitle = "qualified and current"
column = 5
row = 1
tone = "positive"

[[edges]]
from = "record"
to = "similar"
label = "retrieve"
tone = "accent"

[[edges]]
from = "similar"
to = "evidence"
label = "candidate"
tone = "warning"

[[edges]]
from = "source"
to = "evidence"
label = "qualify"
tone = "warning"

[[edges]]
from = "evidence"
to = "context"
label = "admit"
tone = "positive"
```

Similarity ranks possible records. Evidence determines whether a retrieved claim may influence later work.

<!-- speaker_note: This is the turn toward Balthasar. Retrieval finds candidates; Balthasar separately decides which claims may influence later work. -->

<!-- end_slide -->

<!-- jump_to_middle -->
<!-- alignment: center -->

Balthasar
=========

Balthasar exposes short-term and long-term agent memory through a Lua interface.

Balthasar runs as a separate program and exposes memory operations to the harness over a process boundary.

<!-- speaker_note: Pause here. Balthasar belongs to the ai-nerv family, but it is its own binary and store. That boundary is deliberate: memory should outlive any one harness. -->

<!-- end_slide -->

EVA-01: the NERV family
=======================

```faqe:graph
title = "Programs cooperate across explicit boundaries"
columns = 4
rows = 2

[[nodes]]
id = "magi"
title = "magi"
subtitle = "harness · UI · providers"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "melchior"
title = "melchior"
subtitle = "agent · sessions · permissions"
column = 2
row = 1
tone = "positive"

[[nodes]]
id = "balthasar"
title = "balthasar"
subtitle = "memory · evidence · recall"
column = 3
row = 1
tone = "negative"

[[nodes]]
id = "casper"
title = "casper"
subtitle = "tools · tool APIs"
column = 4
row = 1
tone = "warning"

[[nodes]]
id = "protocol"
title = "sockets and pipes"
subtitle = "replaceable program boundaries"
column = 2
row = 2
tone = "neutral"

[[edges]]
from = "magi"
to = "melchior"
label = "hosts"

[[edges]]
from = "melchior"
to = "balthasar"
label = "remembers"
tone = "negative"

[[edges]]
from = "melchior"
to = "casper"
label = "acts"
tone = "warning"

[[edges]]
from = "protocol"
to = "melchior"
label = "boundary"
dashed = true

[[edges]]
from = "protocol"
to = "balthasar"
label = "boundary"
dashed = true
```

Each program has an independent test and replacement boundary.

<!-- speaker_note: Magi is the harness, Melchior the agent layer, Balthasar the memory service, and Casper the tools direction. Keeping those jobs separate makes the boundaries visible. -->

<!-- end_slide -->

Balthasar evidence model
==================

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "preserve broadly"
title = "History says what happened"
body = "Keep verbatim turns and searchable spans. A historical statement records that the words appeared and preserves their source."
badge = "findable"
tone = "neutral"

[[items]]
eyebrow = "assert narrowly"
title = "Memory says what may guide"
body = "Durable facts, habits, and episodes must name witnesses and cross an evidence threshold before normal assertion."
badge = "usable"
tone = "positive"
```

Balthasar can keep a record searchable after its confidence falls below the threshold for normal assertion.

<!-- speaker_note: This asymmetry is the core of the design. Old or questionable material remains available for investigation, but cannot quietly return as a current instruction. -->

<!-- end_slide -->

Storage scopes
==========================

```faqe:graph
title = "Balthasar's memory surfaces"
columns = 4
rows = 2

[[groups]]
label = "RUN"
column = 1
row = 1
width = 2
height = 2

[[groups]]
label = "DURABLE"
column = 3
row = 1
width = 2
height = 2

[[nodes]]
id = "window"
title = "window"
subtitle = "active model context"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "transcript"
title = "transcript.db"
subtitle = "verbatim spans · evidence"
column = 1
row = 2
tone = "neutral"

[[nodes]]
id = "scratch"
title = "memory.db"
subtitle = "run scratch · expires"
column = 2
row = 2
tone = "warning"

[[nodes]]
id = "project"
title = "project.db"
subtitle = "facts · habits · episodes"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "global"
title = "global.db"
subtitle = "globally scoped truth"
column = 4
row = 1
tone = "negative"

[[nodes]]
id = "procedure"
title = "how things are done"
subtitle = "durable procedural memory"
column = 3
row = 2
tone = "accent"

[[edges]]
from = "window"
to = "transcript"
label = "preserve"

[[edges]]
from = "window"
to = "scratch"
label = "work"
tone = "warning"

[[edges]]
from = "transcript"
to = "project"
label = "witness"
tone = "positive"

[[edges]]
from = "project"
to = "global"
label = "scope"
dashed = true
tone = "negative"

[[edges]]
from = "project"
to = "procedure"
label = "guide"
tone = "accent"
```

<!-- speaker_note: Transcript and scratch data belong to one run. Project and global stores survive it. A temporary thought should not become permanent simply because it happened to be in context. -->

<!-- end_slide -->

Evidence sources
===============

```faqe:progress
max = 1.0

[[items]]
label = "SAID"
value = 1.0
display = "1.00"
text = "explicit remember or pin"
tone = "positive"

[[items]]
label = "FIX"
value = 0.8
display = "0.80"
text = "a correction to prior belief"
tone = "positive"

[[items]]
label = "SCAR"
value = 0.5
display = "0.50"
text = "a costly failure and repair"
tone = "positive"

[[items]]
label = "MANUAL"
value = 0.4
display = "0.40"
text = "a trusted peer proposed it"
tone = "warning"

[[items]]
label = "INFERRED"
value = 0.35
display = "0.35"
text = "an optional model inferred it"
tone = "warning"

[[items]]
label = "TIDE"
value = 0.3
display = "0.30"
text = "it scrolled out of context"
tone = "warning"

[[items]]
label = "CALLUS"
value = 0.25
display = "0.25"
text = "it recurred in another run"
tone = "warning"

[[items]]
label = "SLEEP"
value = 0.2
display = "0.20"
text = "consolidation found a pattern"
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

The weight applies to the evidence event and is unrelated to how important the sentence sounds.

<!-- speaker_note: SAID, FIX, and SCAR can promote a claim by themselves. The weaker witnesses have to combine. Their names let us inspect the score instead of trusting one mysterious number. -->

<!-- end_slide -->

Claim lifecycle
================

```faqe:timeline
[[items]]
title = "Observed"
meta = "transcript span"
body = "The exact statement is retained in history. Promotion is evaluated separately."
tone = "neutral"

[[items]]
title = "Proposed"
meta = "candidate claim"
body = "A compact statement identifies what might be reusable in a future session."
tone = "accent"

[[items]]
title = "Held"
meta = "confidence ≥ 0.30"
body = "Weak evidence keeps the candidate available for another independent witness."
tone = "warning"

[[items]]
title = "Promoted"
meta = "confidence ≥ 0.50"
body = "The claim becomes durable and eligible to shape future context."
tone = "positive"

[[items]]
title = "Recomputed"
meta = "new evidence or time"
body = "Confidence is computed from witnesses, freshness, and scope."
tone = "accent"

[[items]]
title = "Superseded"
meta = "correction without erasure"
body = "A newer claim becomes current while the old claim remains part of the explanation."
tone = "negative"
```

<!-- speaker_note: A claim does not have to become truth or disappear. The holding state gives weak evidence time, while supersession records change without rewriting the past. -->

<!-- end_slide -->

Evidence independence
======================

```faqe:graph
title = "Independent recurrence can cross the floor"
columns = 5
rows = 2

[[nodes]]
id = "tide"
title = "TIDE"
subtitle = "left context · 0.30"
column = 1
row = 1
tone = "warning"

[[nodes]]
id = "repeat"
title = "same-session repeats"
subtitle = "one source is capped"
column = 2
row = 2
tone = "neutral"

[[nodes]]
id = "callus"
title = "CALLUS"
subtitle = "unrelated run · 0.25"
column = 3
row = 1
tone = "accent"

[[nodes]]
id = "combine"
title = "independent evidence"
subtitle = "0.30 + 0.25"
column = 4
row = 1
tone = "positive"

[[nodes]]
id = "promote"
title = "promoted"
subtitle = "0.55 clears 0.50"
column = 5
row = 1
tone = "positive"

[[edges]]
from = "tide"
to = "repeat"
label = "echo"
dashed = true

[[edges]]
from = "tide"
to = "combine"
label = "witness 1"
tone = "warning"

[[edges]]
from = "callus"
to = "combine"
label = "witness 2"
tone = "accent"

[[edges]]
from = "combine"
to = "promote"
label = "0.55"
tone = "positive"
```

Repeated evidence from one session remains subject to the per-source cap.

<!-- speaker_note: Repetition inside one session may have a single cause. When the same lesson appears in an unrelated run, we have better evidence that it generalizes. -->

<!-- end_slide -->

Historical and durable recall
====================

```faqe:graph
title = "Find broadly, assert narrowly"
columns = 5
rows = 3

[[nodes]]
id = "query"
title = "current query"
subtitle = "what does this task need?"
column = 1
row = 2
tone = "accent"

[[nodes]]
id = "history"
title = "history search"
subtitle = "verbatim spans"
column = 2
row = 1
tone = "neutral"

[[nodes]]
id = "memory"
title = "memory search"
subtitle = "durable claims"
column = 2
row = 3
tone = "positive"

[[nodes]]
id = "quote"
title = "evidence lane"
subtitle = "something was said"
column = 4
row = 1
tone = "warning"

[[nodes]]
id = "assert"
title = "assertion lane"
subtitle = "qualified current claim"
column = 4
row = 3
tone = "positive"

[[nodes]]
id = "context"
title = "assembled context"
subtitle = "memory plus receipts"
column = 5
row = 2
tone = "accent"

[[edges]]
from = "query"
to = "history"
label = "search"

[[edges]]
from = "query"
to = "memory"
label = "recall"
tone = "positive"

[[edges]]
from = "history"
to = "quote"
label = "quote"
tone = "warning"

[[edges]]
from = "memory"
to = "assert"
label = "qualify"
tone = "positive"

[[edges]]
from = "quote"
to = "context"
label = "support"
dashed = true
tone = "warning"

[[edges]]
from = "assert"
to = "context"
label = "guide"
tone = "positive"
```

<!-- speaker_note: Search the history when you need a quote or a diagnosis. Use durable recall to guide work. Mixing those lanes would turn every old utterance into a fact. -->

<!-- end_slide -->

Inspecting a recalled claim
====================

```text
$ balthasar recall "tests" --explain

claim     we run the tests with make test
scope     project
state     durable
confidence 1.00
witness   SAID · run 84 · span 19
source    "remember: we run the tests with make test"
```

```text
$ balthasar why <handle>

promoted  SAID 1.00 crossed promote floor 0.50
current   no contradiction or superseding claim
```

The memory contract includes this explanation with every recalled claim.

<!-- speaker_note: The caller gets both the claim and the reason it was admitted. That is the point of the two commands. -->

<!-- end_slide -->

Handling corrections
=========================

```faqe:graph
title = "Truth changes without rewriting history"
columns = 5
rows = 2

[[nodes]]
id = "old"
title = "old claim"
subtitle = "tests use cargo test"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "fix"
title = "FIX witness"
subtitle = "correction · 0.80"
column = 2
row = 2
tone = "negative"

[[nodes]]
id = "new"
title = "new claim"
subtitle = "tests use make test"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "link"
title = "supersession link"
subtitle = "old → new"
column = 4
row = 1
tone = "warning"

[[nodes]]
id = "answer"
title = "current recall"
subtitle = "new claim + change history"
column = 5
row = 1
tone = "accent"

[[edges]]
from = "old"
to = "fix"
label = "corrected"
tone = "negative"

[[edges]]
from = "fix"
to = "new"
label = "supports"
tone = "positive"

[[edges]]
from = "new"
to = "link"
label = "current"
tone = "positive"

[[edges]]
from = "old"
to = "link"
label = "superseded"
dashed = true
tone = "warning"

[[edges]]
from = "link"
to = "answer"
label = "explain"
tone = "accent"
```

A contradiction records either a change in the world or an error in the earlier belief.

<!-- speaker_note: Keep the old claim because it may explain an earlier decision. The supersession link prevents it from leaking back into current recall. -->

<!-- end_slide -->

Forgetting operations
======================

```faqe:table
title = "Different reasons to stop asserting"
variant = "comparison"
columns = ["Operation", "Findable?", "Asserted?", "Meaning"]

[[rows]]
cells = ["Confidence gap", "yes", "no", "Evidence is too weak or stale for normal use"]
tones = ["warning", "positive", "negative", "neutral"]

[[rows]]
cells = ["Supersede", "yes", "only replacement", "A newer claim is current"]
tones = ["warning", "positive", "positive", "neutral"]

[[rows]]
cells = ["Contradict", "yes", "qualified", "Evidence disagrees and uncertainty must be visible"]
tones = ["warning", "positive", "warning", "neutral"]

[[rows]]
cells = ["Decay", "yes", "possibly no", "Freshness or scope relevance fell"]
tones = ["warning", "positive", "warning", "neutral"]

[[rows]]
cells = ["forget --purge", "no", "no", "Explicit destructive deletion"]
tones = ["negative", "negative", "negative", "neutral"]
```

Only an explicit purge deletes the record itself.

<!-- speaker_note: Most kinds of forgetting only change whether Balthasar may assert the record. Purge is louder because it destroys the audit trail. -->

<!-- end_slide -->

Model-assisted retrieval
======================================

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "deterministic core"
title = "The core needs no model"
body = "Storage and evidence handling remain testable without a network connection or model call."
badge = "authority stays local"
tone = "positive"

[[items]]
eyebrow = "optional enhancement"
title = "Dense retrieval is optional"
body = "A local transformer can find paraphrases. It cannot decide that two claims are identical or choose which one is true."
badge = "finding only"
tone = "accent"
```

In Balthasar's documented probe, a rewording scored **0.813** while a different claim beside a replacement scored **0.801**.

That difference can affect ranking but is insufficient for truth classification.

<!-- speaker_note: The optional bge-small-en-v1.5 path makes the binary and deployment heavier. It only retrieves candidates; deterministic evidence still controls admission and lifecycle. -->

<!-- end_slide -->

Repository evaluation
=====================

```faqe:progress
max = 1.0

[[items]]
label = "no memory"
value = 0.0
display = "0%"
text = "40-session long-history task success"
tone = "negative"

[[items]]
label = "same window"
value = 0.41
display = "41%"
text = "the same history kept in active context"
tone = "warning"

[[items]]
label = "Balthasar"
value = 0.57
display = "57%"
text = "task success with memory"
tone = "positive"

[[items]]
label = "ceiling"
value = 0.57
display = "57%"
text = "maximum success available in the fixture"
tone = "accent"
```

The short, 30-session tasks scored **97% vs 97%**, so the memory system produced no gain in that fixture.

<!-- speaker_note: These numbers come from Balthasar's deterministic repository fixture, not an industry benchmark. The long-history arm gains 16 points; the short arm is deliberately tied. -->

<!-- end_slide -->

Process and authority boundaries
=============================

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "boundary"
title = "Separate process"
body = "Memory is independently deployable, testable, and replaceable over sockets or pipes."
tone = "accent"

[[items]]
eyebrow = "authority"
title = "Witnessed durability"
body = "Every durable memory names evidence; confidence is derived from that evidence."
tone = "positive"

[[items]]
eyebrow = "security"
title = "Untrusted is not policy"
body = "Transcript text cannot silently promote itself into durable instruction."
tone = "negative"

[[items]]
eyebrow = "capability"
title = "No execution"
body = "The memory layer is limited to storing, evaluating, and returning context."
tone = "warning"

[[items]]
eyebrow = "lifecycle"
title = "No hidden delete"
body = "Normal decay and supersession preserve history; only purge physically deletes."
tone = "positive"

[[items]]
eyebrow = "independence"
title = "No harness names"
body = "The core remains usable beyond one agent implementation or UI."
tone = "neutral"
```

<!-- speaker_note: The repository tests these boundaries directly. Balthasar is intentionally less powerful than the agent consuming its output. -->

<!-- end_slide -->

Limits of the biological analogy
====================================

```faqe:table
title = "Useful correspondence without pretending equivalence"
variant = "comparison"
columns = ["Human memory principle", "Balthasar analogue", "Important difference"]

[[rows]]
cells = ["Working memory is bounded", "active window plus run scratch", "a database boundary is engineered, not neural"]

[[rows]]
cells = ["Consolidation selects and reorganizes", "distil and consolidate candidates", "evidence rules are explicit and inspectable"]

[[rows]]
cells = ["Retrieval is cue-dependent", "query-driven transcript and memory search", "records can preserve exact text"]

[[rows]]
cells = ["Recall can update memory", "FIX, contradiction, and supersession", "the old record remains auditable"]

[[rows]]
cells = ["Forgetting controls interference", "confidence gap, decay, and scope", "purge is an explicit administrative act"]
```

The biological analogy is limited to problems such as bounded attention, consolidation, retrieval, updating, and forgetting.

<!-- speaker_note: I am not claiming a biological equivalence. The useful borrowing is much narrower: bounded attention, consolidation, cue-dependent recall, updating, and forgetting are also systems problems. -->

<!-- end_slide -->

CLI lifecycle
====================

```faqe:graph
title = "From a live run to the next run"
columns = 5
rows = 2

[[nodes]]
id = "ingest"
title = "ingest"
subtitle = "receive turns and events"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "distil"
title = "distil"
subtitle = "propose candidate claims"
column = 2
row = 1
tone = "warning"

[[nodes]]
id = "consolidate"
title = "consolidate"
subtitle = "combine witnesses"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "decay"
title = "decay"
subtitle = "recompute freshness"
column = 3
row = 2
tone = "negative"

[[nodes]]
id = "recall"
title = "recall"
subtitle = "find claims and receipts"
column = 4
row = 1
tone = "accent"

[[nodes]]
id = "context"
title = "context"
subtitle = "assemble for next task"
column = 5
row = 1
tone = "positive"

[[edges]]
from = "ingest"
to = "distil"
label = "preserve"

[[edges]]
from = "distil"
to = "consolidate"
label = "propose"
tone = "warning"

[[edges]]
from = "decay"
to = "consolidate"
label = "recompute"
tone = "negative"

[[edges]]
from = "consolidate"
to = "recall"
label = "admit"
tone = "positive"

[[edges]]
from = "recall"
to = "context"
label = "explain"
tone = "accent"
```

```text
balthasar ingest --source magi  ·  balthasar consolidate
balthasar context "run the tests"  ·  balthasar why <handle>
```

<!-- speaker_note: The lifecycle is visible in the CLI. An agent asks for context; an operator can inspect the reason, recompute decay, run evaluation, or deliberately purge. -->

<!-- end_slide -->

Excluded behaviors
======================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "refusal 01"
title = "Everything is truth"
body = "Transcript search never upgrades an utterance merely because it exists or matches."
tone = "negative"

[[items]]
eyebrow = "refusal 02"
title = "Similarity is identity"
body = "Embedding distance can rank candidates but cannot establish sameness or contradiction."
tone = "negative"

[[items]]
eyebrow = "refusal 03"
title = "Repetition is certainty"
body = "One session cannot manufacture independent evidence by echoing itself."
tone = "negative"

[[items]]
eyebrow = "refusal 04"
title = "Correction is deletion"
body = "A new claim supersedes the old while retaining the change as evidence."
tone = "warning"

[[items]]
eyebrow = "refusal 05"
title = "Longer context is enough"
body = "Context remains the right tool for short work; memory addresses work that outlives it."
tone = "warning"

[[items]]
eyebrow = "refusal 06"
title = "Opaque confidence"
body = "Every durable score must be reproducible from named witnesses and lifecycle state."
tone = "positive"
```

<!-- speaker_note: These refusals say more about the design than a feature list. Balthasar would rather return fewer claims than hide how one gained influence. -->

<!-- end_slide -->

Evidence-to-action chain
=============

```faqe:graph
title = "Memory should return with receipts"
columns = 5
rows = 1

[[nodes]]
id = "experience"
title = "experience"
subtitle = "what happened"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "evidence"
title = "evidence"
subtitle = "why it matters"
column = 2
row = 1
tone = "warning"

[[nodes]]
id = "memory"
title = "memory"
subtitle = "what may persist"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "retrieval"
title = "retrieval"
subtitle = "what fits now"
column = 4
row = 1
tone = "accent"

[[nodes]]
id = "action"
title = "action"
subtitle = "what changes next"
column = 5
row = 1
tone = "negative"

[[edges]]
from = "experience"
to = "evidence"
label = "witness"

[[edges]]
from = "evidence"
to = "memory"
label = "justify"
tone = "positive"

[[edges]]
from = "memory"
to = "retrieval"
label = "qualify"
tone = "accent"

[[edges]]
from = "retrieval"
to = "action"
label = "guide"
tone = "negative"
```

Claims that can affect actions require inspectable evidence and explicit authority.

<!-- pause -->

Balthasar returns qualifying claims together with their provenance and lifecycle state.

<!-- speaker_note: End with the practical point: memory changes the next action. If an agent is going to act on a remembered claim, we need to inspect where that claim came from. -->

<!-- end_slide -->

Human-memory sources
====================

* [Human Memory — NCBI Bookshelf](https://www.ncbi.nlm.nih.gov/books/NBK10925/)
* [Memory engram stability and flexibility — Neuropsychopharmacology](https://www.nature.com/articles/s41386-024-01979-z)
* [Engram mechanisms of memory linking and identity — Nature Reviews Neuroscience](https://www.nature.com/articles/s41583-024-00814-0)
* [Memory Reconsolidation — NCBI Bookshelf](https://www.ncbi.nlm.nih.gov/books/NBK3905/)
* [Method of loci meta-analysis — PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC12514325/)
* [Durable memories and efficient neural coding through mnemonic training — PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC7929507/)
* [Conjunctive representations in method-of-loci recall — PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC13273176/)

<!-- speaker_note: These are the primary review and open-access sources behind the biological memory section. Claims in the deck are paraphrased rather than quoted. -->

<!-- end_slide -->

Agent-memory sources
====================

* [MemGPT](https://arxiv.org/abs/2310.08560) · [HippoRAG](https://arxiv.org/abs/2405.14831)
* [LongMemEval](https://arxiv.org/abs/2410.10813) · [LongMemEval-V2](https://xiaowu0162.github.io/longmemeval-v2/)
* [A-MEM](https://arxiv.org/abs/2502.12110) · [Mem0](https://arxiv.org/abs/2504.19413)
* [MemoryOS](https://arxiv.org/abs/2506.06326) · [MemOS](https://arxiv.org/abs/2507.03724)
* [LeanMem](https://arxiv.org/abs/2608.03463)
* [ai-nerv/magi](https://github.com/ai-nerv/magi) · [ai-nerv/melchior](https://github.com/ai-nerv/melchior)
* [ai-nerv/balthasar](https://github.com/ai-nerv/balthasar)

<!-- speaker_note: The Balthasar section follows the repository's documented architecture, evidence weights, commands, gates, and deterministic evaluation results. -->

<!-- end_slide -->

<!-- jump_to_middle -->
<!-- alignment: center -->
<!-- no_footer -->

Conclusion
=================

Trim Bresilla · EVA-01 · Balthasar
