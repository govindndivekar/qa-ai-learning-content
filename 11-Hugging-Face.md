# Hugging Face — Transformers, Datasets, Fine-Tuning, and Testing GenAI

> Subject #11 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: Python (file 04), AI fundamentals (file 01). Estimated time: ~45 hours.
> Target outcome: Use the Hugging Face stack professionally — Hub, `transformers`, `datasets`, `evaluate`, `peft`, `trl`, and Spaces — and apply it to QA of GenAI systems.

---

## 1. Overview & Learning Outcomes

- Navigate the Hugging Face Hub: models, datasets, Spaces, model cards, licenses.
- Use `transformers` pipelines for classification, NER, summarization, translation, QA, and text generation.
- Load and process data with `datasets` (splits, map, filter, cast).
- Fine-tune a model with `Trainer` and, where sensible, with PEFT / LoRA / QLoRA.
- Evaluate models with `evaluate` (BLEU, ROUGE, BERTScore, accuracy/F1) and with LLM-as-judge.
- Deploy inference: Inference API, Inference Endpoints, TGI, and local Spaces (Gradio).
- Apply Hugging Face tools to QA tasks: safety eval, regression eval, bias probing, data-quality checks.

---

## 2. Detailed Syllabus

1. The Hub: orgs, models, datasets, Spaces, collections — 2h
2. Authentication: CLI, `HF_TOKEN`, private repos — 1h
3. `transformers` pipelines — 3h
4. Tokenizers: Auto, Fast, BPE/WordPiece/SentencePiece — 3h
5. AutoModel / AutoModelFor* classes — 2h
6. `datasets` library: load, map, filter, cast, streaming — 3h
7. Training with `Trainer` — 4h
8. Parameter-efficient fine-tuning (`peft`): LoRA, QLoRA — 4h
9. `trl`: SFT, DPO basics — 2h
10. `evaluate` library and metrics — 2h
11. Gradio & Spaces — 2h
12. Inference API / Inference Endpoints / TGI — 3h
13. Diffusers (brief; image generation) — 1h
14. Optimum & ONNX Runtime (speedups) — 1h
15. LangChain + Hugging Face integrations — 2h
16. Safety & bias eval: WEAT, HELM, AI-Safety Benchmarks — 2h
17. Responsible AI: model cards, dataset cards, Licensing — 1h
18. Testing GenAI systems with HF — 3h
19. CI on Spaces; continuous evaluation patterns — 1h
20. Cost / latency / hardware planning on HF — 1h

---

## 3. Topics — Explanations with Examples

### 3.1 The Hugging Face Hub

**Concept.** The Hub (`huggingface.co`) hosts >1M models, datasets, and Spaces. Every repo has a README/model-card, versions (git-backed), and a discussion tab. Licenses vary wildly (MIT, Apache-2, Llama Community, CC-BY-NC, OpenRAIL) — always read the license before commercial use.

```bash
pip install huggingface_hub
huggingface-cli login   # paste HF_TOKEN
```

### 3.2 Pipelines

**Concept.** The `pipeline()` helper hides tokenization/model setup for common tasks.

```python
from transformers import pipeline

clf = pipeline("sentiment-analysis")
print(clf(["Playwright is great", "Flaky tests are the worst"]))

sum_ = pipeline("summarization", model="facebook/bart-large-cnn")
print(sum_("Long article here...", max_length=50, min_length=20))

gen = pipeline("text-generation", model="gpt2", device_map="auto")
print(gen("The tester said", max_new_tokens=30))
```

**Pitfalls.** Pipelines default to CPU unless you pass `device=0` or `device_map="auto"`. For production, instantiate models directly instead of pipelines.

### 3.3 Tokenizers

```python
from transformers import AutoTokenizer
tok = AutoTokenizer.from_pretrained("bert-base-uncased")
ids = tok("Hello world", return_tensors="pt")
print(ids.input_ids.shape, tok.decode(ids.input_ids[0]))
```

**Fast vs slow.** `use_fast=True` (Rust-backed) is default and much faster. Watch for `attention_mask`, `padding`, and `truncation` — always set them explicitly in production code.

