---
title: "Qwen3.6 27B Quantization Across Engines: Size vs. KL Divergence"
date: 2026-08-09T14:12:01-06:00
draft: false
description: "A cross-engine KLD comparison of Qwen3.6 27B quants (GGUF, NVFP4, AWQ, AutoRound, FP8)."
summary: "Sixteen Qwen3.6 27B quantizations compared by loaded weight size, KL divergence, and top-1 agreement—including GGUF under llama.cpp and several quantization formats under vLLM."
categories:
  - Engineering
tags:
  - LLM Benchmarks
  - Quantization
  - Qwen
  - Research
showToc: true
TocOpen: false
---

There are plenty of KL-divergence benchmarks for GGUF models, but most of them compare one GGUF quant against another. I wanted to know how those quants stack up against other commonly used formats (especially NVFP4).

I tested 16 quantizations of Qwen3.6 27B: GGUF models in llama.cpp and the others in vLLM. At each token in the test set, I compared the quantized model's next-token probability distribution with that of an unquantized reference. The resulting KL divergence measures how far the quant has drifted from the original model; lower is better.

## Results at a glance

{{< plotly src="qwen3.6-27b-size-vs-kl.json" alt="Scatter plot comparing 16 Qwen3.6 27B quantizations by loaded weight size and KL divergence. GGUF models running in llama.cpp occupy most of the quality-size frontier. The lowest divergence is 0.0337 for Unsloth UD-Q8-K-XL at 33.31 GiB. Bartowski Q8_0 reaches 0.0402 at 27.11 GiB, while Qwen FP8 under vLLM reaches 0.1244 at 27.57 GiB. Around 17 to 19 GiB, Bartowski Q4_K_L and Unsloth UD-Q4-K-XL reach about 0.22, AWQ and NVIDIA NVFP4 reach about 0.28, AutoRound reaches 0.38, and one NVFP4-W4A4 model reaches 0.56." />}}

| Quant | Source / recipe | Engine | Loaded weights (GiB) | KL divergence | Top-1 agreement |
|---|---|---|---:|---:|---:|
| `uns_UD_IQ3_XXS` | Unsloth Dynamic 2.0 GGUF · `IQ3_XXS`-base weight mix | llama.cpp | 11.36 | 0.5931 | 85.35% |
| `bart_IQ3_XS` | Bartowski imatrix GGUF · `IQ3_XS` weight mix | llama.cpp | 12.60 | 0.5160 | 86.46% |
| `nvfp4_MTP_gguf` | michaelw9999 GGUF · custom mixed-tensor NVFP4 weights | llama.cpp | 15.07 | 0.4015 | 89.26% |
| `AutoRound_INT4` | Lorbus · symmetric `INT4` weights, group 128 | vLLM | 16.59 | 0.3831 | 89.48% |
| `uns_UD_Q4_K_XL` | Unsloth Dynamic 2.0 GGUF · `Q4_K`-base weight mix | llama.cpp | 16.67 | 0.2273 | 92.46% |
| `bart_Q4_K_L` | Bartowski imatrix GGUF · `Q4_K`, `Q8_0` embed/output | llama.cpp | 17.62 | 0.2218 | 92.41% |
| `NVFP4_Text_MTP` | sakamakismile · NVFP4 W4A4, group 16 | vLLM | 17.62 | 0.5636 | 86.27% |
| `AWQ_INT4` | cyankiwi · asymmetric `INT4` weights, group 32 | vLLM | 18.01 | 0.2776 | 91.27% |
| `NVFP4` | NVIDIA · NVFP4 W4A16 / FP8 W8A8 mixed | vLLM | 18.89 | 0.2807 | 91.00% |
| `uns_UD_Q5_K_XL` | Unsloth Dynamic 2.0 GGUF · `Q5_K`-base weight mix | llama.cpp | 18.94 | 0.1265 | 94.60% |
| `uns_NVFP4` | Unsloth · NVFP4 W4A4 / FP8 W8A8 mixed | vLLM | 20.29 | 0.3912 | 89.51% |
| `bart_Q6_K_L` | Bartowski imatrix GGUF · `Q6_K`, `Q8_0` embed/output | llama.cpp | 22.61 | 0.0565 | 96.62% |
| `uns_UD_Q6_K_XL` | Unsloth Dynamic 2.0 GGUF · `Q6_K`-base weight mix | llama.cpp | 24.22 | 0.0526 | 96.75% |
| `bart_Q8_0` | Bartowski GGUF · `Q8_0` weights | llama.cpp | 27.11 | 0.0402 | 97.37% |
| `qwen_FP8` | Qwen · block FP8 weights, dynamic FP8 activations | vLLM | 27.57 | 0.1244 | 94.74% |
| `uns_UD_Q8_K_XL` | Unsloth Dynamic 2.0 GGUF · `Q8_K`-base weight mix | llama.cpp | 33.31 | 0.0337 | 97.45% |

## What the results show

### Weight-only GGUFs have the best quality-size tradeoffs

GGUF results occupy most of the lower envelope of the chart. For almost every size, a GGUF running in llama.cpp has the lowest measured KL divergence among nearby weight sizes. The main factor here is likely the activation quantization - GGUFs don't quantize activations at all. Several vLLM checkpoints quantize weights, activations, and sometimes the KV cache.

### vLLM quants vary substantially

Quantizations of similar size do not preserve the reference distribution equally well. Particularly of note is the Sakamakismile NVFP4 (W4A4) quant, which has substantially higher KLD compared to similarly sized (and even smaller) quants.

