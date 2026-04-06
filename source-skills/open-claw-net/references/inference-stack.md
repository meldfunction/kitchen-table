# Inference Stack — Ollama, llama.cpp, and Petals on Raspberry Pi

## The three layers

Every node runs all three. They serve different purposes:

| Tool | Purpose | When it's used |
|---|---|---|
| **Ollama** | Primary inference runtime | All personal agent tasks |
| **llama.cpp** | Low-level engine Ollama uses; also direct CLI | Fine-grained control, benchmarking |
| **Petals** | Distributed inference across all nodes | Heavy tasks routed to the collective pool |

---

## Ollama on Pi 5

Ollama is the simplest path. Install once, pull models, point OpenClaw at it.

```bash
curl -fsSL https://ollama.com/install.sh | sh

# Start (runs as a service after install)
sudo systemctl status ollama

# Pull your models
ollama pull phi3.5:3.8b-mini-instruct-q4_K_M   # 2.4GB
ollama pull llama3.2:1b-instruct-q8_0           # 1.3GB

# Test
ollama run phi3.5:3.8b-mini-instruct-q4_K_M "What's 2+2? Answer in one word."
```

### Ollama performance tuning for Pi

```bash
# Set number of threads (Pi 5 has 4 cores, leave 1 for OS)
export OLLAMA_NUM_PARALLEL=1
export OLLAMA_MAX_LOADED_MODELS=2  # keep both models in memory
export OLLAMA_NUM_THREAD=3

# Make persistent
echo 'OLLAMA_NUM_PARALLEL=1' | sudo tee -a /etc/environment
echo 'OLLAMA_MAX_LOADED_MODELS=2' | sudo tee -a /etc/environment
echo 'OLLAMA_NUM_THREAD=3' | sudo tee -a /etc/environment
```

### Keeping models in memory

By default Ollama unloads models after 5 minutes of inactivity. For an always-on
agent, you want your primary model always loaded:

```bash
# Keep model loaded indefinitely
curl http://localhost:11434/api/generate -d '{
  "model": "phi3.5:3.8b-mini-instruct-q4_K_M",
  "keep_alive": -1
}'
```

Put this in a cron job to run at boot: `@reboot curl http://localhost:11434/api/generate -d '{"model":"phi3.5:3.8b-mini-instruct-q4_K_M","keep_alive":-1}'`

---

## llama.cpp on Pi 5

llama.cpp gives you direct control and is the engine under Ollama. Build it once;
use it for benchmarking, fine-tuning workflows, and Petals.

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# Build with OpenBLAS for CPU optimization
cmake -B build \
  -DLLAMA_BLAS=ON \
  -DLLAMA_BLAS_VENDOR=OpenBLAS \
  -DCMAKE_BUILD_TYPE=Release

cmake --build build --config Release -j4
# Takes ~20 minutes on Pi 5

# Test
./build/bin/llama-cli \
  -m /path/to/phi-3.5-mini-q4_K_M.gguf \
  -p "Hello, are you working?" \
  -n 50
```

### Performance benchmarks (Pi 5, 8GB, Q4_K_M models)

| Model | Tokens/sec (prompt) | Tokens/sec (generation) | Notes |
|---|---|---|---|
| Llama 3.2 1B Q8 | ~25 t/s | ~10 t/s | Fastest, background tasks |
| Phi-3.5 Mini Q4 | ~8 t/s | ~4 t/s | Best quality/speed balance |
| Llama 3.2 3B Q4 | ~6 t/s | ~3 t/s | Good general purpose |
| Mistral 7B Q4 | ~3 t/s | ~1.5 t/s | Slow but capable |

These are Pi 5 (8GB) numbers with OpenBLAS and 3 threads. Actual numbers vary based
on prompt length and RAM pressure from other running processes.

---

## Petals — distributed inference across the group

Petals splits a large model's layers across multiple nodes. Each node hosts a portion
of the model and processes inference requests in sequence, passing activations between
nodes over the network.

### How it works

```
Your prompt
    ↓
