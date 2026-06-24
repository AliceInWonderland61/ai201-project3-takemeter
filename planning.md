# TakeMeter — Planning Document

## Community

I chose r/nfrealmusic and NF's YouTube comment sections as my community. NF is a Christian rapper known for deeply personal lyrics about mental health, faith, and trauma, and his fanbase is one of the most engaged I've seen. People don't just say "good song" — they actually break down lyrics, share theories about music videos, and debate what his albums mean. That made it a good fit for a classification task because the discourse is genuinely varied. Some comments are thoughtful and specific, some are just bold opinions with no backup, and some are pure emotional reactions to new drops. You can tell the difference when you read them, which means a model should be able to learn it too.

## Labels

### analysis
A post that makes a substantive argument about NF's music by referencing specific lyrics, production choices, album structure, or comparisons. The reasoning is what carries the post, not just the opinion.

**Clear examples:**
- "The old man in the video is NF watching his younger self drown — the tattoo that says REAL on his hand confirms it's Nathan, showing how he became like his father."
- "The Search said 'cold world out there kids, grab your coats' and then Lost opens with 'self-awareness, pride's a coat and yes I like to wear it' — the connectivity between albums is everything."

**Uncertain example:**
- "I've always loved the guy, but ever since The Search his craft becomes better and better, Clouds mixtape was an absolute banger."
→ This references specific albums but makes no argument about them. Labeled **hot_take** because name-dropping albums without explaining why or how doesn't count as analysis.

---

### hot_take
A bold or confident opinion about NF or his music stated without supporting evidence. The post asserts something but doesn't back it up.

**Clear examples:**
- "He sounds like every other white YouTube rapper tryna be like Eminem with them fast flows that sound the same."
- "NF is like high school talent show rap."

**Uncertain example:**
- "He's just so corny and boring to listen to, only ever heard his songs because my brother would play it on the radio."
→ Casual and low-effort, but still expresses a clear opinion about NF's music. Labeled **hot_take** because any post with a clear opinion and no supporting evidence qualifies, regardless of how intentional or strong it sounds.

---

### reaction
An immediate emotional response triggered by a specific real-world event like a new release or announcement. Little to no argument — the post is expressing a feeling in the moment.

**Clear examples:**
- "We just got an EP and people already talking about the next release 😭🙏"
- "HE JUST DROPPED I WASN'T READY FOR THIS"

**Uncertain example:**
- "booooooo 🍅🍅🍅🍅"
→ Has reactive energy but no context about what triggered it. Too short and contextless to classify reliably. Posts like this were excluded from the dataset.

---

## Hard Edge Cases and Decision Rules

**hot_take vs. reaction:**
If a post expresses an opinion that could have been written any day regardless of what just happened, it's a hot_take. If it only makes sense as a response to something that just dropped or was just announced, it's a reaction.

**hot_take vs. analysis:**
Name-dropping albums or songs without explaining why or how doesn't count as analysis. A post needs specific reasoning — lyrics, structure, comparisons with substance — to qualify as analysis. Tone alone doesn't determine the label; reasoning does.

**Too short to label:**
Posts with no context or substance (e.g. "One of the best." or "booooooo") were excluded from the dataset entirely.

---

## Data Collection Plan

I collected examples mainly from NF's YouTube comment sections across multiple music videos including Let You Down, Leave Me Alone, HAPPY, Lost, and MOTTO. I also pulled some comments from r/nfrealmusic on Reddit for variety. I collected manually by copy-pasting comments into a spreadsheet and labeling them one by one, with some batches pre-labeled using Claude and reviewed by me before adding them.

I aimed for roughly 70 examples per label to keep things balanced. My final distribution was:
- hot_take: 71
- reaction: 70
- analysis: 68
- **Total: 209**

If any label had fallen below 20% of the dataset I would have gone back and collected more examples specifically for that label before moving on.

---

## Evaluation Metrics

I used overall accuracy to get a general sense of how the model is doing, but accuracy alone isn't enough here because the labels are balanced — a model that just guessed "analysis" every time would get around 33% which sounds okay but is actually useless.

The more important metrics are per-class precision, recall, and F1:
- **Precision** tells me if the model is being too aggressive about predicting a label
- **Recall** tells me if the model is missing examples of a label
- **F1** balances both and gives me the most useful single number per class

I also used a confusion matrix to see exactly which labels are getting confused with each other, since that tells me something specific about where the decision boundary is breaking down.

---

## Definition of Success

I'd consider this classifier genuinely useful if it hits at least 70% overall accuracy with no single label falling below an F1 of 0.60. That would mean the model is learning all three distinctions well enough to be useful in a real community tool. Although it's not perfect, it is consistent enough that you could trust its outputs most of the time.

The baseline I needed to beat was random chance (33%) and ideally I wanted to beat the zero-shot Groq baseline too. If the fine-tuned model can't beat a zero-shot LLM prompt, that tells me the training data isn't giving the model enough signal to learn from.

---

## AI Tool Plan

**Label stress-testing:**
Before annotating 200 examples I worked through edge cases with Claude to stress-test my label definitions. I gave Claude example posts and asked it to classify them, then used the disagreements to sharpen my decision rules, especially the hot_take vs. reaction boundary and the question of whether name-dropping albums counts as analysis (it doesn't).

**Annotation assistance:**
I used Claude to pre-label batches of comments during data collection. I would paste 15-20 comments at a time and Claude would assign labels with reasoning. I then reviewed every single label myself and corrected ones I disagreed with before adding them to my spreadsheet. This sped up the process significantly while keeping me close to the data.

**Failure analysis:**
After running the fine-tuned model I pasted the wrong predictions into Claude and asked it to identify patterns. The main pattern it surfaced — that everything was getting pulled toward analysis — matched what I saw in the confusion matrix. I then verified this myself by re-reading the misclassified examples.