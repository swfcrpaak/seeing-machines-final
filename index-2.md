
# Seeing Machines – Life vs Stillness

**Min Jeong Park**  
CompSci for Designers 2 · TH Nürnberg

<img width="450" height="464" alt="IMG_8325" src="https://github.com/user-attachments/assets/d9e404ff-ae0a-4286-ba01-559984174cb7" />

*A small multimodal companion explores a personal archive of fauna and flora, question: what does it mean, for a machine, to “see life” in pictures?*

---

## Abstract

Human babies begin to sort the visual world into “living” and “non‑living” surprisingly early, a recent research found. This AI assisted brain‑imaging study shows that infants can distinguish living beings versus inanimate objects by just a few months of age. This project turns that question toward a small vision–language system and a personal archive of about one hundred images of fauna and flora, mixing live subjects and representation in the forms of: taxidermy, illustrations and paintings. The archive is explored through two routes: a Level 1 SigLIP search that embeds images directly, and a Level 2 caption‑based route where a VLM compresses each picture into a structured description, before text‑embedding and retrieval. By comparing where these compressed representations preserve or blur the line between life and stillness—especially in mis‑seeing cases where taxidermy or prints are treated as live—the project treats each retrieval path as a different form of “image compression as intelligence” and uses the machine’s mistakes as a way to look again at a familiar archive.

---

## Motivation and Corpus

I am interested in how very small systems, human or machine, begin to carve up the visual world. Infant research shows that babies do not just passively receive images; they gradually build categories such as “living” and “non‑living” long before they can talk about them. [web:211] This project translates that curiosity into a personal setting: how does a compact multimodal model see the same images I have collected, and where does its organising of the archive diverge from mine?

The corpus consists of 99 images:

- Original photographs of plants and animals taken by the author in different climate and environments
- Public‑domain illustrations and paintings of animals and plants

All public images in the GitHub repository come from sources that mark the underlying works as public domain or CC0, documented in a `LICENSES.txt` file. Private, rights‑sensitive images remain only in the local Colab corpus.

---

## Approach

<img width="800" alt="seeing_machines_pipeline" src="https://github.com/user-attachments/assets/43aafe8c-ae6f-42c4-a62a-dbd2f15a3cbd" /> Diagram generated with AI

- **Level 1 – The Finder (SigLIP route)**
  - SigLIP embeds all images once.
  - Text queries are embedded into the same space.
  - Cosine similarity returns the top‑k images in a Gradio interface.
  - A retrieval atlas logs at least ten queries (e.g. "live animal in nature", "vintage illustration of a plant") with embedding‑space reasoning: why this query returned these images.

- **Level 2 – The Companion (caption route)**
  - Gemma 3 4B (quantized) generates a structured JSON description for each image.
  - Testing and redesigning schema.
  - Captions are flattened to text and embedded with `all-MiniLM-L6-v2`.
  - A FAISS index powers caption‑based retrieval.
  - A language model (Gemma in text‑only mode) generates grounded answers from the retrieved captions, with the source images displayed alongside the answer.
  - A route toggle in the interface exposes both retrieval paths: caption route (Level 2) and SigLIP route (Level 1 baseline), so the same query can be run through either compression on demand.

Throughout, the project treats each route as a different way of compressing the archive: SigLIP compresses images into dense vectors; Gemma compresses them into compact verbal summaries. The analysis asks which compression better preserves the life vs representation distinction, and where both fail — which is really the same question the infant-cognition framing raises, just asked of a machine instead of a three-month-old.

---

## Lv.1 Results Gallery and Retrieval Atlas

All ten retrieval atlas queries and their embedding‑space reasoning, with screenshots in the appendix:

- Queries like "bright, colorful illustration of an animal" show that SigLIP has a stable "illustration" cluster where imaginary creatures, fish, and cephalopods sit together.
- Queries like "natural landscape with no live animals" reveal that landscapes with geese and fantasy animals remain in the same neighbourhood as empty parks and ponds, suggesting that broad scene statistics ("landscape‑ness") outweigh animal presence.
- Close‑up probes ("a close‑up photo of a live insect on a plant", "a live animal photographed outdoors, not a drawing") show that the embedding often collapses live subjects, taxidermy, printed images, and drawings into one "small creature + foliage" region.

