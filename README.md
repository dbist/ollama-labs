# Ollama Labs

A Docker-based local LLM setup with Ollama, Open WebUI, and nginx reverse proxy.

## Quick Start

```bash
docker compose up -d
```

Access the services:
- **Open WebUI:** http://localhost:8282
- **Ollama API:** http://localhost/api/ (via nginx)

## Services

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
docker exec -it ollama ollama phi3:3.8b

# List available models
docker exec -it ollama ollama list

# Run a model directly
docker exec -it ollama ollama run mistral:7b "What is the capital of France?"
```

## Recommended Models

See [models.md](models.md) for model recommendations optimized for 8GB RAM systems.

## Child Safety

See [child-safe.md](child-safe.md) for guidance on making Ollama safe for children.

## Configuration

- **docker-compose.yaml** - Service definitions
- **nginx.conf** - Reverse proxy routing
- **models.json** - Available models metadata
- **models.md** - Model recommendations and performance notes