Petals client (any node)
    ↓
Node 1 (layers 1–8)
    ↓
Node 2 (layers 9–16)
    ↓
Node 3 (layers 17–24)
    ↓
Node 4 (layers 25–32)
    ↓
Response
```

Each Pi hosts 8–10 layers of a 32-layer model. A 6-person group with Pi 5s (8GB each)
can collectively run Llama 3.1 8B at full precision — a model that wouldn't fit
comfortably on any single Pi.

### Setting up Petals on each node

```bash
# On each Pi, in a dedicated virtual environment
python3 -m venv ~/petals-env
source ~/petals-env/bin/activate
pip install petals

# Start serving (each node automatically finds the others via DHT)
python3 -m petals.cli.run_server \
  meta-llama/Meta-Llama-3.1-8B-Instruct \
  --host_maddrs "/ip4/0.0.0.0/tcp/31330" \
  --num_blocks 8 \
  --device cpu
```

Each node contributes 8 blocks (layers). With 6 nodes, all 48 blocks of the 8B model
are covered. If a node goes offline, Petals routes around it (degraded performance
but doesn't crash).

### Running inference against the Petals pool

```python
from transformers import AutoTokenizer
from petals import AutoDistributedModelForCausalLM

tokenizer = AutoTokenizer.from_pretrained("meta-llama/Meta-Llama-3.1-8B-Instruct")
model = AutoDistributedModelForCausalLM.from_pretrained(
    "meta-llama/Meta-Llama-3.1-8B-Instruct"
)

inputs = tokenizer("Analyze this situation:", return_tensors="pt")
outputs = model.generate(**inputs, max_new_tokens=200)
print(tokenizer.decode(outputs[0]))
```

### OpenClaw integration with Petals

Petals exposes an OpenAI-compatible REST API via a proxy. Run this on the gateway node:

```bash
python3 -m petals.cli.run_server \
  meta-llama/Meta-Llama-3.1-8B-Instruct \
  --host_maddrs "/ip4/0.0.0.0/tcp/31330" \
  --public_api \
  --public_api_port 8080
```

Then point OpenClaw at it:

```json
{
  "ai": {
    "modelOverrides": {
      "heavy": {
        "baseURL": "http://127.0.0.1:8080/v1",
        "model": "meta-llama/Meta-Llama-3.1-8B-Instruct"
      }
    }
  }
}
```

### Petals latency reality check

Inter-node latency adds up. With nodes in different homes connected via Tailscale:

- 3 nodes on same LAN: ~50–100ms per forward pass
- 6 nodes across internet (Tailscale): ~200–500ms per forward pass
- 200 generation tokens: 40–100 seconds total

This is acceptable for async tasks. It's not acceptable for real-time chat.

**The pattern that works:**
- Personal Ollama (local model) → interactive chat, responsive
- Petals collective → long research tasks, analysis, summarization (run in background, results delivered when done)

Configure OpenClaw to route long-context or `heavy` tasks to Petals,
and interactive conversation to your local Ollama. Your agent handles the routing
automatically based on the `modelOverrides` config.

---

## Keeping models up to date

New open-weight model releases happen frequently. When a better small model comes out,
you want to update all nodes simultaneously:

```bash
# Pull new model on all nodes via Ansible
ansible nodes -i inventory.ini -m shell \
  -a "ollama pull phi4-mini:3.8b-instruct-q4_K_M"

# Update OpenClaw config to use new model (push via Ansible)
ansible nodes -i inventory.ini -m lineinfile \
  -a "path=~/.openclaw/config.json regexp='phi3.5' line='phi4-mini:3.8b-instruct-q4_K_M'"

# Restart OpenClaw everywhere
ansible nodes -i inventory.ini -m shell -a "openclaw restart"
```

Monitor r/LocalLLaMA for new model releases. When a new small model beats Phi-3.5 Mini
on reasoning benchmarks, update the group within a week.
