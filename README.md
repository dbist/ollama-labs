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
docker exec -it ollama ollama phi3:3.8b

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