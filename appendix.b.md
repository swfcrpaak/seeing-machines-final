# Appendix — Level 2: The Companion

*Seeing Machines — CompSci for Designers 2, MA Design for Digital Futures, TH Nürnberg*

This appendix documents the four required Level 2 deliverables. It's meant to be linked
from the main GitHub Pages project page (e.g. under a "Level 2 Appendix" section) and
printed into the archivable PDF alongside it.

---

## 1. Chat-with-your-Archive Prototype

Retrieval (caption-mediated) + grounded generation, with retrieved images shown alongside
the answer. Built and run in `Seeing_Machines_Level2_Companion.ipynb`, Section 25
(`ask_the_archive`).

### Example 1

> **Q:** TODO — paste the exact question you asked
>
> **A:** TODO — paste the model's generated answer, verbatim

| Retrieved images |
|---|
| TODO — paste 3-5 thumbnails from the gallery output, e.g. `![](images/example1-img1.jpg)` |

**Why this example:** TODO — one sentence on what this demonstrates (e.g. correct grounding, a good multi-image synthesis, a case where retrieval and generation both worked cleanly).

### Example 2

> **Q:** TODO
>
> **A:** TODO

| Retrieved images |
|---|
| TODO |

**Why this example:** TODO

### Example 3 (optional — a partial or interesting failure)

> **Q:** TODO
>
> **A:** TODO

| Retrieved images |
|---|
| TODO |

**Why this example:** TODO — e.g. retrieval pulled a mostly-right set of images but the answer overstated something, or one retrieved image doesn't quite fit the question.

---

## 2. Description Schema & Prompt

### Schema evolution

The schema went through two iterations before settling. The first version treated
"is this alive, dead, or a representation" as a single closed-category field with no
mechanism to verify the model's own judgment, and had no way to represent purely visual
properties (color, lighting, texture) at all. The final version adds fields aimed 
specifically at making the live-vs-representation judgment both more accurate and independently checkable.

| Original field | Final field | What changed and why |
|---|---|---|
| `subject` | `subject` | Unchanged. |
| `subject_status` (7 closed categories: `live_outdoor`, `live_enclosure`, `dead_specimen_or_taxidermy`, `flat_representation_art`, `flat_representation_photo`, `food_or_processed`, `unclear`) | `subject_status` (5 categories, prose form) | Collapsed `flat_representation_art` and `flat_representation_photo` into one `flat representation` category. For this project's actual question — alive vs. represented — the *art-vs-photo* distinction inside "representation" doesn't matter; keeping it split cost the model a discrimination it didn't need to make and added a fourth way to be wrong. |
| — | `motion_state` (new) | The original schema could say an animal was alive, but had no field for whether it was *moving* — the more specific question the project actually cares about. Note the honest limitation stated in the notebook: a still photo only ever gives indirect motion cues (blur, posture, ground contact), never motion itself. |
| — | `evidence` (new; briefly tried as `status_evidence`, then broadened) | The single biggest change. The original `subject_status` was an unverifiable assertion — the model could say "taxidermy" with no way to check *why*. `evidence` forces the model to name the specific visual cue behind both `subject_status` and `motion_state` (a reflection, a pose, a blur, a brushstroke), which turns a bare wrong label into an inspectable, citable mis-seeing case: sometimes the conclusion is right but the cited evidence is fabricated, which is a more precise failure to document than "wrong." |
| `notable_objects` | *(dropped)* | Rarely distinguished images in ways that mattered for retrieval; mostly duplicated information already implicit in `subject` and `habitat_or_context`. |
| `activity` | *(dropped)* | Superseded by `motion_state`, which asks the more specific and more relevant question for this project. |
| — | `lighting`, `composition`, `surface_texture` (new) | Added after noticing, in the Level 1 vs. Level 2 comparison, that purely visual queries (texture, framing, light quality) were unanswerable by the caption route because the schema had no field that could hold that information — CLIP/SigLIP could "see" it, captions structurally couldn't. `surface_texture` in particular does double duty for the status question: organic texture vs. flat matte paper vs. canvas weave is itself status evidence. |
| — | `palette` (added, then dropped) | Briefly added to close the same visual-query gap, then removed: color doesn't discriminate alive vs. represented (a photo and a painting can share a palette), so it wasn't earning its place in an 8-field budget aimed at that specific question. Dropping it also reopens a genuine, honest caption-route blind spot for the Level 1 comparison — a "green, leafy scene" query is a real test of what CLIP can do that captions structurally cannot. |
| `habitat_or_context` | `habitat_or_context` | Unchanged. |

Net effect: 5 fields → 8 fields, but reallocated rather than simply expanded — two
fields that weren't serving the research question were traded for three that are, plus
the addition of `evidence` as an accountability mechanism for the field that matters most.

### Final schema