The two conventional `Q4` GGUFs are consistent with each other. Bartowski `Q4_K_L` measures 0.2218 and Unsloth `UD_Q4_K_XL` measures 0.2273, with heavily overlapping intervals. AWQ and NVIDIA's mixed NVFP4 are also nearly tied at 0.2776 and 0.2807.

### The quant recipes

| Checkpoint | Weight quantization | Activation quantization | KV cache |
|---|---|---|---|
| `uns_UD_IQ3_XXS` | Dynamic 2.0, `IQ3_XXS` base; per-tensor type from calibration | none | none |
| `bart_IQ3_XS` | `IQ3_XS` imatrix mix | none | none |
| `nvfp4_MTP_gguf` | custom tensor mix on NVFP4 weights; RSF scale fitting on the `Q_K` tensors; MTP tensors NVFP4 | none | none |
| `AutoRound_INT4` | `INT4`, symmetric, group 128 | none | none |
| `uns_UD_Q4_K_XL` | Dynamic 2.0, `Q4_K` base; per-tensor type from calibration | none | none |
| `bart_Q4_K_L` | `Q4_K` imatrix mix | none | none |
| `NVFP4_Text_MTP` | NVFP4, group 16, static scales, all LM `Linear` | NVFP4, group 16, static (W4A4) | none |
| `AWQ_INT4` | `INT4`, asymmetric (int8 zero-point), group 32 | none | none |
| `NVFP4` | NVFP4 group 16 on `mlp.*` + `lm_head`; FP8 E4M3 on `self_attn.*` and `linear_attn.{in_proj_qkv,in_proj_z,out_proj}` | static FP8 on the FP8 group (W8A8) | static FP8 |
| `uns_UD_Q5_K_XL` | Dynamic 2.0, `Q5_K` base; per-tensor type from calibration | none | none |
| `uns_NVFP4` | NVFP4 group 16 on `mlp.{gate,up,down}_proj` in layers 0-55; FP8 E4M3 per-channel on `self_attn.*`, `linear_attn.*`, `lm_head`, and `mlp.*` in layers 56-63 | NVFP4 group 16 on the NVFP4 group (W4A4); dynamic per-token FP8 on the FP8 group (W8A8) | static FP8 |
| `bart_Q6_K_L` | `Q6_K` imatrix mix | none | none |
| `uns_UD_Q6_K_XL` | Dynamic 2.0, `Q6_K` base; per-tensor type from calibration | none | none |
| `bart_Q8_0` | uniform `Q8_0` | none | none |
| `qwen_FP8` | FP8 E4M3, 128×128 weight blocks | dynamic per-token FP8 (W8A8) | none |
| `uns_UD_Q8_K_XL` | Dynamic 2.0, `Q8_K` base; per-tensor type from calibration | none | none |

## What the KL number means

At every prompt position, the benchmark computes `D_KL(P_reference || P_quant)`: how much the quantized model's next-token distribution differs from the full-precision distribution. Zero means no measured change; larger values mean more of the reference distribution was displaced.

Both engines compute exact full-vocabulary softmax probabilities, but only the top 200 log probabilities per position are used. The benchmark solves for the minimum KL consistent with the two measured top-200 lists, their remaining probability budgets, and the fact that an unlisted quant token cannot exceed the quant's smallest reported probability, in order to get a lower bound on full-vocabulary KL.

The mean reference tail mass outside the top 200 was 0.0025 for both engines in this run. Top-1 agreement does not depend on the tail approximation and provides a complementary check.

## Methodology

Each quant was measured against a reference model in its own engine:

- GGUF quants were compared with a BF16 GGUF reference under llama.cpp.
- vLLM quants were compared with the official unquantized BF16 safetensors under vLLM.

> I also ran a subset of the same GGUF quants through vLLM. Their measured KL values stayed within roughly 0.01 of the llama.cpp values. Thus, while the cross-engine measurements do carry some amount of noise, it is much smaller than the differences measured in the benchmark.

I created my own dataset for the KL measurements, which ended up being 100 structured agentic tool-use conversations containing 182,306 tokens. Prompts range from 1,700 to 1,950 tokens.

> The dataset used heavily determines the overall KLD measured by these benchmarks. The low KLD numbers you may be used to seeing in other benchmarks are due to datasets like Wikipedia at low context being used. I did do runs on longer prompts from 10k to 30k tokens, but I found that it simply increased the overall KLD without much relative difference between quants.

Quantized checkpoints ran as shipped, including any declared compute dtype, activation quantization, or KV-cache scheme, in order to measure the true fidelity of each quant recipe.

Top-1 agreement is the fraction of positions at which the quantized model and its reference assign the highest probability to the same token.

The size measurement includes MTP/NextN layers and excludes KV/recurrent caches, activations, workspaces, CUDA graphs, runtime context, and unloaded multimodal components. It is not total serving memory. Take these measurements with a grain of salt, as they'll vary in actual deployment depending on your configuration.

## Practical takeaways

- Quantization format alone is not enough to predict quality. Look at the quantization recipe to determine if it fits your needs.
- Activation quantization can improve throughput on supported hardware, but this comes at the cost of quality.
- If quality per loaded GiB is the priority, the tested GGUF recipes provide the strongest tradeoffs.
- GGUF `Q5` for Qwen3.6 27B seems to be the sweet spot from the results.

## Final notes

KLD benchmarks may be able to show the relative differences in quantization quality, but this doesn't translate perfectly into real-world performance. The results should be read as comparisons between the tested quant recipes, not universal rankings of GGUF, AWQ, FP8, or NVFP4 as formats.
