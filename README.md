# Ollama Labs

A Docker-based local LLM setup with Ollama, Open WebUI, and nginx reverse proxy.

## Quick Start

Copy the environment template and start the services:

```bash
cp .env.example .env
docker compose up -d
```

Access the services:
- **Open WebUI:** http://localhost:8282
- **Ollama API:** http://localhost/api/ (via nginx)

## Hardware Requirements

### Minimum Configuration
- **CPU:** 2+ cores
- **RAM:** 8 GB
- **Storage:** 30 GB (for small models)
- **OS:** Linux, macOS, or Windows with Docker

### Recommended Configuration
- **CPU:** 4+ cores
- **RAM:** 16 GB (supports 13B range models)
- **Storage:** 50+ GB (for multiple models)
- **GPU:** NVIDIA or Apple Silicon (optional, enables faster inference)

### Model-Specific Requirements
- **3-7B parameters:** 8 GB RAM, 4-8 GB storage
- **13-14B parameters:** 16 GB RAM, 8-14 GB storage
- **34B+ parameters:** 24+ GB RAM, 34+ GB storage

The default configuration (`docker-compose.yaml`) is tuned for 8GB systems with smaller models. Adjust `OLLAMA_NUM_THREADS`, `OLLAMA_CPUS`, and memory limits in `.env` for your hardware.

## Configuration

### Environment Variables
Copy `.env.example` to `.env` and customize:

```bash
# Resource Allocation
OLLAMA_CPUS=2.0              # CPU cores for Ollama
OLLAMA_MEMORY=4g             # Memory for Ollama
WEBUI_CPUS=1.0               # CPU cores for Open WebUI
WEBUI_MEMORY=1g              # Memory for Open WebUI
OLLAMA_NUM_THREADS=4         # CPU threads (match available cores)

# Ports & URLs
OLLAMA_PORT=127.0.0.1:11434  # Change to 0.0.0.0:11434 to expose externally (requires auth)
WEBUI_PORT=8282
NGINX_PORT=80

# Model Behavior
OLLAMA_KEEP_ALIVE=24h        # Keep models in memory for this duration
OLLAMA_MAX_CTX=2048          # Context window size (increase for longer conversations)
```

### Services

### Ollama
Local LLM inference engine.
- **Port:** 11434 (internal)
- **Environment:**
  - `OLLAMA_KEEP_ALIVE=24h` - Keep models loaded
  - `OLLAMA_NUM_THREADS=4` - CPU threads (adjust for your system)
  - `OLLAMA_MAX_CTX=2048` - Context window size

### Open WebUI
Web interface for Ollama.
- **Port:** 8282
- **Features:** Chat, model management, history

### Nginx
Reverse proxy for unified access.
- **Port:** 80
- Routes `/api/` to Ollama and `/` to Open WebUI

## Pull Models

```bash
# Pull a model
docker exec -it ollama ollama pull mistral:7b

# List available models
docker exec -it ollama ollama list

# Run a model directly
docker exec -it ollama ollama run mistral:7b "What is the capital of France?"
```

## Pre-commit Hooks

Set up automated checks before each commit:

```bash
pip install pre-commit
pre-commit install
```

This will:
- Validate YAML and JSON syntax
- Check for merge conflicts and secrets
- Verify docker-compose configuration
- Format code according to project standards

Run hooks manually: `pre-commit run --all-files`

## Docker Volumes

The setup uses two named volumes:
- **ollama_data** - Persists Ollama configuration and model metadata
- **open-webui** - Persists Open WebUI database and user settings

Stop containers without data loss: `docker compose down`
Remove all data (if needed): `docker compose down -v`

## Health Checks & Monitoring

All services include health checks that run every 30 seconds. View status:

```bash
docker compose ps
```

Services won't start dependent containers until they pass initial health checks (`start_period: 40s`).

## Security Notes

- **Local Access Only:** By default, ports bind to `127.0.0.1`. To expose externally, edit `.env` OLLAMA_PORT to `0.0.0.0:11434`
- **Add Authentication:** For external access, implement basic auth in nginx or use a VPN
- **Rate Limiting:** Nginx includes rate limiting (10 req/s for API, 30 req/s for WebUI)
- **Security Headers:** X-Frame-Options, X-Content-Type-Options, and XSS protection enabled

## Recommended Models

See [models.md](models.md) for model recommendations optimized for 8GB RAM systems.

## Child Safety

See [child-safe.md](child-safe.md) for guidance on making Ollama safe for children.

## File Reference

- **docker-compose.yaml** - Service definitions with health checks and resource limits
- **.env.example** - Environment variable template (copy to .env)
- **.pre-commit-config.yaml** - Automated commit checks
- **nginx.conf** - Reverse proxy routing with rate limiting
- **models.json** - Available models metadata
- **models.md** - Model recommendations and performance notes


## Intel N100 / CPU-Only Systems (e.g., Pinova P1 Mini)

If you're running on a **Pinova P1 Mini** or similar Intel N100/Celeron system without a dedicated GPU, Ollama will run in CPU mode. You'll see this in the logs:

```
inference compute id=cpu library=cpu compute="" name=cpu description=cpu
```

This is expected behavior. To optimize performance on CPU-only systems:

| Option | Description | How To |
|--------|-------------|--------|
| **Use smaller models** | Models with fewer parameters run faster on CPU | `ollama pull phi3:mini`, `gemma:2b`, `tinyllama:1.1b` |
| **Increase CPU threads** | Utilize all available CPU cores | Set `OLLAMA_NUM_THREADS=4` in `.env` (match your core count) |
| **Use quantized models** | 4-bit quantization reduces memory and speeds up inference | `ollama pull llama3:8b-q4_0`, `mistral:7b-q4_0` |
| **Reduce context window** | Smaller context = less RAM usage per request | Set `OLLAMA_MAX_CTX=2048` in `.env` |
| **Add swap space** | Prevents OOM errors on 8GB systems | `sudo fallocate -l 2G /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile` |

### Recommended Models for Intel N100

| Model | Size | Speed | Best For |
|-------|------|-------|----------|
| `phi3:mini` | 3.8B | ⚡⚡⚡ Fast | Chat, quick Q&A |
| `gemma:2b` | 2B | ⚡⚡⚡⚡ Very Fast | Simple tasks, classification |
| `tinyllama:1.1b` | 1.1B | ⚡⚡⚡⚡⚡ Fastest | Ultra-light demos |
| `llama3:8b-q4_0` | 8B | ⚡⚡ Moderate | Better quality, slower |
| `mistral:7b-q4_0` | 7B | ⚡⚡ Moderate | Good balance |

### Expected Performance on Intel N100

| Model | Tokens/Second | First Token Latency |
|-------|---------------|---------------------|
| `phi3:mini` | 5-8 t/s | ~2-3 seconds |
| `gemma:2b` | 8-12 t/s | ~1-2 seconds |
| `llama3:8b-q4_0` | 2-4 t/s | ~5-8 seconds |

> 💡 **Tip:** CPU inference is perfectly usable for chat and Q&A. Expect 2-5 tokens/second for 7-8B models. For faster responses, stick to models under 4B parameters.

### Troubleshooting CPU Mode

If you have an NVIDIA GPU but still see CPU inference:

1. **Install NVIDIA Container Toolkit:** See [GPU Support](#gpu-support) section
2. **Verify GPU detection:** `docker exec ollama nvidia-smi`
3. **Restart container:** `docker compose down && docker compose up -d`
4. **Check logs:** `docker logs ollama | grep -i gpu`
```

---

## Quick Command to Add It

If you have terminal access, run this from the `ollama-labs` directory:

```bash
cat >> README.md << 'EOF'

## Intel N100 / CPU-Only Systems (e.g., Pinova P1 Mini)

If you're running on a **Pinova P1 Mini** or similar Intel N100/Celeron system without a dedicated GPU, Ollama will run in CPU mode. You'll see this in the logs:

```
inference compute id=cpu library=cpu compute="" name=cpu description=cpu
```

This is expected behavior. To optimize performance on CPU-only systems:

| Option | Description | How To |
|--------|-------------|--------|
| **Use smaller models** | Models with fewer parameters run faster on CPU | `ollama pull phi3:mini`, `gemma:2b`, `tinyllama:1.1b` |
| **Increase CPU threads** | Utilize all available CPU cores | Set `OLLAMA_NUM_THREADS=4` in `.env` (match your core count) |
| **Use quantized models** | 4-bit quantization reduces memory and speeds up inference | `ollama pull llama3:8b-q4_0`, `mistral:7b-q4_0` |
| **Reduce context window** | Smaller context = less RAM usage per request | Set `OLLAMA_MAX_CTX=2048` in `.env` |
| **Add swap space** | Prevents OOM errors on 8GB systems | `sudo fallocate -l 2G /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile` |

### Recommended Models for Intel N100

| Model | Size | Speed | Best For |
|-------|------|-------|----------|
| `phi3:mini` | 3.8B | ⚡⚡⚡ Fast | Chat, quick Q&A |
| `gemma:2b` | 2B | ⚡⚡⚡⚡ Very Fast | Simple tasks, classification |
| `tinyllama:1.1b` | 1.1B | ⚡⚡⚡⚡⚡ Fastest | Ultra-light demos |
| `llama3:8b-q4_0` | 8B | ⚡⚡ Moderate | Better quality, slower |
| `mistral:7b-q4_0` | 7B | ⚡⚡ Moderate | Good balance |

### Expected Performance on Intel N100

| Model | Tokens/Second | First Token Latency |
|-------|---------------|---------------------|
| `phi3:mini` | 5-8 t/s | ~2-3 seconds |
| `gemma:2b` | 8-12 t/s | ~1-2 seconds |
| `llama3:8b-q4_0` | 2-4 t/s | ~5-8 seconds |

> 💡 **Tip:** CPU inference is perfectly usable for chat and Q&A. Expect 2-5 tokens/second for 7-8B models. For faster responses, stick to models under 4B parameters.

### Troubleshooting CPU Mode

If you have an NVIDIA GPU but still see CPU inference:

1. **Install NVIDIA Container Toolkit:** See [GPU Support](#gpu-support) section
2. **Verify GPU detection:** `docker exec ollama nvidia-smi`
3. **Restart container:** `docker compose down && docker compose up -d`
4. **Check logs:** `docker logs ollama | grep -i gpu`
EOF