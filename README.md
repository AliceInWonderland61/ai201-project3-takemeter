# TakeMeter — NF Community Discourse Classifier

A fine-tuned text classifier that categorizes comments from NF's YouTube videos and r/nfrealmusic into three types of discourse: analysis, hot_take, and reaction.

---

## Community Choice

I chose NF's YouTube comment sections and r/nfrealmusic as my community. NF is a Christian rapper known for deeply personal lyrics about mental health, faith, and trauma, and his fanbase is genuinely one of the most engaged I've come across. People don't just drop "good song" and leave — they break down lyrics, share theories about music video symbolism, and react hard to new drops. That made it a great fit for a classification task because the discourse is actually varied in a meaningful way. Some comments are substantive and specific, some are bold opinions with no backup, and some are pure emotional reactions to new releases. The distinctions matter to people in that community, and they're real enough that a model should be able to learn them.

---

## Label Taxonomy

### analysis
A post that makes a substantive argument about NF's music by referencing specific lyrics, production choices, album structure, or comparisons. The reasoning is what carries the post, not just the opinion.

**Examples:**
- "The old man in the video is NF watching his younger self drown — the tattoo that says REAL on his hand confirms it's Nathan, showing how he became like his father."
- "The Search said 'cold world out there kids, grab your coats' and then Lost opens with 'self-awareness, pride's a coat and yes I like to wear it' — the connectivity between albums is everything."

### hot_take
A bold or confident opinion about NF or his music stated without supporting evidence. The post asserts something but doesn't back it up.

**Examples:**
- "He sounds like every other white YouTube rapper tryna be like Eminem with them fast flows that sound the same."
- "NF is like high school talent show rap."

### reaction
An immediate emotional response triggered by a specific real-world event like a new release or announcement. Little to no argument — the post is expressing a feeling in the moment.

**Examples:**
- "We just got an EP and people already talking about the next release 😭🙏"
- "HE JUST DROPPED I WASN'T READY FOR THIS"

---

## Data Collection

**Sources:** Mainly NF's YouTube comment sections (Let You Down, Leave Me Alone, HAPPY, Lost, MOTTO) with some comments from r/nfrealmusic on Reddit for variety.

**Process:** I collected manually by copy-pasting comments into a spreadsheet. I used an AI assistant to pre-label batches of 15-20 comments at a time based on my label definitions, then reviewed and corrected every label myself before adding them to the dataset. I skipped anything too short to classify or without enough context.

**Label distribution:**
| Label | Count |
|-------|-------|
| hot_take | 71 |
| reaction | 70 |
| analysis | 68 |
| **Total** | **209** |

**Three difficult-to-label examples:**

1. *"Holding the gas can, asking God if he started the fire... that hit hard man, it's hard to take accountability sometimes"* — This quotes a specific lyric (reaction signal) but then makes a general reflective claim about life (hot_take signal). Labeled **hot_take** because the second half is making a general assertion, not reacting to a specific event.

2. *"I've always loved the guy, but ever since The Search his craft becomes better and better, Clouds mixtape was an absolute banger"* — References specific albums which sounds like analysis, but there's no argument about them. Labeled **hot_take** because name-dropping albums without explaining why doesn't count as analysis.

3. *"Its definitely different than his usual style but its so fire ngl"* — Could be a hot_take (opinion) or a reaction (triggered by a new release). Labeled **reaction** because the "different than his usual style" framing implies something just dropped and the person is responding to it in the moment.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased`

**Training setup:**
- 3 epochs
- Learning rate: 2e-5
- Batch size: 16
- Train/val/test split: 70% / 15% / 15% (146 / 31 / 32 examples)

**Hyperparameter decision:** I kept the default 3 epochs rather than increasing them because with only 146 training examples, more epochs would risk overfitting. The validation accuracy was still improving at epoch 3 (32% → 42% → 64%) so the training was moving in the right direction without overfitting signs.

---

## Baseline

**Model:** Groq's `llama-3.3-70b-versatile` with zero-shot prompting

**Prompt used:**
```
You are classifying comments from r/nfrealmusic and NF YouTube videos.
Assign each post to exactly one of the following categories.

