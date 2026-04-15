# ASOS-CRM: Automated Semantic Scoping for CIDOC CRM Population

This repository provides the implementation of **ASOS-CRM** (Automated Semantic Ontological Scoping for CRM), a two-stage pipeline that automatically selects the relevant CIDOC CRM classes and properties for a given structured dataset, and uses the resulting ontological scope to drive LLM-based RDF triple extraction.

> **The archaeological datasets used in this project are not included in the repository due to privacy, confidentiality, and legal restrictions.**

Developed as part of the French ANR project **Digitalis** (ANR-22-CE38-0011-01). For more information, visit the [project page](https://digitalis.humanities.science/).

## Overview

Mapping structured data to large ontologies such as CIDOC CRM remains a challenging task for Large Language Models (LLMs). Prior work demonstrated that providing the LLM with a small, domain-relevant ontology *subset* rather than the full specification significantly improves annotation quality. However, constructing this subset has so far required manual expert work, limiting automation.

ASOS-CRM addresses this bottleneck through a two-stage approach:

1. **Stage 1 — Semantic Embedding:** For each column of the input dataset, a multilingual sentence embedding model retrieves the most semantically similar CIDOC CRM classes and properties using cosine similarity.
2. **Stage 2 — OWL Logical Validation:** Every candidate triplet *(subject class, property, object class)* is checked against the OWL domain and range axioms of CIDOC CRM. Only logically valid triplets are retained. If no valid triplet is found for a column, the retrieval scope is expanded adaptively.

The union of valid triplets across all columns forms the **ontological scope S**, which replaces the full ontology as context for LLM annotation.

<!-- ![ASOS-CRM pipeline overview](docs/figures/pipeline_overview.png) -->

## Features

- Automatically generates a focused, logically valid CIDOC CRM ontological scope from any structured CSV dataset.
- Combines multilingual dense retrieval (`paraphrase-multilingual-mpnet-base-v2`) with formal OWL constraint checking.
- Supports optional **expert column descriptions** to further improve retrieval precision.

## Approach

### Stage 1: Semantic Embedding

For each class and property in CIDOC CRM, a textual representation is built by concatenating the official label, the scope note, and an example of use. For each dataset column, the system builds a data vector from the column header, sample values, and (optionally) an expert natural-language description. Cosine similarity is then used to retrieve the Top-K_C most similar classes and Top-K_P most similar properties.

### Stage 2: OWL Logical Validation

The OWL axioms of CIDOC CRM are parsed to extract the declared domain D(p) and range R(p) for each property p. For every candidate triplet (c_s, p, c_o), the engine checks:

```
c_s ⊑ D(p)  and  c_o ⊑ R(p)
```

where ⊑ denotes subsumption in the CIDOC CRM class hierarchy. Only triplets satisfying both constraints are retained. When no valid triplet is found for a column, K is incremented by 1 and the search is retried (adaptive expansion).

## Prerequisites

- Python 3.9 or higher
- At least 8 GB RAM recommended for embedding model loading

## Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/lias-laboratory/asos-crm
   cd asos-crm
   ```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Configure API credentials and paths in `config.py`:

   ```python
   OPENAI_API_KEY = "your-api-key-here"
   CIDOC_CRM_OWL  = "data/ontologies/cidoc_crm.owl"
   ```

## Embedding Model Selection
 
The selection of the embedding model for Stage 1 was not arbitrary. Because the input datasets are in French while CIDOC CRM is defined in English, cross-lingual semantic retrieval is a hard requirement. To make an informed and reproducible choice, we systematically benchmarked four representative retrieval approaches across an archaeological dataset.
 
| Model | Type | Selected |
|-------|------|:--------:|
| BM25-Okapi | Lexical baseline | ✗ |
| all-mpnet-base-v2 | Monolingual dense | ✗ |
| LaBSE | Cross-lingual (translation) | ✗ |
| **paraphrase-multilingual-mpnet-base-v2** | **Multilingual paraphrase** | **✓** |
 
`paraphrase-multilingual-mpnet-base-v2` consistently outperformed all alternatives. It is the only model in the benchmark explicitly trained for semantic paraphrase alignment across 50+ languages, which is exactly the capability required to match a short French column label to a multi-sentence English ontological scope note. This benchmark was omitted from the submitted paper due to space constraints; full results are available in this repository.
 
→ [`docs/EMBEDDING_BENCHMARK.md`](docs/EMBEDDING_BENCHMARK.md)

## Environmental Impact
 
The computational cost of LLM-based ontology population is non-trivial, particularly for commercial cloud models. We measured the carbon footprint of all LLM inference runs using [EcoLogits](https://ecologits.ai/latest/). GPT-4o produces approximately **43.90 gCO₂eq** for the full evaluation, compared to **4.99 gCO₂eq** for Llama 4 Scout, a nine-fold difference for the same 22-request workload.
 
These measurements were omitted from the submitted paper due to space constraints. Full results and discussion are available in this repository.
 
→ [`docs/CARBON_EMISSIONS.md`](docs/CARBON_EMISSIONS.md)

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Historic Contributors (core developers first, then alphabetical order)

- [Ali Hariri (core developer)](https://www.lias-lab.fr/members/alihariri/), LIAS, ISAE-ENSMA
- [Mickaël Baron](https://www.lias-lab.fr/members/mickaelbaron/), LIAS, ISAE-ENSMA
- [Stéphane Jean](https://www.lias-lab.fr/members/stephanejean/), LIAS, Université de Poitiers