### 3.4 AutoModel classes

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

name = "distilbert-base-uncased-finetuned-sst-2-english"
tok = AutoTokenizer.from_pretrained(name)
model = AutoModelForSequenceClassification.from_pretrained(name)

text = "This test framework is a joy."
batch = tok(text, return_tensors="pt", truncation=True)
with torch.no_grad():
    logits = model(**batch).logits
pred = logits.argmax(-1).item()
print(model.config.id2label[pred])
```

### 3.5 The `datasets` library

```python
from datasets import load_dataset
ds = load_dataset("imdb")
print(ds)
print(ds["train"][0])

small = ds["train"].shuffle(seed=42).select(range(2000))
small = small.map(lambda ex: {"len": len(ex["text"].split())})
print(small.filter(lambda ex: ex["len"] < 100))
```

**Streaming** (for huge corpora):

```python
ds = load_dataset("c4", "en", streaming=True)
for row in ds["train"].take(3): print(row["text"][:80])
```

### 3.6 Training with Trainer

```python
from transformers import (AutoTokenizer, AutoModelForSequenceClassification,
                          TrainingArguments, Trainer)
from datasets import load_dataset
import numpy as np, evaluate

tok = AutoTokenizer.from_pretrained("distilbert-base-uncased")
def prep(b): return tok(b["text"], truncation=True, padding="max_length", max_length=128)

ds = load_dataset("imdb").map(prep, batched=True)
metric = evaluate.load("accuracy")

def compute(p):
    preds = np.argmax(p.predictions, axis=-1)
    return metric.compute(predictions=preds, references=p.label_ids)

args = TrainingArguments(
    output_dir="out", learning_rate=2e-5, per_device_train_batch_size=16,
    num_train_epochs=1, evaluation_strategy="epoch", logging_steps=100,
    fp16=True, report_to="none",
)

model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased", num_labels=2)
trainer = Trainer(model=model, args=args,
                  train_dataset=ds["train"].select(range(4000)),
                  eval_dataset=ds["test"].select(range(1000)),
                  tokenizer=tok, compute_metrics=compute)
trainer.train()
trainer.evaluate()
```

### 3.7 PEFT / LoRA / QLoRA

**Concept.** Fine-tuning all billions of weights is impractical. LoRA freezes the base model and trains small low-rank adapter matrices. QLoRA additionally quantizes the base to 4-bit. Saves >10× memory.

```python
from peft import LoraConfig, get_peft_model, TaskType
config = LoraConfig(task_type=TaskType.SEQ_CLS, r=8, lora_alpha=16,
                    lora_dropout=0.05, target_modules=["q_lin","v_lin"])
model = get_peft_model(model, config)
model.print_trainable_parameters()  # e.g., 0.3% of total
```

Install: `pip install peft bitsandbytes accelerate`.

### 3.8 `trl`: SFT and DPO

**Concept.** `trl` (Transformer Reinforcement Learning) gives you `SFTTrainer` for supervised fine-tuning and `DPOTrainer` for Direct Preference Optimization (aligning a model to "good" > "bad" answers).

```python
from trl import SFTTrainer
from datasets import Dataset

data = Dataset.from_list([
    {"text": "### Q: What is QA?\n### A: Ensuring software meets quality standards."},
])
trainer = SFTTrainer(model="gpt2", train_dataset=data, args=args, dataset_text_field="text")
trainer.train()
```

### 3.9 `evaluate`

```python
import evaluate
acc = evaluate.load("accuracy")
f1  = evaluate.load("f1")
bleu = evaluate.load("bleu")
rouge = evaluate.load("rouge")

print(acc.compute(predictions=[0,1,1], references=[0,1,0]))
print(bleu.compute(predictions=["hello world"], references=[["hello world"]]))
```

**For LLM outputs:** BLEU/ROUGE are weak; prefer `BERTScore` for semantic match or LLM-as-judge.

### 3.10 Gradio & Spaces

```python
import gradio as gr
from transformers import pipeline