These entries support the claim that Level 1's compression preserves colour, texture, and composition strongly, but only weakly distinguishes life vs representation.

- **Successful retrievals**
  - Query No.2: "bright, colorful illustration of an animal"
    <img src="Lv1.2.png" alt="Screenshot" width="800">
    Embedding‑space reasoning: Here the model excels. Imaginary creatures, sea creature drawings, and fantasy scenes form a coherent "bright illustration" neighbourhood with strong colour, clean outlines, and flat backgrounds.

- **Failure probes and binding tests**
  - Query No.1: "natural landscape with no live animals"
    <img src="Lv1.1.png" alt="Screenshot" width="800">  
    Embedding‑space reasoning: This query is meant to test absences (no live animals) over the "landscape" environment. Geese appears as the top image alongside pure landscapes. SigLIP finds a coherent "landscape" cluster—parks, trees, ponds—but animals and other creatures inside those scenes don't push the images out of that region. Geese and painted animals stay near pure landscapes if the colours and scene layout match. Negation ("no live animals") doesn't move the text embedding very far from "natural landscape", so the model still returns visually similar scenes even when they contain animals. This entry is a good example of granularity and negation issues: the model can find landscapes, but cannot prune away subjects that visually belong to the same cluster.

- **Mixed retrievals**
  - Query No.8: "close‑up of a live flower outdoors"
    <img src="Lv1.8.png" alt="Screenshot" width="800">
    Embedding‑space reasoning: SigLIP is clearly locking onto "big central figure + green background + outdoor lighting", not on whether the flower is actually alive. That's why it twice pulls printed flowers on a bus and a picture with a glass jar with nature in the background: visually they share saturated colours, petal/stem shapes, and centered composition. Medium (print vs real plant) seems much weaker in the embedding than basic colour and layout.

This is really where the whole project's question starts: if colour and layout outweigh liveness in a joint embedding, does swapping in a model that has to *say* what it sees, in words, do any better?

---

## Lv.2 Results: The Companion

### From subject_status to evidence: redesigning the schema

My first Level 2 schema treated "alive or not" as a single closed category — five original fields: `subject`, `habitat_or_context`, `notable_objects`, `activity`, and a seven-way `subject_status` (`live_outdoor`, `live_enclosure`, `dead_specimen_or_taxidermy`, `flat_representation_art`, `flat_representation_photo`, `food_or_processed`, `unclear`). It worked, in the sense that Gemma reliably produced a label. But a label on its own is exactly the kind of thing the Level 1 atlas already warned me about: SigLIP could confidently place a printed flower and a real one in the same neighbourhood without ever being wrong about anything it was actually asked to measure. A caption schema that only asks for a conclusion has the same failure mode — the model can assert `dead_specimen_or_taxidermy` with no more grounding than SigLIP's colour-and-layout clustering had.

So the redesign is built around one change: `subject_status` is no longer allowed to stand alone. It's paired with an `evidence` field that has to name the specific visual cue behind the judgment — a glass-case reflection, a rigid mounted pose, visible brushstrokes, natural depth-of-field blur on fur. If the model says "taxidermy," it now has to say *why*, in terms I can actually check against the photo. That turns a bare disagreement ("the model got this wrong") into something closer to a real mis-seeing case: sometimes the label is right but the cited evidence isn't there at all, which tells me much more about how the model is seeing than a wrong label by itself ever could.

Two fields — `notable_objects` and `activity` — were tried in the first version and dropped. Both described the image in general terms without bearing on the life/stillness question specifically. In their place I added `motion_state`, because "alive" and "moving" turned out to be two different questions the original schema quietly conflated — a sleeping animal and a taxidermy mount can both read as "still," for very different reasons, and the schema needs to be able to tell those apart. I also tried, then dropped, a `palette` field: colour doesn't actually discriminate alive from represented (a photograph and a painting can share a palette), so it wasn't earning its place against fields that do. `lighting`, `composition`, and `surface_texture` stayed, because the Level 1 atlas had already shown that colour, texture, and composition were exactly what SigLIP *was* picking up on — giving the caption route the same visual vocabulary lets me compare the two routes on genuinely equal footing instead of comparing a visual model against a schema that can't talk about visual things at all.

