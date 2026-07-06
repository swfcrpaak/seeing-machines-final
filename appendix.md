## Appendix 

Appendix A: Retrieval Atlas — Level 1 

This appendix documents all ten Level 1 queries used in the retrieval atlas.  
Each entry shows the query, settings, and a brief explanation grounded in embedding-space reasoning.

### 1. "natural landscape with no live animals"
![Lv1.1 screenshot](Lv1.1.png)
- top_k: 8 · min_similarity: -0.05
- Why: SigLIP builds a “landscape” cluster: parks, ponds,and forests. Animals and creatures inside those scenes do not push images out of this region, so “no live animals” behaves like a “landscape” query.

### 2. "bright, colorful illustration of an animal"
![Lv1.1 screenshot](Lv1.2.png)
- top_k: 8 · min_similarity: -0.05  
- Why: Here the model excels. Medieval creatures, fish plates, cephalopod drawings, and fantasy scenes form a coherent “bright illustration” neighbourhood with strong colour, clean outlines, and flat backgrounds.

### 3. "a live animal photographed outdoors, not a drawing"
![Lv1.1 screenshot](Lv1.3.png)
- top_k: 8 · min_similarity: -0.05  
- Why: Even with “not a drawing” in the text, one illustration appears, and indoor or screen images sneak in. The embedding seems dominated by “animal + natural-ish background” and only weakly respects modality and negation.

### 4. "a close-up photo of a live insect on a plant"
![Lv1.1 screenshot](Lv1.4.png)
- top_k: 8 · min_similarity: -0.05  
- Why: The neighbourhood is “insect + plant + close framing”, not “live on plant”. Pinned specimens, cartoon insects, decorative screens, and vintage plates all rank high because they share silhouettes and leaf textures.

### 5. "a live animal in its natural habitat"
![Lv1.1 screenshot](Lv1.5.png)
- top_k: 8 · min_similarity: -0.05  
- Why: The embedding creates one big “animal + environment” cluster. Aquariums, zoo enclosures, biodome scenes, and outdoor habitats all look similar in this space, so “natural habitat” doesn’t separate wild from designed exhibits.

### 6. "a taxidermy animal on display in a museum"
![Lv1.1 screenshot](Lv1.6.png)
- top_k: 8 · min_similarity: -0.05  
- Why: The model clusters “animal + display context”: pinned insects, butterflies in cases, plush toys, and even a sleeping cat on a shelf. It recognises “specimen on display” but doesn’t separate dead specimens from toys or live animals.

### 7. "a detailed drawing or painting of a plant"
![Lv1.1 screenshot](Lv1.7.png)
- top_k: 8 · min_similarity: -0.05  
- Why: SigLIP strongly detects “this is art” (brush strokes, paper texture) but relaxes the “plant” part, pulling in cephalopods and hybrid creatures. Art style is encoded more strongly than the specific plant category.

### 8. "close-up of a live flower outdoors"
![Lv1.1 screenshot](Lv1.8.png)
- top_k: 8 · min_similarity: -0.05  
- Why: SigLIP focuses on “big pink flower + green background + outdoor lighting”, so printed flowers on buses and vending machines cluster with real flowers. Medium (print vs live plant) is weaker than colour and composition.

### 9. "a detailed drawing or engraving of a fish"
![Lv1.1 screenshot](Lv1.10.png)
- top_k: 8 · min_similarity: -0.05  
- Why: Goldfish and cephalopod plates cluster together as “scientific illustration”, but fish‑like colours and shapes in photos can drift into the same neighbourhood, showing how composition and outline sometimes outweigh medium.

### 10. "a red flower on a green background" vs "a green flower on a red background"
![Lv1.1 screenshot](Lv1.9.1.png)
![Lv1.1 screenshot](Lv1.9.2.png)
- top_k: 8 · min_similarity: -0.05
- Why: Both queries return very similar sets, suggesting SigLIP treats “red + green + flower” as a bag of features. It doesn’t consistently bind the colour to the right object, which matches known compositional weaknesses of CLIP‑style encoders.