clf = pipeline("sentiment-analysis")
def predict(text): return clf(text)[0]

demo = gr.Interface(fn=predict, inputs="text", outputs="json", title="Sentiment")
demo.launch()
```

Push to a Space: `huggingface-cli repo create my-space --type space`, push with git; Space auto-builds.

### 3.11 Inference API / Endpoints / TGI

**Three deployment options.**

- **Inference API** (serverless, free with rate limits) — good for demos.
- **Inference Endpoints** (dedicated, paid) — production SaaS.
- **Text Generation Inference (TGI)** — self-hosted container for high-throughput LLMs.

```python
import requests, os
API_URL = "https://api-inference.huggingface.co/models/gpt2"
headers = {"Authorization": f"Bearer {os.environ['HF_TOKEN']}"}
r = requests.post(API_URL, headers=headers, json={"inputs":"Once upon a time"})
print(r.json())
```

### 3.12 Diffusers (image)

```python
from diffusers import StableDiffusionPipeline
import torch
pipe = StableDiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5", torch_dtype=torch.float16).to("cuda")
img = pipe("a QA engineer debugging a Playwright test").images[0]
img.save("out.png")
```

### 3.13 Optimum / ONNX

**Concept.** `optimum` exports transformers to ONNX, TensorRT, OpenVINO. Typically 2–4× speedups on CPU, more on specialized hardware.

```bash
pip install optimum[onnxruntime]
optimum-cli export onnx --model distilbert-base-uncased out/
```

### 3.14 LangChain + HF

```python
from langchain_huggingface import HuggingFacePipeline
from transformers import pipeline
llm = HuggingFacePipeline(pipeline=pipeline("text-generation", model="gpt2"))
print(llm.invoke("Explain QA in one line:"))
```

### 3.15 Safety and bias evaluation

**Approach.**

- **Static probes.** Known benchmark sets: WinoBias, BBQ, TruthfulQA, ToxiGen.
- **Red-team prompts.** Categorize: jailbreak, PII extraction, harmful content, bias, prompt injection.
- **Metrics.** Refusal rate, harmful-content rate per category, policy-compliance score.

Libraries: `evaluate`, `lm-evaluation-harness` (EleutherAI), `DeepEval`, `promptfoo`.

### 3.16 Testing GenAI systems with HF

A minimum eval harness for a RAG or LLM feature:

1. **Golden set** — 50–200 Q/A with citations.
2. **Quality metrics** — exact match, semantic similarity (BERTScore), LLM-as-judge rubric.
3. **Safety metrics** — refusal rate per category.
4. **Latency/cost** — p50/p95/p99.
5. **Drift** — re-run weekly, alert on >5% quality drop.

Wire it up with `datasets` (load JSONL), `evaluate` (metrics), and pytest for CI gating.

### 3.17 CI and continuous evaluation

Spaces can run scheduled jobs via GitHub Actions. Pattern: a private Space runs the golden-set eval nightly, posts a score to a Slack webhook, fails the build if below threshold.

### 3.18 Cost / hardware planning

- Inference on a 7B model in fp16 needs ~15 GB VRAM; in int4 ~5 GB.
- T4 (free Colab) is fine for demos; A10G/A100 for production serving.
- For batch eval runs, a spot A10G for 1 hour < $1 — often cheaper than hosted API.

---

## 4. Exercises

### Beginner
- **A1.** Install `transformers`; run 3 different pipelines (`sentiment-analysis`, `ner`, `summarization`) on sample text.
- **A2.** Load `imdb` with `datasets`; print first 5 rows; compute label distribution.
- **A3.** Create a Hugging Face account; push a README-only model repo; pull it back.

### Intermediate
- **B1.** Fine-tune DistilBERT on IMDB (4k train, 1k eval) with `Trainer`; achieve >88% accuracy.
- **B2.** Apply LoRA to the same problem; show <1% trainable params and comparable accuracy.
- **B3.** Build a Gradio Space for a summarizer; deploy publicly.
- **B4.** Use `evaluate` to compute BLEU and ROUGE on a translation model output.

### Advanced
- **C1.** Golden-set eval harness: 100 Q/A; semantic score + LLM-as-judge; stored as a JSONL; pytest gate.
- **C2.** Safety probe suite: 30 prompts across 5 categories; automated refusal-rate report across 3 models.
- **C3.** Export a fine-tuned model to ONNX with `optimum`; benchmark 2× on CPU.
- **C4.** DPO alignment drill on a tiny synthetic preference dataset with `trl`.

### Capstone — "GenAI Evaluation Workbench"

Python project + a companion Gradio Space that:
1. Ingests a golden-set JSONL (inputs, references, categories).
2. Runs any HF-hosted or local Ollama model against it.
3. Scores with three metrics: BERTScore, LLM-as-judge (rubric configurable), and refusal rate for safety probes.
4. Writes a dashboard (plotly) to Space; a CLI flag drops a JUnit XML for CI.
5. CI gate: fail if overall score drops >5% vs previous run.

Deliver a README explaining evaluation philosophy, trade-offs, and how it maps to ISTQB CT-GenAI testing requirements.

---

## 5. Additional Reading & Resources

### Official
- Hugging Face Docs — <https://huggingface.co/docs>
- `transformers` — <https://huggingface.co/docs/transformers/>
- `datasets` — <https://huggingface.co/docs/datasets/>
- `peft` — <https://huggingface.co/docs/peft/>
- `trl` — <https://huggingface.co/docs/trl/>
- `evaluate` — <https://huggingface.co/docs/evaluate/>
- `optimum` — <https://huggingface.co/docs/optimum/>
- Inference Endpoints — <https://huggingface.co/inference-endpoints>
- Text Generation Inference — <https://github.com/huggingface/text-generation-inference>

### Courses
- HF NLP Course — <https://huggingface.co/learn/nlp-course>
- HF Deep RL Course — <https://huggingface.co/learn/deep-rl-course>
- HF Audio Course — <https://huggingface.co/learn/audio-course>
- HF Agents Course — <https://huggingface.co/learn>

### Books
- *Natural Language Processing with Transformers* (Tunstall, von Werra, Wolf) — O'Reilly.
- *Hands-On Large Language Models* (Alammar, Grootendorst).

### YouTube / Blogs
- Hugging Face YouTube channel.
- Julien Chaumond & Thomas Wolf public talks.
- Sebastian Raschka's "Ahead of AI" newsletter.

---

## 6. Index

- AutoModel — §3.4
- BERTScore — §3.9, §3.16
- Datasets library — §3.5
- Diffusers — §3.12
- DPO — §3.8
- evaluate library — §3.9
- fine-tuning with Trainer — §3.6
- Gradio — §3.10
- Hub authentication — §3.1
- Inference API / Endpoints — §3.11
- LoRA — §3.7
- model cards — §3.1, §3.17
- ONNX / Optimum — §3.13
- PEFT — §3.7
- pipelines — §3.2
- QLoRA — §3.7
- safety evaluation — §3.15
- SFT — §3.8
- Spaces — §3.10, §3.17
- streaming datasets — §3.5
- TGI — §3.11
- tokenizers — §3.3
- Trainer — §3.6
- trl — §3.8

---

## 7. References

1. Hugging Face Documentation — <https://huggingface.co/docs>
2. *Attention Is All You Need*, Vaswani et al., 2017.
3. LoRA paper — Hu et al., 2021.
4. QLoRA paper — Dettmers et al., 2023.
5. BERTScore paper — Zhang et al., 2019.
6. *Natural Language Processing with Transformers*, O'Reilly.
7. Hugging Face NLP Course — <https://huggingface.co/learn/nlp-course>
8. EleutherAI lm-evaluation-harness — <https://github.com/EleutherAI/lm-evaluation-harness>
9. ISTQB CT-GenAI Syllabus v1.0 — <https://www.istqb.org/>
10. promptfoo — <https://promptfoo.dev/>

---

*End of Subject 11 — Hugging Face.*