analysis: The post makes a substantive argument about NF's music by referencing 
specific lyrics, production choices, album structure, or comparisons. The reasoning 
is what carries the post, not just the opinion.
Example: "The old man in the video is NF watching his younger self drown — the tattoo 
that says REAL on his hand confirms it's Nathan, showing how he became like his father."

hot_take: A bold or confident opinion about NF or his music stated without supporting 
evidence. The post asserts something but doesn't back it up.
Example: "NF is like high school talent show rap."

reaction: An immediate emotional response triggered by a specific real-world event 
like a new release or announcement. Little to no argument — the post is expressing 
a feeling in the moment.
Example: "HE JUST DROPPED I WASN'T READY FOR THIS"

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:
analysis
hot_take
reaction
```

All 32 test examples were parseable (100% parse rate).

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq) | 0.750 |
| Fine-tuned DistilBERT | 0.562 |

The baseline outperformed the fine-tuned model by 18.8 percentage points. This is a meaningful finding — it tells me that a large LLM with a well-written prompt can outperform a small fine-tuned model on a subjective task with only 146 training examples.

---

### Per-Class Metrics

**Fine-tuned model:**
| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.50 | 1.00 | 0.67 | 10 |
| hot_take | 0.67 | 0.18 | 0.29 | 11 |
| reaction | 0.67 | 0.55 | 0.60 | 11 |
| **accuracy** | | | **0.56** | 32 |

**Baseline (Groq):**
| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.71 | 1.00 | 0.83 | 10 |
| hot_take | 1.00 | 0.55 | 0.71 | 11 |
| reaction | 0.67 | 0.73 | 0.70 | 11 |
| **accuracy** | | | **0.75** | 32 |

---

### Confusion Matrix (Fine-tuned Model)

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|--|--|--|--|
| **True: analysis** | 10 | 0 | 0 |
| **True: hot_take** | 6 | 2 | 3 |
| **True: reaction** | 4 | 1 | 6 |

The diagonal shows correct predictions. The most obvious pattern is that analysis is being over-predicted — 6 hot_takes and 4 reactions got classified as analysis instead.

---

### Wrong Prediction Analysis

Before writing this section I pasted all 14 wrong predictions into an AI assistant and asked it to identify common patterns across the errors. It surfaced three things: (1) most errors were predictions of analysis when the true label was something else, (2) confidence scores were uniformly low on wrong predictions (all around 0.34-0.36), and (3) posts with specific references to lyrics or video scenes were disproportionately misclassified as analysis. I verified all three patterns myself by re-reading the examples and they all held up. The one thing I pushed back on was the AI framing it as a "length problem" — I don't think it's about length, it's about specificity. Posts that mention specific details get pulled toward analysis regardless of whether they're actually making an argument.

**#1 — "NF is the literal definition of UNDERRATED" (true: hot_take, predicted: analysis, confidence: 0.34)**

This is a textbook hot_take — bold claim, no evidence, nothing specific referenced. The model predicted analysis with very low confidence, which suggests it had no idea what to do with it. Short declarative hot_takes without obvious emotional markers probably weren't well represented enough in the training data, so the model defaulted to analysis. To fix this I'd need more examples of short punchy hot_takes that don't reference any specific content.

**#2 — "Old man: 'I'm sorry that I let you down.' Also Old man: Proceeds to let a man drown" (true: reaction, predicted: analysis, confidence: 0.36)**

This is a humorous observation about the Let You Down music video — someone pointing out the irony. It's a reaction to something specific in the video, not a structured argument. But the model saw a reference to a specific visual element and treated it like analysis. This reveals the model learned "mentions specific video content = analysis" which isn't quite right — reactions can reference specific moments too. The boundary here is whether the post is making an argument or just observing something funny.

**#3 — "I really love NF's music. He doesn't curse, he doesn't talk about inappropriate things and stuff like other rappers. He hits home every time. It's so meaningful." (true: hot_take, predicted: analysis, confidence: 0.35)**

This is a longer hot_take that compares NF to other rappers. The model got confused by the length and the comparative framing — longer posts with comparisons tend to be analysis in the training data. But there's no actual argument here, just a series of assertions. This is the hardest boundary in the dataset: detailed hot_takes that use comparative language look structurally similar to weak analysis. To fix this I'd need more training examples that are long and specific but still asserting rather than reasoning.

---

### Sample Classifications

| Text | Predicted Label | Confidence | Correct? |
|------|----------------|------------|----------|
| "The old man is NF watching his younger self drown — his fear is the only thing that has been there, burying it isn't as great as he thought" | analysis | 0.71 | ✅ |
| "NF is like high school talent show rap" | hot_take | 0.68 | ✅ |
| "WHO IS HERE AFTER NF DROPPED HOPE" | reaction | 0.74 | ✅ |
| "NF is the literal definition of UNDERRATED" | analysis | 0.34 | ❌ |
| "This song made me cry" | reaction | 0.65 | ✅ |

The first example is a reasonable correct prediction — it references a specific visual from the music video (the drowning scene) and makes an interpretive claim about what the symbolism means. That's exactly what analysis looks like in this community: specific reference plus an argument about what it means.

---

### Reflection: What the Model Learned vs What I Intended

What I intended was for the model to learn the difference between reasoning (analysis), assertion (hot_take), and in-the-moment emotion (reaction).

What the model actually learned was closer to: "if the post mentions something specific about the music or video, it's probably analysis." That's not wrong exactly, but it's too broad. Hot_takes can mention specific things too (like comparing NF to Eminem), and reactions can reference specific moments (like a lyric that hit hard). The model didn't learn the reasoning vs. assertion distinction — it learned a surface-level pattern about specificity and detail.

The low confidence scores on wrong predictions (all 0.34-0.36, barely above random chance for 3 classes) tell me the model was essentially guessing on the hard cases. With only 146 training examples it didn't have enough data to learn the subtle distinction between a detailed hot_take and a weak analysis. That boundary is genuinely hard — even I found it hard to label consistently during annotation.

The baseline did better because Llama 3.3 already understands language well enough that my label definitions gave it enough to work with. DistilBERT needed way more examples to pick up on those same cues from scratch.

---

## Spec Reflection

One way the spec helped me: the requirement to document 3 genuinely difficult-to-label examples before I started annotating forced me to think hard about my label boundaries early. The edge case between hot_take and analysis — specifically whether name-dropping albums counts as analysis — was something I worked out during planning, which made the actual annotation much more consistent.

One way my implementation diverged: the spec assumed I'd be pulling from one subreddit, but I ended up using YouTube comments as my main source because r/nfrealmusic didn't have enough varied content for 200 examples. YouTube gave me a much wider range of comment styles, which I think actually made the dataset better — more variety means more diverse signal for the model.

---

## AI Usage

**Instance 1 — Label stress-testing and edge case analysis:**
I used an AI assistant to stress-test my label definitions by running example posts through it and comparing its classifications to mine. When we disagreed I used that as a signal to sharpen my decision rules. For example, the "name-dropping albums" edge case came out of one of these disagreements — the AI initially labeled "Clouds mixtape was an absolute banger" as analysis while I labeled it hot_take. Working through why helped me write a clearer definition: name-dropping albums without explaining why or how doesn't count as analysis, you need actual reasoning. I kept all final labeling decisions myself and overrode the AI's classification in this case.

**Instance 2 — Batch pre-labeling during data collection:**
I used an AI assistant to pre-label batches of 15-20 YouTube comments based on my label definitions. It would produce a table with labels and brief reasoning for each comment. I then reviewed every single label and corrected ones I disagreed with — I changed roughly 15-20% of them, mostly cases where the AI labeled something as analysis because it mentioned specific content, when I felt it was actually a hot_take or reaction. All corrections were my own judgment.