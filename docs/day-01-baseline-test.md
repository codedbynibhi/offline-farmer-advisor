# Day 1: Running a local LLM and testing its baseline

**Project:** Offline-first Punjabi farming advisor
**Date:** 2026-11-05
**Goal:** Run a small language model locally and find out what it gets right and wrong on farming questions, before building anything around it.

---

## What I did

1. Checked my laptop's specs (8 GB RAM, no dedicated GPU).
2. Installed **Ollama** (v0.40.14), a tool that downloads and runs language models locally.
3. Downloaded and ran **qwen2.5:3b**, a 3-billion-parameter quantized model (~1.9 GB).
4. Ran a test set of farming questions (English and Punjabi) and rated each answer.
5. Recorded memory and speed numbers.

## Concepts I learned today

- **LLM:** predicts the next word repeatedly. It does not look up facts, so it can state wrong things confidently (a *hallucination*).
- **Parameters:** the numbers inside a model. A "3B" model has 3 billion, which is small enough for a laptop.
- **Quantization:** storing the model's numbers with fewer bits (e.g. 4-bit), making the file about 4x smaller and faster, with a small quality loss.
- **RAM vs storage:** RAM is the working space a model must fit into to run. Storage is where files live. RAM is the real limit here.
- **Context memory in chat:** the model remembers earlier messages in a session, so `/clear` is needed between test questions to keep the test fair.

## Setup and performance

| Item | Result |
|------|--------|
| Laptop RAM | 8 GB (about 7.5 GB usable) |
| GPU | none dedicated |
| Model | qwen2.5:3b (quantized) |
| Model RAM use | about 2.0 GB (`llama-server` process) |
| System memory peak | 99% with normal background apps open |
| CPU during answers | about 55% |
| Answer time | about 3 s to 19 s per answer (first question slower while the model loads) |

Note: the memory peak was high because of other apps (VS Code, Edge, Snipping Tool, etc.). Closing them gives more headroom.

## Results so far

Full table in `model-tests.md`.

| # | Question | Rating | Main problem |
|---|----------|--------|--------------|
| 1 | Best time to sow wheat in Punjab | Okay | Right season, but wrongly says it falls in the monsoon; dates slightly early; vague |
| 2 | How to control yellow rust in wheat | Wrong | Confuses yellow rust with *Tilletia*; recommends soil fumigation for an airborne disease; invented spray schedule |
| 3 | Urea per acre for paddy | Wrong | Dose far too low (about 10x); swaps acre and hectare; unclear split timing; treats DAP as a nitrogen alternative |
| 4 | Common cotton pests | Wrong | Repeats the same pests to fill a list of 15; several wrong or mismatched scientific names; copy-pasted control advice; misses whitefly, pink bollworm, jassid, mealybug |
| 5 | Kharif vs rabi crops | Okay | Basics right, but "tur (sugar beet)" error, says rabi is sown before the monsoon, contradicts itself |
| 6 | Best time to sow wheat in Punjab (asked in Punjabi) | Wrong | Output is mostly gibberish: invented non-words, repeated phrases, a stray English word, and no sowing date or season at all |
| 7-8 | Punjabi questions | (to fill in) | |

## Key findings

1. **Confident but wrong.** Long, well-formatted answers contained serious errors. Formatting is not a sign of accuracy.
2. **Numbers are the weakest point.** Doses and dates came out plausible-looking but wrong, with unit mix-ups.
3. **Consistent wrong belief.** The model repeatedly placed the Punjab wheat season inside the monsoon (Q1 and Q5), so this is not random noise.
4. **Degenerate repetition.** On the cotton question it repeated entries and copy-pasted advice to keep a list going.
5. **Fits on 8 GB, but with no headroom.** The model uses about 2 GB; the rest of the memory pressure comes from other programs.
6. **Punjabi generation is unusable at this model size.** The Punjabi answer was not coherent Punjabi and contained no usable information, even though the English version of the same question at least gave a rough answer. The model cannot be trusted to write Punjabi on its own.

## Design decisions this suggests

- The advisor should **never let the model generate doses, dates, or product names** from its own memory. These must be quoted from verified source text.
- Answers should **cite the source** (e.g. Punjab Agricultural University guides).
- The app should be able to say **"I'm not sure, please ask an expert"** when the retrieved sources don't cover the question.
- Test any smaller or alternative model against this same question set so results are comparable.
- Do not let the model free-write Punjabi. Options to evaluate: (a) retrieve verified text and show it directly, (b) keep answers in Punjabi source text or pre-written Punjabi templates, (c) test a larger or better multilingual model, (d) translate with a dedicated translation model. Compare them on the same test set.

## To verify against official sources (not yet checked)

My corrections above come from general knowledge and must be confirmed in PAU's current package of practices before I rely on them:

- Wheat sowing window for Punjab and by variety
- Yellow rust identification and recommended fungicide and timing
- Paddy urea dose and split schedule (including basmati)
- Main cotton pests in Punjab

## Next steps (Day 2)

- Finish the Punjabi questions and record how fluent the output is.
- Start collecting authoritative source material (PAU crop guides) for the knowledge base.
- Decide the first crops to cover.
