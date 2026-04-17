# Embedding Model Benchmark — Stage 1
 
This document reports the full benchmark results for the four retrieval models evaluated in Stage 1 of ASOS-CRM. The purpose of Stage 1 is to retrieve, for each column of a structured CSV dataset, the most semantically relevant CIDOC CRM classes and properties prior to OWL validation. Because the input data is in French while CIDOC CRM is defined in English, cross-lingual semantic retrieval capability is a critical selection criterion.
 
These results were omitted from the DaWaK 2026 paper due to space constraints.
 
 
## Experimental Context
 
The benchmark is conducted on the **Stratigraphic units list** dataset (*liste des faits*, `Stratigraphic units.csv`), one of the four archaeological datasets used in the evaluation. This dataset records stratigraphic observations from the *Hypogeum of the Dunes* (Poitiers, France) and comprises **8 columns** in French: `n° US`, `Localisation`, `Etendue`, `Antérieur à`, `Postérieur à`, `Egal à`, `Etat`, `Mortier`. All column names and values are in French; all CIDOC CRM labels, scope notes, and examples are in English.

## Models Evaluated
 
| Model | HuggingFace reference | Type | Cross-lingual |
|-------|-----------------------|------|:-------------:|
| BM25-Okapi | — (lexical) | Lexical baseline | ✗ |
| all-mpnet-base-v2 | `sentence-transformers/all-mpnet-base-v2` | Monolingual dense | ✗ |
| LaBSE | `sentence-transformers/LaBSE` | Cross-lingual (translation) | ✓ |
| **paraphrase-multilingual-mpnet-base-v2** | `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` | **Multilingual paraphrase** | **✓** |
 

## Metrics
 
**OWL-Valid Triplets** : candidate triplets satisfying the CIDOC CRM domain and range axioms: `c_s ⊑ D(p)` and `c_o ⊑ R(p)`.
 
**OWL Validity Rate** : OWL-valid triplets as a percentage of total candidates.
 
**Accepted Triplets** : OWL-valid triplets that also correspond to a semantically correct mapping, as determined by expert validation against a reference annotation.
 
**Precision among Valid** : accepted triplets as a percentage of OWL-valid triplets.
 
**Avg. Class / Property Similarity** : average cosine similarity of accepted triplets, reflecting the quality of the embedding alignment.
 


## Results
 
 
| Model | Total Candidates | OWL-Valid | OWL Validity Rate (%) | Accepted | Precision among Valid (%) | Coverage | Avg. Class Sim. | Avg. Prop. Sim. |
|-------|:----------------:|:---------:|:---------------------:|:--------:|:-------------------------:|:--------:|:---------------:|:---------------:|
| BM25-Okapi | 400 | 12 | 3.0 | 0 | 0.0 | 0/8 | — | — |
| all-mpnet-base-v2 | 400 | 42 | 10.5 | 4 | 9.5 | 3/8 | 0.3705 | 0.4717 |
| LaBSE | 400 | 6 | 1.5 | 5 | 83.3 | 5/8 | 0.3622 | 0.4840 |
| **paraphrase-multilingual-mpnet-base-v2** | **400** | **22** | **5.5** | **12** | **54.5** | **6/8** | **0.4185** | **0.5537** |
 
Bold = selected model.


## Discussion
 
BM25-Okapi produces zero accepted triplets, confirming that lexical retrieval is incompatible with this cross-lingual setting. `all-mpnet-base-v2` generates the most OWL-valid candidates (42, 10.5%) but achieves only 9.5% precision among them, the lowest of the neural models, indicating that structural admissibility and semantic correctness are largely decorrelated for a monolingual model operating across languages. LaBSE reaches 83.3% precision but on only 6 valid candidates, reflecting its tendency to succeed where direct translation suffices and fail elsewhere. `paraphrase-multilingual-mpnet-base-v2` achieves the highest number of accepted triplets (12), the highest class and property similarity scores (0.4185 and 0.5537), and is the only model that produces semantically correct mappings for columns requiring paraphrase-level alignment between French domain terms and English ontological scope notes. It is therefore retained for Stage 1 of ASOS-CRM.
 
Across all models, the OWL validation layer rejects between 89.5% and 98.5% of candidates, confirming that semantic similarity alone is insufficient to enforce ontological structural constraints.