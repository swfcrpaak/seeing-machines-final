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

## Lv.1 The Finder and Retrieval Atlas

All ten retrieval atlas queries and their embedding‑space reasoning, with screenshots in the appendix:

- Queries like "bright, colorful illustration of an animal" show that SigLIP has a stable "illustration" cluster where imaginary creatures, fish, and cephalopods sit together.
- Queries like "natural landscape with no live animals" reveal that landscapes with geese and fantasy animals remain in the same neighbourhood as empty parks and ponds, suggesting that broad scene statistics ("landscape‑ness") outweigh animal presence.
- Close‑up probes ("a close‑up photo of a live insect on a plant", "a live animal photographed outdoors, not a drawing") show that the embedding often collapses live subjects, taxidermy, printed images, and drawings into one "small creature + foliage" region.

These entries support the claim that Level 1's compression preserves colour, texture, and composition strongly, but only weakly distinguishes life vs representation.

- **Successful retrievals**
  - Query No.2: "bright, colorful illustration of an animal"
    <img src="Lv1.2.png" alt="Screenshot" width="800">
    Embedding‑space reasoning: Here the model excels. Imaginary creatures, sea creature drawings, and fantasy scenes form a coherent     "bright illustration" neighbourhood with strong colour, clean outlines, and flat backgrounds.

- **Failure probes and binding tests**
  - Query No.1: "natural landscape with no live animals"
    <img src="Lv1.1.png" alt="Screenshot" width="800">

    Embedding‑space reasoning: This query is meant to test absences (no live animals) over the "landscape" environment. Geese     appears as the top image alongside pure landscapes. SigLIP finds a coherent "landscape" cluster—parks, trees, ponds—but animals and other creatures inside those scenes don't push the images out of that region. Geese and painted animals stay near pure landscapes if the colours and scene layout match. Negation ("no live animals") doesn't move the text embedding very far from "natural landscape", so the model still returns visually similar scenes even when they contain animals. This entry is a good example of granularity and negation issues: the model can find landscapes, but cannot prune away subjects that visually belong to the same cluster.
    
    UMAP Analysis
    <img width="1021" height="876" alt="Unknown-29" src="https://github.com/user-attachments/assets/d136c797-a56c-4090-8f7d-622341a2fff0" />

    Observations from the Visualization:

-Clustering: In the UMAP plot, you'll see that the 'geese' image (IMG_2611.HEIC) is clustered very closely to images of landscapes. This suggests that the model is prioritizing the 'natural landscape' part of query over the 'no live animals' negative constraint.
-Texture & Composition: SigLIP often focuses on global features. The visual texture of birds grass often mimics the patterns found in purely natural textures (like rocks or dappled light), causing the model to see them as high-quality 'landscape' matches.
-Similarity Matrix: As shown in Table A1 (Appendix A), several pairwise similarity values between the geese image (IMG_2611.HEIC) and other landscape photos are notably high (e.g., 0.707, 0.716). This indicates that, in the model’s internal representational space, these images are nearly indistinguishable from “natural landscapes.”

- **Mixed retrievals**
  - Query No.8: "close‑up of a live flower outdoors"
    <img src="Lv1.8.png" alt="Screenshot" width="800">

    UMAP Analysis
      <img width="1983" height="955" alt="Unknown-31" src="https://github.com/user-attachments/assets/fe7f68bb-3bbf-458f-a232-98a22aeb9fd8" />

     Embedding‑space reasoning: SigLIP is clearly locking onto "big central figure + green background + outdoor lighting", not on whether the flower is actually alive. That's why it twice pulls printed flowers on a bus and a picture with a glass jar with nature in the background: visually they share saturated colours, petal/stem shapes, and centered composition. Medium (print vs real plant) seems much weaker in the embedding than basic colour and layout.
    
Moving on to Level 2...
"*This is really where the project’s question starts: if, in this vision-only embedding, colour and layout outweigh liveness, what changes when I swap in a model that has to put what it sees into words? Does asking the model to speak make liveness matter more—or does it just translate the same visual bias into language?*"

---

## Lv.2 The Companion

### Schema evolution

The schema went through a few iterations before settling. The first version treated
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


### Side-by-side Comparison on Visual Cues

**Motion**

  "a still, motionless pose that could be mistaken for taxidermy"
<img width="1427" height="941" alt="Lv2 c 2" src="https://github.com/user-attachments/assets/51d3d8df-dafb-429d-958d-a63636fbc0e8" />

  - SigLIP wins, while Caption route focus on bringing images closer to taxidermy, can't understand the query as well as CLIP. Motion cue alone can't predict correct status, and "still" language biases the model toward false taxidermy calls even for a sleeping live animal.

