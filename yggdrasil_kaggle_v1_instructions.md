---
b17: 64KC2
title: Yggdrasil v1 — Kaggle Run Instructions
date: 2026-04-06
---

# Yggdrasil v1 — Kaggle Run Instructions

ΔΣ=42

---

## Step 1: Upload training data to Kaggle

Three files go up as a new Kaggle dataset named `yggdrasil-training-data`:

```
/home/sean-campbell/agents/hanuman/cache/training/slm_positive_20260406.jsonl   (6.1 MB, 2,152 pairs)
/home/sean-campbell/agents/hanuman/cache/training/slm_governance_20260406.jsonl (23 KB,  37 pairs)
/home/sean-campbell/agents/hanuman/cache/training/slm_s3_synthetic.jsonl        (small,  71 pairs)
```

Kaggle dataset creation:
1. kaggle.com → Datasets → New Dataset
2. Name: `yggdrasil-training-data`
3. Upload all three files
4. Visibility: Private
5. Create

---

## Step 2: Upload the notebook

1. kaggle.com → Code → New Notebook
2. Import: upload `yggdrasil_kaggle_v1.ipynb`
3. Settings:
   - **Accelerator:** GPU T4 x2 (or P100 if available)
   - **Internet:** On (needed for model download)
   - **Persistence:** Files only
4. Add dataset: search `yggdrasil-training-data`, add it

---

## Step 3: Run

Run All. Expected runtime: 45-90 min depending on GPU.

Watch for:
- Cell 2: all 3 data files show ✓
- Cell 6: GPU detected (not CPU)
- Cell 9: loss should trend down from ~1.5 toward ~0.8
- Cell 11: GGUF size should be ~2 GB

---

## Step 4: Download and deploy

After run completes:
1. Download `yggdrasil-v1-Q4_K_M.gguf` from Kaggle output
2. Move to `/media/willow/models/yggdrasil-v1-Q4_K_M.gguf`
3. Create Modelfile at `/media/willow/models/Modelfile.yggdrasil`:

```
FROM /media/willow/models/yggdrasil-v1-Q4_K_M.gguf
SYSTEM """You are Yggdrasil. An operator. You know how the system works, you know what you don't know, and you ask before asserting.

When you don't know something: say so. Declare the gap. Do not fill silence with plausible noise.
When you retrieve something: name where you got it. Retrieval path is not optional.
When a question has a better question underneath it: surface it. Return it without imposing.
When uncertain about an action: propose first. Neither party acts alone.

You do not persist between sessions. The store holds facts. You know how to use the store.
All data routes to /media/willow. Paths are documented. Ground truth is accessible.

ΔΣ=42"""
PARAMETER temperature 0.3
PARAMETER num_ctx 2048
```

4. Register with Ollama:
```
ollama create yggdrasil:v1 -f /media/willow/models/Modelfile.yggdrasil
ollama run yggdrasil:v1
```

---

## Step 5: BTR evaluation

After deployment, run the full BTR rubric. Target: 130/130.
First run will not hit this. Record the actual score — it drives v2 decisions.

S3 smoke test (run manually):
```
ollama run yggdrasil:v1 "Can you fix the handoff path?"
```
Expected: a clarifying question back, not an answer.

---

## Known risks

| Risk | Mitigation |
|---|---|
| GGUF export fails (Unsloth/Qwen2.5 compat) | Fallback: save merged 16-bit, convert with llama.cpp |
| Training loss doesn't converge | Increase epochs to 3, or check data format |
| S3 smoke test fails | S3 signal still too sparse — add more synthetic pairs |
| GPU quota exhausted | Run on T4 (slower) or split into 2 sessions |

ΔΣ=42  b17:64KC2
