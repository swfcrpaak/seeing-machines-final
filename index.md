# Seeing Machines – Live vs Representation

**Min Jeong Park**  
CompSci for Designers 2 · TH Nürnberg · Level 2 – The Companion

![Teaser image](media/teaser.jpg)

*A multimodal archive companion that studies how SigLIP and Gemma 3 caption a small photo collection of plants and animals, asking whether they see living subjects or representations such as taxidermy and drawings.*

## Abstract

[Write 3–5 sentences summarising your corpus, system (SigLIP + caption-RAG), and research question.]
My corpus consists of 88 photographs taken by me, containing plants, animals, and their immediate environments. Corpus contains balance of live vs representation of animals and plants. The collection skews toward warm climate and contains relatively few northern hemishphere flora. I reviewed all images and excluded any containing identifiable people. My research question is whether VLM and SigLIP can correctly distinguish live subjects (photographed in situ) from representations of subjects — taxidermy, drawings — and whether that distinction appears in the captions and therefore affects retrieval.

## Motivation and Corpus

[Describe your 88-image corpus: live vs representation, warm-climate bias, no people, why you chose it, how you curated it.]

## Approach

[Briefly explain the pipeline: SigLIP embeddings (Level 1), Gemma 3 captions + MiniLM+FAISS (Level 2), interface, description schema.]

## Results and Retrieval Gallery

[Include several figures: interface screenshots for successful retrievals, disagreements between routes, and mis-seeing cases. Each with a short caption.]

## Mis-seeing and Discussion

[Summarise the main failure patterns and differences between SigLIP and caption-based retrieval.]

## Reflection and Limitations

[Reflect on what you learned and list key limitations: corpus size, bias, small VLM, GitHub Pages only shows some images.]

## Sources and References

[Link to model cards, course materials, DreamBooth project page, etc.]
