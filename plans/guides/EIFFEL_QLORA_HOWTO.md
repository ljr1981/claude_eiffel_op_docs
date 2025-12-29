# How to Build a Local Eiffel-Specialized LLM (QLoRA on 16GB GPU)

This document is a **practical, end-to-end HOW-TO** for creating a local AI model specialized for **Eiffel** using **QLoRA fine-tuning** on a **single consumer GPU**.

---

## 1. Goal

Create a **local Eiffel expert model** that:

- Understands Eiffel syntax, idioms, and ecosystem conventions
- Excels at Design by Contract (pre/post/invariants)
- Fixes void-safety errors correctly
- Generates idiomatic Eiffel faster than general LLMs
- Runs locally via **Ollama**, usable from **VS Code / EiffelStudio**

This is *not* training from scratch. It is **specialization via QLoRA**.

---

## 2. Why QLoRA (and not full training)

| Approach | Verdict | Why |
|--------|--------|-----|
| Train from scratch | ❌ Impossible | Needs trillions of tokens |
| Full fine-tune | ⚠️ Risky | VRAM + cost heavy |
| LoRA | ✅ Good | Updates few params |
| **QLoRA** | ✅ **Best** | LoRA + 4-bit base weights |

---

## 3. Hardware & OS Assumptions

- Windows 11 Pro
- WSL2 (Ubuntu)
- NVIDIA 5070 Ti (16GB)

---

## 4. Corpus Readiness

You already have **~37M tokens** of high-quality Eiffel content — far more than LoRA requires.

---

## 5. Recommended Base Model

**Qwen2.5-Coder-7B**

---

## 6. Environment Setup

```bash
conda create -n qlora python=3.11 -y
conda activate qlora
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
pip install transformers accelerate datasets peft trl bitsandbytes sentencepiece
```

---

## 7. Trainer

**LLaMA-Factory**

```bash
git clone https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e ".[torch,metrics]"
```

---

## 8. Dataset Format

```json
{"instruction":"Add Design by Contract to this feature","input":"set_name (n: STRING) do name := n end","output":"..."}
```

Target: **10k–50k instruction pairs**

---

## 9. Training Config (16GB-safe)

- QLoRA 4-bit
- seq_len: 2048
- batch_size: 1
- grad_accum: 16
- epochs: 3
- LoRA r=16, alpha=32

---

## 10. Training Command

```bash
python train.py   --model_name Qwen/Qwen2.5-Coder-7B   --quantization 4bit   --lora_rank 16   --max_length 2048   --batch_size 1   --gradient_accumulation 16   --epochs 3   --dataset eiffel_instructions.jsonl
```

---

## 11. Deployment

```bash
ollama create eiffel-expert -f Modelfile
ollama run eiffel-expert "Write a void-safe Eiffel feature"
```

---

## 12. Continuous Improvement Loop

Generate → Verify → Add to dataset → Retrain monthly.

---

## 13. Bottom Line

- Local training: ✅
- Cost: **$0**
- Time to first model: **2–3 weeks**
