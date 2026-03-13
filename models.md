# Ollama Models for Pinova P1 Mini

Below is a **quick‑reference list of Ollama‑only models that run comfortably on a
Pinova P1 Mini PC** (Intel N100 / Celeron‑class CPU, 8 GB RAM, no dedicated GPU).

| # | Ollama model name | Approx. parameters | Recommended quantisation (size on disk) | Estimated RAM for inference* | Why it’s a good fit for the P1 Mini |
|---|-------------------|--------------------|------------------------------------------|------------------------------|-------------------------------------|
| 1 | `llama3:8b` | 8 B | `Q4_0` ≈ 4 GB | 4–5 GB | Modern Llama 3 base, decent instruction‑following ability while still fitting in 8 GB RAM. |
| 2 | `llama2:7b` | 7 B | `Q4_0` ≈ 3.5 GB | 3.5–4 GB | Very mature, well‑tested on CPUs; good for chat & Q&A. |
| 3 | `phi3:





\` | 3.8 B | `Q4_0` ≈ 2 GB | 2–2.5 GB | Smallest “Phi‑3” variant; fast response, low memory, decent reasoning. |
| 4 | `phi:2` | 2.7 B | `Q4_0` ≈ 1.8 GB | 1.8–2.2 GB | Tiny, ultra‑fast, works well for short prompts or code completion. |
| 5 | `gemma:2b` | 2 B | `Q4_0` ≈ 1.5 GB | 1.5–1.9 GB | Light‑weight, good for classification, summarisation, and simple chat. |
| 6 | `mistral:7b` | 7 B | `Q4_0` ≈ 3.8 GB | 3.8–4.3 GB | Strong general‑purpose model, still fits comfortably in 8 GB. |
| 7 | `tinyllama:1.1b` | 1.1 B | `Q4_0` ≈ 0.9 GB | 0.9–1.2 GB | Ultra‑small, great for embedded‑style utilities or when you need many concurrent instances. |
| 8 | `qwen2:0.5b` | 0.5 B | `Q4_0` ≈ 0.5 GB | 0.5–0.8 GB | The smallest Qwen 2 model; surprisingly capable for very short prompts. |
| 9 | `nomic-embed-text` | 2 B (embedding) | `Q4_0` ≈ 2 GB | 2 GB (embedding inference) | If you need vector embeddings for search / RAG, this runs in the same RAM budget. |
|10| `deepseek-coder:6.7b` | 6.7 B | `Q4_0` ≈ 3.5 GB | 3.5–4 GB | Optimised for code generation; still fits on an 8 GB system. |

\* **RAM estimate** includes the model weights plus a modest amount of working
memory for the prompt; leave ~1 GB headroom if you run other services.

---

## Running Ollama in Docker

```yaml
version: "3.8"

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ./ollama-data:/root/.ollama
    command: ["ollama", "serve"]
```

```bash
docker compose up -d
docker exec -it ollama ollama pull llama3:8b
curl http://localhost:11434/api/generate -d '{
  "model":"llama3:8b",
  "prompt":"Explain why the sky is blue in two sentences.",
  "stream":false
}'
```

---

### Tips for low‑spec CPUs

| Tip | Why it helps | Quick command |
|-----|---------------|---------------|
| **Use `Q4_0`/`Q5_1` quantisation** | 4‑bit weights shrink RAM/cpu cost with minimal quality loss. | `ollama pull llama3:8b-q4_0` |
| **Limit CPU threads** | Keeps the host responsive. | `-e OLLAMA_NUM_THREADS=4` |
| **Reduce context window** | Smaller `OLLAMA_MAX_CTX` uses less RAM per request. | `-e OLLAMA_MAX_CTX=2048` |
| **Run detached** | One background container uses memory once. | `docker compose up -d` |
| **Add swap space** | Prevents occasional OOMs on 8 GB machines. | `sudo fallocate -l 2G /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile` |

---

## TL;DR – model cheat sheet

| Use‑case | Best small pick | Full 8 GB free | Absolute fastest |
|----------|-----------------|---------------|------------------|
| Chat / Q&A | `llama3:8b` | `mistral:7b`/`llama2:7b` | `phi3:mini` |
| Code help | `deepseek-coder:6.7b` | `mistral:7b` | `phi:2` |
| Embeddings | `nomic-embed-text` | — | — |
| Tiny demos | `tinyllama:1.1b` | — | `qwen2:0.5b` |

All models live in the official Ollama registry—`ollama pull <model>` and you’re
ready.