### Chat with the Archive

The Companion interface takes a question, retrieves the most relevant captions through the FAISS index, and asks Gemma to answer using only those retrieved descriptions — with the source images shown alongside the answer, so the grounding is checkable rather than asserted. Retrieval and generation are deliberately separated: what comes back as "relevant" is entirely a property of the caption embeddings, and what gets said about it is a second, separate step that can be wrong even when retrieval was right.

*[Example query/answer pair with screenshot goes here — e.g. "which photo is most likely to be mistaken for a live animal but actually isn't, and why?" is the strongest single demo, since it forces the model to use `subject_status` and `evidence` together rather than just retrieve on subject.]*

### Level 1 vs Level 2: when does compression preserve the life/stillness line, and when does it blur it

I ran both routes on the same five queries to see, directly, where the two forms of compression agree and where they part ways.

**"Which photos show living animals in motion?"** SigLIP's top five (similarities of 0.058 down to 0.043) and the caption route's top five (0.616 down to 0.555) share only two images. More telling than the overlap is what each route is actually keying on: the caption route is retrieving on an explicit `motion_state` judgment, so a genuinely blurred or mid-stride animal should surface reliably; SigLIP has no equivalent channel, so whatever visual correlate of "motion" it's using — blur, dynamic framing, maybe just outdoor-action-photo aesthetics in general — is much harder to name.

**"Which photos show living animals that are resting or still?"** was the closer pair — three of five images overlap. But two of those same images (IMG_5788, IMG_7077) also appear in *both* routes' top five for the *motion* query above, which is the more interesting finding than the overlap itself: neither compression cleanly separates "moving" from "still" on these particular photos. That's a real candidate for the mis-seeing dossier — whether it's genuine ambiguity in the photograph (an alert, motionless animal can honestly read either way) or a schema weakness is something I still need to check by opening those two images directly, not something either route's score can settle on its own.

**"Which images are flat representations like paintings, illustrations, or photos of a photo?"** is where the two routes diverge most sharply, and where the argument of this project is clearest. SigLIP's top five here include two *negative* similarities — meaning it found nothing it was actually confident about. The caption route's top five, by contrast, surfaced several images with filenames that don't match my own photography at all (a numbered stock-photo-style filename, a captioned specimen plate) — genuine representations, correctly identified as such. This is the schema doing exactly the job it was redesigned for: "is this a representation" is a categorical judgment I built a dedicated field to make and justify, while a general-purpose joint embedding has no comparable channel for it — the distinction has to fall out incidentally from whatever an image and a short phrase happen to share in embedding space, and here, it visibly doesn't.

**"Which photos look low light, dim, or artificially lit?"** and **"which photos look blurry, grainy, or low quality"** both show weak-to-no overlap between routes, and both map directly onto fields (`lighting`, `surface_texture`) that didn't exist in my first schema draft. The caption route answers both with clearly differentiated results; whether that's because the schema is actually working, or because SigLIP's raw similarity scores are compressed toward zero for reasons closer to how the model was trained than to any query-specific weakness, is a caveat I want to be upfront about rather than paper over — SigLIP's sigmoid-style training doesn't spread out raw cosine similarities the way a softmax-contrastive model like CLIP does, so a flatter score range on its own isn't proof of a worse embedding, just a different one.

Taken together: caption-mediated retrieval wins decisively on the one query that most directly targets the life/representation distinction (flat representations), which is the result I trust most, because it's exactly the distinction the schema was rebuilt to make explicit. On the more purely visual queries (lighting, texture), the routes diverge in ways that are harder to attribute cleanly to "better" versus "worse" without also accounting for how differently SigLIP and a sigmoid-trained sentence embedder produce their raw numbers in the first place.

### Level 2 Mis-Seeing Dossier

Documented cases where Gemma's caption diverges from what I can see with my own eyes, organised by failure type rather than by image:

