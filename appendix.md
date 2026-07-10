---
layout: default
title: "Appendix"
---

# Appendix

## Appendix A. Retrieval Atlas — Level 1

This appendix documents all ten Level 1 queries used in the retrieval atlas. Each entry shows the query, settings, and a brief explanation grounded in embedding-space reasoning.

### Figure A1. Query: "natural landscape with no live animals"
![Lv1.1 screenshot](Lv1.1.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** SigLIP builds a “landscape” cluster composed of parks, ponds, and forests. Animals and creatures inside those scenes do not push images out of this region, so “no live animals” behaves more like a general “landscape” query than a true absence test.

### Figure A2. Query: "bright, colorful illustration of an animal"
![Lv1.2 screenshot](Lv1.2.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** Here the model excels. Medieval creatures, fish plates, cephalopod drawings, and fantasy scenes form a coherent “bright illustration” neighbourhood with strong colour, clean outlines, and flat backgrounds.

### Figure A3. Query: "a live animal photographed outdoors, not a drawing"
![Lv1.3 screenshot](Lv1.3.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** Even with “not a drawing” in the text, one illustration appears and indoor or screen images still sneak in. The embedding seems dominated by “animal + natural-ish background” and only weakly respects modality and negation.

### Figure A4. Query: "a close-up photo of a live insect on a plant"
![Lv1.4 screenshot](Lv1.4.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** The neighbourhood is “insect + plant + close framing”, not “live on plant”. Pinned specimens, cartoon insects, decorative screens, and vintage plates all rank high because they share silhouettes and leaf textures.

### Figure A5. Query: "a live animal in its natural habitat"
![Lv1.5 screenshot](Lv1.5.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** The embedding creates one large “animal + environment” cluster. Aquariums, zoo enclosures, biodome scenes, and outdoor habitats all look similar in this space, so “natural habitat” does not separate wild from designed exhibits.

### Figure A6. Query: "a taxidermy animal on display in a museum"
![Lv1.6 screenshot](Lv1.6.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** The model clusters “animal + display context”: pinned insects, butterflies in cases, plush toys, and even a sleeping cat on a shelf. It recognises “specimen on display” but does not separate dead specimens from toys or live animals.

### Figure A7. Query: "a detailed drawing or painting of a plant"
![Lv1.7 screenshot](Lv1.7.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** SigLIP strongly detects “this is art” (brushstrokes, paper texture), but relaxes the “plant” part, pulling in cephalopods and hybrid creatures. Art style is encoded more strongly than the specific plant category.

### Figure A8. Query: "close-up of a live flower outdoors"
![Lv1.8 screenshot](Lv1.8.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** SigLIP focuses on “large pink flower + green background + outdoor lighting”, so printed flowers on buses and vending machines cluster with real flowers. Medium (print versus live plant) is weaker than colour and composition.

### Figure A9. Query: "a detailed drawing or engraving of a fish"
![Lv1.9 screenshot](Lv1.9.1.png)
![Lv1.9 screenshot](Lv1.9.2.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** Goldfish and cephalopod plates cluster together as “scientific illustration”, but fish-like colours and shapes in photos can drift into the same neighbourhood, showing how composition and outline sometimes outweigh medium.

### Figure A10. Query pair: "a red flower on a green background" vs. "a green flower on a red background"
![Lv1.10 screenshot](Lv1.10.png)
- `top_k`: 8 · `min_similarity`: -0.05
- **Why:** Both queries return very similar sets, suggesting that SigLIP treats “red + green + flower” as a bag of features. It does not consistently bind the colour to the correct object, which matches known compositional weaknesses of CLIP-style encoders.

### Table A1. Pairwise cosine similarity for the top 8 matches for Figure A1

| index | IMG_2611.HEIC | IMG_5719.HEIC | IMG_1449.HEIC | IMG_1403.HEIC | IMG_1450.HEIC | IMG_1885.HEIC | IMG_7997.HEIC | IMG_4353.JPG |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| IMG_2611.HEIC | 1.000 | 0.606 | 0.707 | 0.716 | 0.733 | 0.695 | 0.556 | 0.579 |
| IMG_5719.HEIC | 0.606 | 1.000 | 0.570 | 0.596 | 0.557 | 0.611 | 0.555 | 0.574 |
| IMG_1449.HEIC | 0.707 | 0.570 | 1.000 | 0.832 | 0.919 | 0.753 | 0.611 | 0.700 |
| IMG_1403.HEIC | 0.716 | 0.596 | 0.832 | 1.000 | 0.866 | 0.741 | 0.661 | 0.687 |
| IMG_1450.HEIC | 0.733 | 0.557 | 0.919 | 0.866 | 1.000 | 0.758 | 0.638 | 0.713 |
| IMG_1885.HEIC | 0.695 | 0.611 | 0.753 | 0.741 | 0.758 | 1.000 | 0.622 | 0.628 |
| IMG_7997.HEIC | 0.556 | 0.555 | 0.611 | 0.661 | 0.638 | 0.622 | 1.000 | 0.656 |
| IMG_4353.JPG | 0.579 | 0.574 | 0.700 | 0.687 | 0.713 | 0.628 | 0.656 | 1.000 |

---

## Appendix B. Level 2 Deliverables and Comparison Figures

The figures below correspond to the Level 2 comparison discussion in `index.md`. Where the main documentation refers to motion, lighting, composition, colour, the adversarial pair, and the Level 1-vs-Level 2 comparison queries, it is referring to these numbered screenshots.

### Figure B1. Level 2 comparison screenshot 1
<img alt="Lv2 c 1" src="https://github.com/user-attachments/assets/4e381695-b63f-495b-9c4d-b02df611fc9b" />

### Figure B2. Level 2 comparison screenshot 2
<img alt="Lv2 c 2" src="https://github.com/user-attachments/assets/433b50a9-33d0-4b71-8a10-d695e63dd962" />

### Figure B3. Level 2 comparison screenshot 3
<img  alt="Lv2 c 3" src="https://github.com/user-attachments/assets/dfef8dd4-9877-453a-b5f8-d731ccd0035a" />

### Figure B4. Level 2 comparison screenshot 4
<img alt="Lv2 c 4" src="https://github.com/user-attachments/assets/5f89a4a8-1e59-45a9-a0de-c236dc5b41cc" />

### Figure B5. Level 2 comparison screenshot 5
<img alt="Lv2 c 5" src="https://github.com/user-attachments/assets/ba5e2c07-052f-442b-8253-79abe06b6823" />

### Figure B6. Level 2 comparison screenshot 6
<img alt="Lv2 c 6" src="https://github.com/user-attachments/assets/ffdce83d-f6e5-4b97-bba1-24f9238f782b" />

### Figure B7. Level 2 comparison screenshot 7
<img alt="Lv2 c 7" src="https://github.com/user-attachments/assets/3b4cf5fc-7add-4f57-976a-d408568c521e" />

### Figure B8. Level 2 comparison screenshot 8
<img alt="Lv2 c 8" src="https://github.com/user-attachments/assets/043e3460-b7a2-487f-bf85-65f7fe564334" />

### Figure B9. Level 2 comparison screenshot 9
<img alt="Lv2 c 9" src="https://github.com/user-attachments/assets/49895983-6748-44c8-ab21-c96e8c2c27d0" />

### Figure B10. Level 2 comparison screenshot 10
<img alt="Lv2 c 10" src="https://github.com/user-attachments/assets/bd0dd614-da39-4819-b33f-6952794f95ad" />

### Figure B11. Level 2 comparison screenshot 11
<img alt="Lv2 c 11" src="https://github.com/user-attachments/assets/740b3470-72da-4a1b-8b05-f54cd3060ad9" />

### Figure B12. Level 2 comparison screenshot 12
<img alt="Lv2 c 13" src="https://github.com/user-attachments/assets/3f04e307-726c-42c9-b4ca-5d6ab7a0e48a" />

### Figure B13. Level 2 comparison screenshot 13
<img alt="Lv2 c 12" src="https://github.com/user-attachments/assets/2e9057b5-2ed4-4fc8-8955-d370f1738106" />

### Figure B14. Level 2 comparison screenshot 14
<img alt="Lv2 c 14" src="https://github.com/user-attachments/assets/3fa143cc-fd09-41db-8a75-d7e5c677ef93" />

### Figure B15. Level 2 comparison screenshot 15
<img alt="Lv2 c 15" src="https://github.com/user-attachments/assets/5e43fdb6-f7b7-4377-aae6-ba87636e20cd" />
