# Aura AI – Prototype V1 Weight File Documentation

> Developed by **Nexora.ai**, Aura AI is an experimental LLM designed for intelligent interaction and multimodal reasoning.  
> **This documentation covers the weight structure and configuration details of the first prototype (v1)**.

---

## New Fields in `config.json`

- **model_type**: Specifies the model type, now set to `aura_v1`.
- **num_nextn_predict_layers**: Indicates the number of Multi-Token Prediction (MTP) modules. The prototype v1 includes **1 MTP module**.
- **quantization_config**: Describes the configuration for FP8 quantization (used for performance optimization).

---

## Weight Structure Overview

The Aura AI prototype weight file consists of two main components: **Main Model Weights** and **MTP Modules**.

### 1. Main Model Weights

- **Composition**:
  - Includes embedding layers and 61 Transformer hidden layers.
- **Parameter Count**:
  - Total parameters: **671B**
  - Activation parameters: **36.7B** (including 0.9B for the embedding and 0.9B for the output head)

#### Structural Details

- **Embedding Layer**:
  - `model.embed_tokens.weight`
- **Transformer Layers**:
  - `model.layers.0` through `model.layers.60`
- **Output Layer**:
  - `model.norm.weight`
  - `lm_head.weight`

### 2. Multi-Token Prediction (MTP) Module

- **Composition**:
  - Includes an extra transformer layer and associated normalization and projection modules.
- **Parameter Count**:
  - Parameters: **11.5B unique**, excluding the shared 0.9B Embedding and 0.9B output Head.
  - Activation parameters: **2.4B** (shared values included)

#### Structural Details

- **embed_tokens**: Shared with the main model.
- **enorm & hnorm**: RMSNorm layers for speculative decoding.
- **eh_proj**: Projection parameters.
- **Extra Transformer Layer**:
  - `model.layers.61.self_attn` and `model.layers.61.mlp`
- **shared_head**: Shared with main model output head.

---

## Loading Rules

- **Main Model**: Controlled via the `num_hidden_layers` value in `config.json`.
- **MTP Module**: Controlled via the `num_nextn_predict_layers` value.
  - If `num_hidden_layers = 61` and `num_nextn_predict_layers = 1`, the MTP layer uses index `61`.

---

## FP8 Weight Format

Aura AI v1 supports FP8 quantization with efficient memory and compute usage via 128x128 block scaling.

### FP8 Configuration

```json
"quantization_config": {
  "activation_scheme": "dynamic",
  "fmt": "e4m3",
  "quant_method": "fp8",
  "weight_block_size": [128, 128]
}