**Lighting**

  "a subject under flat, even indoor artificial lighting"
<img width="1427" height="941" alt="Lv2 c 6" src="https://github.com/user-attachments/assets/8cf66196-73ce-4ab1-823b-0644c2c76012" />

  - Neither routes suggest that flat even lighting skewing for staged/displayed settings. Lighting isn't actually carrying status signal, just retrieval signal. Subject status are varying from inanimate object to living plant. 

**Composition**

  "a macro close-up of a live subject in its natural environment"
<img width="1427" height="941" alt="Lv2 c 7" src="https://github.com/user-attachments/assets/603c4621-43a9-43e6-8872-c09096c3c3ad" />

  -Mixed result for both routes. Caption route concentrates more on living subject in its natural environment than the composition (close-up). CLIP brings close-up but not all living subjects. 

**Color**

  "a mostly green, leafy scene" 
<img width="1427" height="941" alt="Lv2 c 9" src="https://github.com/user-attachments/assets/b380a936-5c3a-4ec0-899c-b61975be46da" />

  -SigLIP retrieved more "greeness". Caption route retrieval: color starts correlating with status, worth reconsidering dropping palette in schema. 

**The adversarial pair**

  "a live animal that could be mistaken for taxidermy because it's holding still"
<img width="1427" height="941" alt="Lv2 c 11" src="https://github.com/user-attachments/assets/5deac64b-625c-4253-94f0-9e72f3058af8" />

  "a taxidermy mount or illustration that could be mistaken for a living animal"
<img width="1427" height="941" alt="Lv2 c 10" src="https://github.com/user-attachments/assets/d161fb8e-e555-4871-94c7-c89499d5cf1e" />

  - Caption route wins, CLIP retrieved illustrations of imaginary beings. Clearly not enough to tell what is actually "a living animal." 


### Level 1 vs Level 2: when does compression preserve the life/stillness line, and when does it blur it

I ran both routes on the same five queries to see, directly, where the two forms of compression agree and where they part ways.

**"Which photos show living animals in motion?"** SigLIP's top five (similarities of 0.058 down to 0.043) and the caption route's top five (0.616 down to 0.555) share only two images. More telling than the overlap is what each route is actually keying on: the caption route is retrieving on an explicit `motion_state` judgment, so a genuinely blurred or mid-stride animal should surface reliably; SigLIP has no equivalent channel, so whatever visual correlate of "motion" it's using — blur, dynamic framing, maybe just outdoor-action-photo aesthetics in general — is much harder to name.

**"Which photos show living animals that are resting or still?"** was the closer pair — three of five images overlap. But two of those same images (IMG_5788, IMG_7077) also appear in *both* routes' top five for the *motion* query above, which is the more interesting finding than the overlap itself: neither compression cleanly separates "moving" from "still" on these particular photos. That's a real candidate for the mis-seeing dossier — whether it's genuine ambiguity in the photograph (an alert, motionless animal can honestly read either way) or a schema weakness is something I still need to check by opening those two images directly, not something either route's score can settle on its own.

**"Which images are flat representations like paintings, illustrations, or photos of a photo?"** is where the two routes diverge most sharply, and where the argument of this project is clearest. SigLIP's top five here include two *negative* similarities — meaning it found nothing it was actually confident about. The caption route's top five, by contrast, surfaced several images with filenames that don't match my own photography at all (a numbered stock-photo-style filename, a captioned specimen plate) — genuine representations, correctly identified as such. This is the schema doing exactly the job it was redesigned for: "is this a representation" is a categorical judgment I built a dedicated field to make and justify, while a general-purpose joint embedding has no comparable channel for it — the distinction has to fall out incidentally from whatever an image and a short phrase happen to share in embedding space, and here, it visibly doesn't.

**"Which photos look low light, dim, or artificially lit?"** and **"which photos look blurry, grainy, or low quality"** both show weak-to-no overlap between routes, and both map directly onto fields (`lighting`, `surface_texture`) that didn't exist in my first schema draft. The caption route answers both with clearly differentiated results; whether that's because the schema is actually working, or because SigLIP's raw similarity scores are compressed toward zero for reasons closer to how the model was trained than to any query-specific weakness, is a caveat I want to be upfront about rather than paper over — SigLIP's sigmoid-style training doesn't spread out raw cosine similarities the way a softmax-contrastive model like CLIP does, so a flatter score range on its own isn't proof of a worse embedding, just a different one.

