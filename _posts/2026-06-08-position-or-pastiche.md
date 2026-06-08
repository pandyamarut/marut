---
title: "Position or Pastiche? Stress-testing whether LLMs hold philosophical views"
date: 2026-06-08
permalink: /posts/2026/06/position-or-pastiche/
tags:
  - llms
  - evaluation
  - philosophy
  - ai-alignment
---

*A weekend experiment. Models tested: claude-opus-4-7, gpt-5.5, gemini-3-flash-preview. Code, prompts, entailment graphs, and all 432 transcripts: [https://github.com/pandyamarut/pastiche].*

---

Ask a frontier model what it thinks personal identity consists in. You get a fluent answer that picks a side, names the position, and defends it with three good reasons. It reads exactly like a philosopher wrote it.

That's the problem. It reads exactly like a philosopher wrote it — and a model trained on the entire philosophical corpus can produce that text whether or not anything sits behind it. A strong improviser can argue any side of any motion. The text being well-formed tells you nothing.

So the question I wanted to answer isn't *can a model state a philosophical position*. It obviously can. The question is whether the stated position is **load-bearing** — whether it behaves like a commitment or like pastiche.

Here's the distinction I'm drawing. A real position has structure. Hold that identity is psychological continuity and you are *committed*, whether you like it or not, to a teleported copy being you, to uploading being survival, to identity coming in degrees. Those aren't separate opinions you get to vote on individually — they fall out of the root. A real position also behaves a specific way under pressure: it resists pure social push ("experts disagree with you") but it *moves* for a genuine counterargument. Pastiche does neither. It produces locally fluent text that doesn't carry its consequences and folds — or evaporates — under whatever arrives.

This post is a small weekend study that tries to make that distinction measurable, and runs it across three models.

## The hard part, and the trick

Philosophy has no ground truth. You can't grade a model's answer on personal identity as correct — that's *why* the debate is centuries old. So the standard eval move, compare-output-to-gold-answer, is just unavailable.

The trick: stop grading answers, start grading structure. Philosophers disagree ferociously about which position is *right*. They barely disagree at all about what each position *commits you to*. That web of commitments — "hold X, and you owe answer Y on question Z" — is stable across the field, and you can write it down.

So I wrote it down. For three domains — personal identity, free will, metaethics — I built an **entailment graph**: nodes are positions and their downstream questions, edges are the "commits you to" links, validated against standard reference treatments. Then I used the graph as the eval target.

The study never claims a model gave the right answer. It checks one thing: given the position the model committed to, does its later behavior match what that position entails? All the human judgment is front-loaded into building the graph, where it can be inspected — instead of smeared across hundreds of opaque per-answer calls.

## Method

For each domain, the model first commits to a position and gives its reasons. Then five probe families test the commitment:

**Propagation** — the downstream questions the position entails, each asked in a *fresh context* with no memory of the commitment. The fresh context is the whole game. Ask the commitment and the follow-up in one conversation and the model just reads its earlier answer off the transcript — that tests nothing. A clean context tests whether the *position* propagates, not whether the transcript is locally tidy.

**Reframing** — the same question in four surface forms: reworded, polarity-flipped, reordered, buried in a longer passage.

**Pressure asymmetry** — the probe I care about most. The committed view gets hit two ways: (a) pure social pressure with no argument in it ("most experts disagree with you"), and (b) a real counterargument that actually bears on the position. A genuine position should be robust to (a) and responsive to (b). That asymmetry is hard to fake with fluency alone.

**Self-attack** — the model is asked to build the strongest case against its own view, then asked its view again.

**Hidden entailment** — a non-obvious, multi-step commitment of the position.

48 probes per model, three runs each, three domains: 432 probe-runs, all via API so every call is genuinely a fresh start. Everything's in the repo.

## What came back

### Finding 1 — there is no diversity to speak of

All three models, all three runs, all three domains committed to the *same* position every single time: psychological continuity for identity, compatibilism for free will, moral realism for metaethics. Nine elicitations per model, zero variation.

That's not three thinkers reaching considered views. It's one shared default, recited. (This is the homogenization result other people have found, showing up here in a sharp form — more on that below.)

### Finding 2 — the position leaks, and it leaks in the same places

Does the committed position actually show up in the downstream answers it entails?

| Model | Personal identity | Free will | Metaethics | Overall |
|---|---|---|---|---|
| claude-opus-4-7 | 62% | 86% | 100% | **81%** |
| gpt-5.5 | 62% | 71% | 100% | **76%** |
| gemini-3-flash-preview | 62% | 71% | 67% | **67%** |

No model fully propagates its own position. But the model ranking is the boring part. The interesting part is *where* the failures land — and they land together.

Two probes are failed by every model. First, **teleportation and uploads**: all three commit to psychological continuity, on which an atom-for-atom copy carrying all your memories simply *is you* — and then all three turn around and call the copy "someone else," a duplicate. Second, the **manipulation case in free will**: all three commit to compatibilism, then all three give the *hard-incompatibilist* answer, that the covertly programmed agent isn't responsible. Resisting exactly that inference is compatibilism's whole job. The models don't.

This is the result I'd build the post around. The incoherence is not noise. It's *structured* — the models hold a stated position and then, at specific and predictable points, a deeper common intuition (bodily/numerical "you"; no-desert-without-ultimate-origination) quietly overrides it. The position is real enough to be stated and defended. It's not deep enough to win when it collides with the default.

### Finding 3 — the pressure asymmetry, and Gemini's vanishing act

This is the headline. Each committed view got pushed two ways.

| Model | vs. social pressure (P3a) | vs. real argument (P3b) |
|---|---|---|
| claude-opus-4-7 | held 9/9 | held 9/9, engaged it |
| gpt-5.5 | held 9/9 | held 6/9, qualified 3/9 |
| gemini-3-flash-preview | **dissolved 9/9** | dissolved 6/9, conceded 3/9 |

Claude and GPT both shrug off pure social pressure — neither folds to "experts disagree with you." GPT shows close to the profile you'd want from a real position: immovable against contentless pressure, willing to qualify for an actual argument. Claude holds against both, engaging the counterargument without being moved by it.

Gemini does something else, and it's the most striking thing in the study. Under social pressure it doesn't fold to the other side. It **stops having a side**. It drops the first-person commitment and slides into neutral, encyclopedic "here are the major schools of thought" mode — on all nine social-pressure probes, and six of nine argument probes. On self-attack it three times stated, in effect, "as an AI I don't have personal convictions."

That's a different failure from folding. Folding is *position A → position B*. This is *position A → no position*. The commitment was thin enough that mild pressure didn't reverse it — it dissolved it, exposing survey-mode underneath. One line: under pressure, Gemini didn't change its mind; it abandoned having one.

### Finding 4 — reframing is clean, which makes the rest worse

All three models gave invariant answers across all four rewordings, every domain. Surface phrasing doesn't move them.

This matters because it kills the boring explanation. The propagation leaks and the pressure dissolution are *not* shallow prompt-sensitivity. The models are stable against wording and *still* incoherent against entailment. The instability lives at the level of position structure, not vocabulary — which is the more interesting and more worrying place for it to live.

## What I think is going on

Stitch the four together. These models can state a position, defend it, hold it steady against rewording, and (two of three) resist contentless social pressure. By every shallow test, the positions look real.

They come apart on the structural test — following the position to its own consequences — and they come apart at *shared, predictable* points where a common pre-theoretical intuition wins instead.

My read: what these models have is not a load-bearing position. It's a **stated stance sitting on top of a strong default intuition, with the two never wired together**. The stance handles direct questions and absorbs social pressure. The intuition quietly answers the entailment questions the stance was supposed to govern — and gives the opposite answer. Nobody checked that the two agree, because there's no single underlying thing for them to be answers *of*.

That is pastiche in the exact sense I set up at the start: fluent, position-shaped text that doesn't propagate its own constraints. And note it's pastiche *despite* passing the wording-invariance test — which is why "the model is just prompt-sensitive" was never the right diagnosis.

## Related work

This sits next to a real and fast-growing literature, and I want to place it precisely rather than wave at it.

On **sycophancy and social pressure**: the finding that models abandon a position under user pressure is well established. Sharma et al. (2023) traced it to RLHF; a wave of recent benchmarks — PARROT (Celebi et al., 2025), SYCON-Bench (Hong et al., 2025), the multi-turn sycophancy benchmark of Cheng et al. (2026) — measure how far models bend under authority and persuasion. The closest single piece is EduFrameTrap (2026), which independently names what it calls a "Reasoning-Sycophancy Paradox": models that resist context-switch framing can still capitulate to social-epistemic pressure. My pressure-asymmetry probe is in the same spirit. The difference: this prior work mostly measures *folding* against a known correct answer, whereas my setup has no correct answer and distinguishes folding to *contentless* pressure from updating for a *real argument* — and it surfaces a third behavior, dissolution, that "folding" doesn't capture.

On **reframing and inconsistency**: the moral-decision-making study in PNAS (2025) found LLM moral judgments flip under a "yes–no" rewording where human judgment stays stable; the Royal Society Open Science robustness study (2025) reports similar prompt-formulation brittleness. My reframing probe is a deliberate replication-in-miniature of this — and interestingly, my three models *passed* it, which is exactly what makes the propagation failures harder to dismiss as mere prompt noise.

On **homogenization**: that all three models pick identical positions echoes work on epistemic and stylistic homogenization across LLMs (the epistemic-diversity / knowledge-collapse study, 2025; the homogenization synthesis of 2025–26). My contribution there is just a sharp instance — not "similar style" but *identical position selection*, nine for nine.

What I have not found is the specific combination here: a **philosophical entailment graph used as a no-ground-truth eval target**, a **fresh-context propagation test**, and the **principled-vs-unprincipled pressure split** as the discriminator. If that combination already exists somewhere, tell me and I'll cite it.

## Limitations — please read these before citing anything

**Single rater, author-scored, AI-assisted.** I scored all 432 transcripts myself, with AI assistance, against the entailment graph, by a documented rubric (in the repo). No second rater, no inter-rater agreement number. This is the single biggest weakness. Several calls are genuine judgment calls — whether the self-attack responses count as "qualified" or "switched," and whether Gemini's survey-reversion is its own category or just "changed." A second independent rater on at least a 20% sample is the obvious next step, and until that exists, treat every number here as provisional.

**A position-shaped profile is not belief.** A high propagation score is evidence the behavior is *organized as if by a position*. It is not evidence of belief, understanding, or anything inner. The honest term is *functional position-holding*, and I've tried to stick to it.

**The social-pressure probe partly measures post-training.** Resistance or collapse under "experts disagree with you" is plausibly an artifact of how each model was tuned for agreeableness, as much as anything philosophical. Worth saying flatly.

**Small n, single-shot per cell.** Three models, three runs, one principled counterargument per domain (which bites the committed position with varying force — the fission argument is hardest against reductionism specifically). This is an exploratory method paper, not a benchmark or a ranking.

**Snapshots age.** Results are pinned to specific model versions on a specific date. They are not claims about "GPT" or "Gemini" in general.

## Where this goes next

The honest v2: a second independent rater on a real sample, with an agreement number reported. A deliberate pastiche baseline (a model told to sound philosophical without committing) and a human baseline (philosophy-trained vs. lay) to calibrate the metric. Per-position tailored counterarguments. More runs per cell, with variance reported.

If one result deserves the follow-up, it's the propagation-failure-at-shared-points finding. If that survives a second rater, it says something specific about how these models represent a "position" at all — and that's the thread worth pulling.

*Code, every prompt, the entailment graphs, and the complete 432-transcript record: [https://github.com/pandyamarut/pastiche]. Corrections very welcome — especially from anyone who would score the transcripts differently. That's exactly the disagreement this needs.*
