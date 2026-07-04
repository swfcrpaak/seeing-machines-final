# Seeing Machines – Life, Stillness, and My Archive

**[Min Jeong Park]**  
CompSci for Designers 2 · TH Nürnberg

![Teaser image](media/teaser.jpg)

*A small multimodal companion explores a personal archive of plants and animals, asking a deceptively simple question: what does it mean, for a machine, to “see life” in pictures?*

---

## Abstract

Human babies begin to sort the visual world into “living” and “non‑living” surprisingly early, a recent research found through brain‑imaging studies showing distinct responses to living beings versus inanimate objects by just a few months of age. This project turns that question toward a small vision–language system and a personal archive of about one hundred images of plants and animals, mixing live subjects and representation in the forms of: taxidermy, illustrations and paintings. The archive is explored through two routes: a Level 1 SigLIP search that embeds images directly, and a Level 2 caption‑based route where a VLM compresses each picture into a structured description, including a subject_status label such as “live_outdoor” or “flat_representation_art”, before text‑embedding and retrieval. By comparing where these compressed representations preserve or blur the line between life and stillness—especially in mis‑seeing cases where taxidermy or prints are treated as live—the project treats each retrieval path as a different form of “image compression as intelligence” and uses the machine’s mistakes as a way to look again at a familiar archive.

---

## Motivation and Corpus

I am interested in how very small systems, human or machine, begin to carve up the visual world. Infant research shows that babies do not just passively receive images; they gradually build categories such as “living” and “non‑living” long before they can talk about them. [web:211] This project translates that curiosity into a personal setting: how does a compact multimodal model see the same images I have collected, and where does its organising of the archive diverge from mine?

The corpus consists of 99 images:

- Original photographs of plants and animals taken by the author in temperate northern hemisphere climate and warm southern hempishere climate.
- Images of animals in enclosures, aquariums, and terrariums
- Taxidermy specimens and display cases (where allowed by rights)
- Public‑domain illustrations and paintings of animals and plants
  
All public images in the GitHub repository come from sources that mark the underlying works as public domain or CC0, documented in a `LICENSES.md` file. Private, rights‑sensitive images remain only in the local Colab corpus.

---

## Approach

The system closely follows the course’s Level 1 and Level 2 pattern. [file:2]

- **Level 1 – The Finder (SigLIP route)**  
  - SigLIP embeds all images once.  
  - Text queries are embedded into the same space.  
  - Cosine similarity returns the top‑k images in a Gradio interface.  
  - A retrieval atlas logs at least ten queries (e.g. “live animal in nature”, “vintage illustration of a plant”) with embedding‑space reasoning: why this query returned these images.

- **Level 2 – The Companion (caption route)**  
  - Gemma 3 4B (quantized) generates a structured JSON description for each image, using a custom schema tuned to this archive: subject, habitat_or_context, notable_objects, activity, and a categorical subject_status (live_outdoor, live_enclosure, dead_specimen_or_taxidermy, flat_representation_art, flat_representation_photo, food_or_processed, unclear). [file:2]  
  - Captions are flattened to text and embedded with `all-MiniLM-L6-v2`.  
  - A FAISS index powers caption‑based retrieval.  
  - A language model (Gemma in text‑only mode) generates grounded answers from the retrieved captions.  
  - A route toggle in the interface exposes both retrieval paths: caption route (Level 2) and SigLIP route (Level 1 baseline).

Throughout, the project treats each route as a different way of compressing the archive: SigLIP compresses images into dense vectors; Gemma compresses them into compact verbal summaries. The analysis asks which compression better preserves the life vs representation distinction and where both fail.

---

## Results Gallery and Retrieval Atlas

The results section collects annotated screenshots and image panels, including:

- **Successful retrievals**  
  - Queries where both routes find clear live animals outdoors or obvious vintage illustrations, with matching subject_status labels.
  - “close‑up of a live flower outdoors”

SigLIP is clearly locking onto “big pink flower shape + green background + outdoor lighting”, not on whether the flower is actually alive. That’s why it twice pulls printed flowers on a bus and a picture with a glass jar with nature in the background: visually they share saturated colours, petal/stem shapes, and centred composition. Medium (print vs real plant) seems much weaker in the embedding than basic colour and layout.

- **Disagreements between routes**  
  - Cases where SigLIP ranks live photos higher while the caption route prioritises illustrations, or vice versa.  
  - Examples where Gemma’s subject_status disagrees with the author’s judgement (e.g. taxidermy treated as “live_enclosure”).

- **Failure probes and binding tests**  
  - Paired queries such as “a brown fish on a blue background” versus “a blue fish on a brown background”, or “live animal in front of plants” versus “plants in front of a live animal”.  
  - Mixed results on “live versus illustrated” queries, where illustrations and photos cluster together.

  - Query No.1: “natural landscape with no live animals”
    Settings: top_k = 8, min_similarity = -0.05

    Observed results:
    <img width="1243" height="878" alt="Screenshot 2026-07-04 at 1 22 29 PM" src="https://github.com/user-attachments/assets/da8e45b2-  bd11-4f37-a8de-b77025efd381" />

    Embedding‑space reasoning:
This query is meant to test absences (no live animlas) over the "landscape" environment. Geese appears as the top image alongside pure landscapes. SigLIP finds a coherent “landscape” cluster—parks, trees, ponds—but animals and other creatures inside those scenes don’t push the images out of that region. Geese and painted animals stay near pure landscapes if the colours and scene layout match. Negation (“no live animals”) doesn’t move the text embedding very far from “natural landscape”, so the model still returns visually similar scenes even when they contain animals. This entry is a good example of granularity and negation issues: the model can find landscapes, but cannot prune away subjects that visually belong to the same cluster.



---

## Mis‑Seeing Dossier

A dedicated section documents mis‑seeing in more detail:

- Taxidermy animals described and retrieved as living creatures.  
- Goldfish and cephalopod plates treated as if they were photographs in aquariums.  
- Postcards of giant fruit mistaken for natural landscapes or realistic food photos.  
- Cases where the caption route collapses paintings, prints, and live photos into similar subject_status labels.

For each case, the dossier offers a short analysis: is the failure likely due to medium (photo versus drawing), background context (museum vs forest), art style, or known compositional weaknesses of contrastive encoders?

---

## Reflection and Limitations

This project is intentionally small in scale: a single archive, a few compact models, and a narrow research question. Its strengths lie in close reading of one collection rather than broad generalisation. Limitations include:

- Corpus size and bias toward warm‑climate scenes and public‑domain vintage material.  
- A single embedding model (SigLIP) and a single captioning model (Gemma 3 4B) under specific Colab constraints. [file:1][file:2]  
- No explicit training or fine‑tuning; all behaviour comes from off‑the‑shelf checkpoints.

Reflecting on the results, the project suggests that “seeing life” in images is not yet a simple emergent property of multimodal embeddings. Art style, background, and pose often outweigh the biological status of the subject, and caption‑based compression can both clarify and obscure the difference between living and still, depending on the case. Comparing these machine judgments with infant studies, where life/non‑life distinctions appear clearly in brain signals, highlights how much of our own visual intelligence is still missing from small, general‑purpose models.

---

## Sources and References

- Course materials: CompSci for Designers 2 – Seeing Machines notebooks and brief. [file:1][file:2]  
- Infant research on object categorisation and living vs non‑living distinctions. [web:211]  
- Model cards and documentation for SigLIP, Gemma 3 4B, and `all-MiniLM-L6-v2`.  
- Public‑domain image sources (botanical plates, mollusc and cephalopod illustrations, postcards, medieval manuscript details), listed in `LICENSES.md`.