Taken together: caption-mediated retrieval wins decisively on the one query that most directly targets the life/representation distinction (flat representations), which is the result I trust most, because it's exactly the distinction the schema was rebuilt to make explicit. On the more purely visual queries (lighting, texture), the routes diverge in ways that are harder to attribute cleanly to "better" versus "worse" without also accounting for how differently SigLIP and a sigmoid-trained sentence embedder produce their raw numbers in the first place.


### Mis-Seeing Dossier

- **Skewed description (subject_status).**
  - "image_name": "IMG_4351.JPG",
    "model_said": "representation: Lack of Biological Signs: There are no indications of any living organisms",
    "actually_is": "trees in the picture; cityscape outdoors",
    "note": = "The big red sculpture and architecture context give wrong category",

- **Hallucination / context confusion.**

  - "image_name": "IMG_5888.HEIC",
   Old schema had "subject": "two felted rabbits indoors" as classified with "subject_status":"living animal or plant photographed outdoors'".
  +Counting error: Other elephant felted toy in the back most likely got mistaken as second rabbit.
    "model_said": "two rabbits, three carrots",
    "actually_is": "one felted rabbit and one carrot",
    "note": "most likely counting elephant felted animal in the back as the second rabbit, and doubling the result. "
    
  - "image_name": "IMG_7063.HEIC",
    "model_said": "model says it's a representation, and it also sees a robin and a blue jay.",
    "actually_is": "real floral bouquet in a vase on a kitchen counter",
    "note": "wilted and drooping look of flowers give false verdict. It claims to see several birds but probably mistaken from the leaves."
  
      

---

## Reflection and Limitations

This project is intentionally small in scale: a single archive, a few compact models, and a narrow research question. Its strengths lie in close reading of one collection rather than broad generalisation. Limitations include:

-Corpus size and bias toward warm‑climate scenes and less representation material.
-A single embedding model (SigLIP) and a single captioning model (Gemma 3 4B) under specific Colab constraints. 
-No explicit training or fine‑tuning; all behaviour comes from off‑the‑shelf checkpoints.
-Both the corpus and the schema went through several rounds of revision over the course of the project.

Reflecting on the results, the project suggests that "seeing life" in images is not yet a simple emergent property of multimodal embeddings, nor is it fully solved by asking a VLM to describe what it sees. Art style, background, and pose often outweigh the biological status of the subject in a joint embedding; a caption schema can do better, but only to the extent it is deliberately built to ask the right question and to show its reasoning rather than assert a conclusion. The comparison above suggests the caption route's advantage is working but narrower than I first expected — strongest exactly where the schema was built to be strong. But it is much less clear on purely visual properties where a joint embedding may simply be doing better. That the schema needed multiple revisions to get there is itself part of the finding: a first attempt at "ask the model to say what it sees" defaults to the same unverifiable-label problem the embeddings had, and it took deliberate building in a field that forces evidence, not just a conclusion, to move past it. Comparing these machine judgments with infant studies, where life/non‑life distinctions appear clearly in brain signals, highlights how much of our own visual intelligence — the kind that doesn't need a field called evidence to justify itself — is still missing from small, general‑purpose models.


---

## AI Use Statement

I acknowledge the use of the following AI tools in the preparation of this documentation:

- **Perplexity AI**  – used to generate organization list and refine explanatory text for the documentation.  The output was reviewed, edited, and integrated manually; no content was used verbatim without adaptation.

- **Claude AI**  –  used as a coding and writing assistant: debugging notebook errors, reviewing and revising code structure, iterating on the description schema through discussion, drafting and editing this documentation page, and generating the pipeline diagram. All corpus curation, schema design decisions, query selection, retrieval and mis-seeing analysis, and final interpretations are my own. Where documentation text was AI-drafted, it was written from my own design reasoning and reviewed and edited by me before submission.
  
- **GitHub Copilot** (https://github.com/features/copilot) – used within the code editor to assist with writing and checking Markdown syntax and small code snippets. All suggestions were accepted, modified, or rejected at my discretion.

No AI tool was used to generate core research content, design decisions, or original analysis. All substantive content, structure, and conclusions in this documentation are my own work.

--

## Sources and References

- O’Doherty, C., Dineen, Á.T., Truzzi, A. et al. Infants have rich visual categories in ventrotemporal cortex at 2 months of age. Nat Neurosci 29, 693–702 (2026). https://doi.org/10.1038/s41593-025-02187-8
- Public‑domain image sources (botanical plates, mollusc and cephalopod illustrations, postcards, medieval manuscript details), listed in `LICENSES.md`.
