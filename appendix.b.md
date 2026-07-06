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

### Final schema

| Field | Description |
|---|---|
| `subject` | TODO — paste from your final `DESCRIPTION_PROMPT` if you changed it from the example |
| `habitat_or_context` | TODO |
| `notable_objects` | TODO |
| `activity` | TODO |
| `subject_status` | TODO |
| *(add/remove rows to match your actual final schema)* | |

### Full prompt text

```text
TODO — paste your final DESCRIPTION_PROMPT verbatim, from Section 16 of the notebook
```

### Justification

> TODO — Why these fields specifically? What vocabulary choices did you make and why
> (constrained categories vs. free text, for which fields)?
>
> TODO — What did you try first and drop? What didn't work, and why?
>
> TODO — What does this schema make easy to retrieve? What does it make impossible —
> i.e. what visual properties of your corpus does it *not* capture? (Section 3's
> color/texture stress-test query is good raw material for this paragraph.)
>
> TODO — How does this connect back to your Level 1 corpus statement and your research
> question about live subjects vs. representations?

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