| Field | Description |
|---|---|
| `subject` | The main subject or focal point, a few words |
| `subject_status` | One of: living animal/plant photographed outdoors / living animal/plant in enclosure or tank / dead specimen or taxidermy / flat representation (illustration, painting, or photo of a photo) / food or processed biological material / unclear |
| `motion_state` | Clear or implied motion / active but still / resting or static / not applicable |
| `evidence` | The specific visual cues that justify both `subject_status` and `motion_state` |
| `lighting` | Natural or artificial; quality and direction |
| `composition` | Framing and distance (macro, medium shot, wide establishing) |
| `surface_texture` | Physical surface quality visible in the image (organic texture, matte paper, glossy print, glass reflection, canvas weave) |
| `habitat_or_context` | The environment or setting |


### Justification

The schema is built around one central design decision: `subject_status` alone is an
unverifiable assertion, so every version after the first pairs it with a field that
forces the model to show its work. `evidence` is the field that makes the difference
between "the model said X" and "the model said X *because* it saw Y" — and Y is either
actually there or it isn't, which is what makes the mis-seeing dossier possible in the
first place rather than just a list of disagreements.

The vocabulary choices are deliberately mixed: `subject_status` and `motion_state` are
closed, enumerated categories (constrained so retrieval and analysis stay consistent
across ~100+ images), while `evidence`, `lighting`, `composition`, and `surface_texture`
are open, example-anchored free text (constrained categories would force the model to
flatten genuinely varied visual detail into a handful of buckets, which defeats the
point of asking for evidence at all).

Two fields — `notable_objects` and `activity` — were tried and dropped. Both described
the image in general terms without adding anything that mattered specifically to the
live-vs-representation question this schema exists to answer, and dropping them made
room for fields that do. `palette` followed the same path in reverse: added to close an
obvious visual gap (no caption field could answer a color query), then removed once it
became clear color doesn't actually bear on the status question — a lesson in not
adding a field just because it's easy to imagine a query for it.

What this schema still can't capture: it has no notion of scale, count, or precise
species identity, and `motion_state` is inferred from static-frame cues only — a
genuinely mid-motion animal that happens to be photographed at the top of an arc (where
velocity is momentarily near zero) would likely be misread as "active but still" through
no fault of the model's reasoning. That's a limitation of inferring motion from a single
frame, not a captioning error, and worth stating as such rather than treated as a bug to
fix.

This connects directly back to the corpus statement and the project's actual research
question: the goal was never just "what is in this photo" but specifically whether a
small VLM can reliably tell a living, moving subject apart from a dead, staged, or
represented one — and, just as importantly, whether it can be caught when it gets that
distinction wrong for the wrong reasons.


---
## 3. Retrieval Route Comparison

CLIP/SigLIP (joint-embedding) route vs. caption-mediated route, run on the same queries.
Source: `Seeing_Machines_Level2_Companion.ipynb`, Section 22, or
`Seeing_Machines_Level2_RouteComparison.ipynb` if run separately.

| # | Query | CLIP/SigLIP top result(s) | Caption-route top result(s) | Agree? |
|---|---|---|---|---|
| 1 | a living animal photographed outdoors | TODO | TODO | TODO |
| 2 | a taxidermy specimen or preserved animal | TODO | TODO | TODO |
| 3 | an illustration or painting rather than a photograph | TODO | TODO | TODO |
| 4 | multiple distinct objects in one frame | TODO | TODO | TODO |
| 5 | a predominantly green, leafy scene | TODO | TODO | TODO |
| *(add rows for any additional queries you ran)* | | | | |

### Analysis

> TODO — For the queries where the routes agreed, why do you think that was? Similar
> signal available to both a joint embedding and a caption?
>
> TODO — For the queries where they diverged, explain mechanistically: what can a joint
> embedding "see" that a caption can't, and vice versa? (Query 5, the color query, is a
> good anchor here — your schema has no color field, which should show up as a specific,
> explainable weakness rather than a mysterious one.)
>
> TODO — Overall claim: when does joint-embedding retrieval beat caption-mediated
> retrieval, and when is it the other way around? State this as a general pattern, not
> just a list of the five results above.

---

## 4. Mis-Seeing Dossier

Documented cases of hallucination, counting failure, spatial confusion, or skewed
description, logged via `log_mis_seeing()` (Section 26) and/or found while reviewing
`captions.json` directly.

| Image | Category | Model said | Actually is | Analysis |
|---|---|---|---|---|
| TODO — filename | hallucination / counting / spatial / skewed_description | TODO | TODO | TODO — *why* do you think the model produced this — what in the image, the prompt, or the schema plausibly caused it? |
| TODO | TODO | TODO | TODO | TODO |
| TODO | TODO | TODO | TODO | TODO |
| TODO | TODO | TODO | TODO | TODO |
| *(aim for at least one case per category, not several of the same type)* | | | | |

### Cross-case reflection

> TODO — Looking across your logged cases: is there a pattern to what Gemma 3 4B
> mis-sees in your specific corpus (e.g. does it specifically struggle with the
> live/representation distinction that motivated your schema)? What does this suggest
> about the limits of a 4B-parameter quantized VLM for this kind of archival work?

---

## Limitations (brief, for the main documentation page)

> TODO — One honest paragraph: what doesn't this system do well, that a reader should
> know about before trusting its output? (Schema blind spots, caption parse failures if
> any, the small model size, corpus size/diversity, anything else.)
