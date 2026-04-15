# Carbon Emissions Report

This document reports the carbon footprint of the LLM inference runs conducted during the ASOS-CRM evaluation. These measurements were omitted from the DaWaK 2026 paper due to space constraints and are provided here in the interest of reproducibility and environmental transparency.

All emissions were estimated using [EcoLogits](https://ecologits.ai/latest/), a framework for measuring the environmental impact of LLM inference requests.

---

## Measurement Context

| Parameter | Value |
|-----------|-------|
| Tracking tool | EcoLogits |
| Emission unit | gCO₂eq (grams of CO₂ equivalent) |
| Electricity mix zone | France (FRA) |
| LLM requests (per model) | 22 |
| Scope | Full ASOS-CRM evaluation across all four datasets |

---

## Results

### Llama 4 Scout

| Parameter | Value |
|-----------|-------|
| Backend | Ollama |
| Provider | huggingface\_hub |
| Model | llama4:scout |
| LLM requests | 22 |
| Output tokens | 17,658 |
| Request latency | 4,589.80 s |
| Quantization bits | 4 |
| Estimated energy | 0.019445 kWh |
| **Est. CO₂ emissions** | **4.9872 gCO₂eq** |

### GPT-4o

| Parameter | Value |
|-----------|-------|
| Provider | OpenAI |
| Model | gpt-4o |
| LLM requests | 22 |
| Output tokens | 53,699 |
| Request latency | 1,133.03 s |
| Estimated energy | 0.107612 kWh |
| **Est. CO₂ emissions** | **43.8989 gCO₂eq** |

---

## Summary

| Model | LLM requests | Output tokens | Energy (kWh) | CO₂ (gCO₂eq) |
|-------|:------------:|:-------------:|:------------:|:-------------:|
| Llama 4 Scout | 22 | 17,658 | 0.019445 | **4.99** |
| GPT-4o | 22 | 53,699 | 0.107612 | **43.90** |

---

## Discussion

GPT-4o produces approximately nine times more CO₂ emissions than Llama 4 Scout (43.90 vs. 4.99 gCO₂eq) despite completing the same 22-request workload. This disparity is partly explained by output token volume — GPT-4o generated 53,699 tokens compared to 17,658 for Llama 4 Scout, a factor of approximately three — but the ratio in emissions (×9) substantially exceeds the ratio in output tokens (×3), reflecting the additional carbon overhead of large-scale cloud data centre infrastructure as modelled by EcoLogits.

These figures should be read in conjunction with annotation quality. ASOS-CRM with Llama 4 Scout achieves 85.3% precision and 73.2% recall at a footprint of approximately 5 gCO₂eq — roughly one ninth of the 44 gCO₂eq required by GPT-4o for a gain of approximately 6 percentage points in precision. For institutions operating under data governance or carbon budget constraints, local deployment with an open-weight model therefore represents the environmentally preferable configuration, provided that the modest reduction in annotation quality is acceptable for the target use case.