- **Skewed description (subject_status).** Taxidermy animals described and retrieved as living creatures — the case the redesigned schema exists to catch. Under the old schema this was just a wrong label; under the new one, I can check whether the model's `evidence` field cites something real (a pose, a texture) or something fabricated, which is the actual diagnostic question.
- **Hallucination / context confusion.** Goldfish and cephalopod plates treated as though they were photographs taken inside an aquarium — the model appears to be inferring a plausible setting from subject matter alone, rather than reading the actual surface of the image (paper, printed line work) that would rule an aquarium photograph out.
- **Hallucination.** Postcards of giant fruit mistaken for natural landscapes or realistic food photography — a case where scale and genre cues (postcard framing, exaggerated proportion) get overridden by more generic scene recognition.
- **Systematic skew.** Across several images, the caption route collapses paintings, prints, and live photographs into similar `subject_status` labels — the general failure mode that motivated the whole schema redesign, rather than a single isolated case.

For each of these, the useful next step is the same: re-run the image through `ask_about_image` under the current schema and read what the `evidence` field actually claims. If the cited evidence is real but the conclusion is still wrong, that's a genuinely different kind of failure — a reasoning error — than a case where the evidence itself is invented, which is closer to a perceptual one. That distinction, not just a tally of right and wrong labels, is what the mis-seeing dossier is for.

---

## Reflection and Limitations

This project is intentionally small in scale: a single archive, a few compact models, and a narrow research question. Its strengths lie in close reading of one collection rather than broad generalisation. Limitations include:

- Corpus size and bias toward warm‑climate scenes and public‑domain vintage material.
- A single embedding model (SigLIP) and a single captioning model (Gemma 3 4B) under specific Colab constraints. [file:1][file:2]
- No explicit training or fine‑tuning; all behaviour comes from off‑the‑shelf checkpoints.
- `motion_state` is inferred from a single still frame, never from actual motion — an animal photographed at the top of a leap, where it is briefly nearly motionless, would likely read as "active but still" through no fault of the model's reasoning. That's a limit of asking about motion from a photograph at all, not a captioning bug to fix.

Reflecting on the results, the project suggests that "seeing life" in images is not yet a simple emergent property of multimodal embeddings, nor is it fully solved by asking a VLM to describe what it sees. Art style, background, and pose often outweigh the biological status of the subject in a joint embedding; a caption schema can do better, but only to the extent it is deliberately built to ask the right question and to show its reasoning rather than assert a conclusion. The comparison above suggests the caption route's advantage is real but narrower than I first expected — strongest exactly where the schema was built to be strong, and much less clearly ahead on purely visual properties where a joint embedding may simply be doing something differently, not worse. Comparing these machine judgments with infant studies, where life/non‑life distinctions appear clearly in brain signals, highlights how much of our own visual intelligence — the kind that doesn't need a field called `evidence` to justify itself — is still missing from small, general‑purpose models.

---

## AI Use Statement

I acknowledge the use of the following AI tools in the preparation of this documentation:

- **Perplexity AI**  – used to generate organization list and refine explanatory text for the documentation.  The output was reviewed, edited, and integrated manually; no content was used verbatim without adaptation.

- **Claude AI**  –  used as a coding and writing assistant: debugging notebook errors, reviewing and revising code structure, iterating on the description schema through discussion, drafting and editing this documentation page, and generating the pipeline diagram. All corpus curation, schema design decisions, query selection, retrieval and mis-seeing analysis, and final interpretations are my own. Where documentation text was AI-drafted, it was written from my own design reasoning and reviewed and edited by me before submission.
  
- **GitHub Copilot** (https://github.com/features/copilot) – used within the code editor to assist with writing and checking Markdown syntax and small code snippets. All suggestions were accepted, modified, or rejected at my discretion.

No AI tool was used to generate core research content, design decisions, or original analysis. All substantive content, structure, and conclusions in this documentation are my own work.

--

## Sources and References

- Infant research on object categorisation and living vs non‑living distinctions. [web:211]
- Model cards and documentation for SigLIP, Gemma 3 4B, and `all-MiniLM-L6-v2`.
- Public‑domain image sources (botanical plates, mollusc and cephalopod illustrations, postcards, medieval manuscript details), listed in `LICENSES.md`.
