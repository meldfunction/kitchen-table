# Open Models — Selection, Licenses, and Fine-tuning

## The rule: if you can't download it and run it offline, it's not open

"Open" AI is a marketing term. For this stack, open means:
1. Model weights are publicly downloadable
2. You can run inference without an internet connection
3. The license permits your use case (personal + small group)

---

## Recommended models by use case (Pi 5, 8GB RAM)

### Personal agent / everyday assistant

**Phi-3.5 Mini Instruct (3.8B, Q4_K_M)**
Best reasoning-per-RAM-dollar in its class. Handles coding, writing, analysis, and
planning well. Microsoft's training choices for this model size are genuinely impressive.
License: MIT (fully open, commercial use permitted)
Pull: `ollama pull phi3.5:3.8b-mini-instruct-q4_K_M`
RAM: ~2.4GB | Speed: ~3–5 tok/s on Pi 5

**Llama 3.2 3B Instruct (Q4_K_M)**
Meta's small model, excellent instruction following. Slightly behind Phi-3.5 on
reasoning but strong on conversational tasks.
License: Llama 3.2 Community License (free for most uses under 700M MAU)
Pull: `ollama pull llama3.2:3b-instruct-q4_K_M`
RAM: ~2.0GB | Speed: ~4–6 tok/s on Pi 5

### Background tasks / heartbeats / subagents

**Llama 3.2 1B Instruct (Q8_0)**
Use this for everything OpenClaw runs in the background. It's fast enough for
heartbeats, light enough to not crowd out your main model, and smart enough for
simple classification and routing tasks.
Pull: `ollama pull llama3.2:1b-instruct-q8_0`
RAM: ~1.3GB | Speed: ~8–12 tok/s on Pi 5

### Coding tasks

**Qwen2.5 Coder 3B (Q4_K_M)**
Purpose-built for code. Outperforms larger general models on coding benchmarks at
this parameter count. If your group does software development, this is your coding
assistant.
License: Apache 2.0 (fully open)
Pull: `ollama pull qwen2.5-coder:3b-instruct-q4_K_M`
RAM: ~2.0GB | Speed: ~4–5 tok/s on Pi 5

### Collective pool / Petals (larger models, slower, shared)

**Llama 3.1 8B Instruct**
The workhorse. A 6-person Pi cluster running Petals can host this at full precision.
Noticeably more capable than 3B models for complex reasoning, longer context, and
nuanced instructions.
License: Llama 3.1 Community License
Petals model ID: `meta-llama/Meta-Llama-3.1-8B-Instruct`

**Mistral 7B Instruct v0.3**
Alternative to Llama 3.1 8B. Apache 2.0 license (more permissive). Similar capability.
Good choice if license terms matter for your use case.
License: Apache 2.0
Petals model ID: `mistralai/Mistral-7B-Instruct-v0.3`

---

## License summary

| Model family | License | Commercial use | Distribution |
|---|---|---|---|
| Phi-3/3.5 (Microsoft) | MIT | ✅ Yes | ✅ Yes |
| Gemma 2 (Google) | Gemma | ✅ Yes (with limits) | ⚠️ Restricted |
| Mistral / Mixtral | Apache 2.0 | ✅ Yes | ✅ Yes |
| Qwen2.5 (Alibaba) | Apache 2.0 | ✅ Yes | ✅ Yes |
| Llama 3.x (Meta) | Llama Community | ✅ Yes (<700M MAU) | ⚠️ Restricted |
| DeepSeek | DeepSeek License | ⚠️ Check terms | ⚠️ Restricted |

For a private friend group, all of the above are usable. The license differences matter
primarily if you're redistributing software that includes the weights.

---

## Where to download models

**Hugging Face Hub**
The primary repository for open-weight models. Every model above is available here.
Use the GGUF format for llama.cpp and Ollama compatibility.
→ https://huggingface.co/models

**Ollama Library**
Curated list with pre-quantized models, pull commands, and memory requirements.
The easiest path — `ollama pull` handles download and format conversion.
→ https://ollama.com/library

**LM Studio model catalog**
Alternative browser-based download interface with visual VRAM/RAM requirements.
Useful for exploring before committing to a download.
→ https://lmstudio.ai/models

---

## Fine-tuning your own model on group data

Fine-tuning creates a model that knows your group's specific context, terminology,
and preferences. A 3B model fine-tuned on your group's notes, documentation, and
workflows will outperform a 70B model on tasks specific to your domain.

### What you need

- A Pi 5 (8GB) or any machine with 8GB+ RAM
- Unsloth (memory-efficient fine-tuning framework)
- A dataset of your group's text (notes, docs, Slack/Matrix exports, etc.)
- ~4–12 hours of compute time per training run

### Dataset preparation

Format your training data as instruction-response pairs:

```json
[
  {
    "instruction": "What's our deployment process for the API?",
    "response": "Our API deploys via the deploy.sh script in /scripts. Run it with --env prod for production. Always run tests first with npm test. Requires VPN to be active."
  },
  {
    "instruction": "Who owns the database migrations?",
    "response": "Database migrations are owned by the backend team. Alice is the primary contact. All migrations go in /db/migrations and must be reviewed before merging."
  }
]
```

The more specific to your actual context, the better. Generic instructions add noise.

### Training with Unsloth

```bash
pip install unsloth

python3 << 'EOF'
from unsloth import FastLanguageModel
from datasets import Dataset
import json

# Load base model
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Phi-3.5-mini-instruct",
    max_seq_length=2048,
    load_in_4bit=True,  # essential for Pi (reduces RAM)
)

# Load your dataset
with open("group-knowledge.json") as f:
    data = json.load(f)

dataset = Dataset.from_list(data)

# Fine-tune
trainer = FastLanguageModel.get_peft_model(model, ...)
trainer.train()

# Save
model.save_pretrained("./our-group-model")
tokenizer.save_pretrained("./our-group-model")
EOF
```

Convert to GGUF for Ollama:
```bash
cd llama.cpp
python3 convert_hf_to_gguf.py ../our-group-model --outfile ../our-group-model.gguf
./build/bin/llama-quantize ../our-group-model.gguf ../our-group-model-q4.gguf Q4_K_M

# Import into Ollama
cat > Modelfile << 'EOF'
FROM ./our-group-model-q4.gguf
SYSTEM "You are the group assistant. You know our team's context, tools, and workflows."
EOF
ollama create our-group-model -f Modelfile
```

### When fine-tuning is worth it

Fine-tuning is worth the effort when:
- Your group has a specific domain (technical field, company, project)
- You find yourselves repeatedly explaining the same context to the base model
- The base model doesn't know your terminology or conventions

Fine-tuning is not worth it when:
- You just want a better general assistant (get a bigger model instead)
- Your knowledge base has fewer than ~500 high-quality examples
- Your context changes frequently (a fine-tuned model is a snapshot)

The lightweight alternative: a detailed SOUL.md + CLAUDE.md (or their OpenClaw
equivalents) with your group's context embedded. This works surprisingly well and
requires no training time.
