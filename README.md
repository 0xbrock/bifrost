# Bifrost LLM Gateway

Local deployment of [Bifrost](https://github.com/maximhq/bifrost), a unified API gateway for routing LLM requests across multiple providers through a single OpenAI-compatible endpoint.

## Providers

| Provider | Type | Models |
|----------|------|--------|
| **Azure OpenAI** | Native | `gpt-4o`, `gpt-4o-mini` |
| **Azure Serverless (Llama)** | Custom (OpenAI-compat) | `llama-3.3-70b` |
| **Azure Serverless (Mistral)** | Custom (OpenAI-compat) | `mistral-large` |
| **Google Gemini** | Native | `gemini-2.5-flash`, `gemini-2.5-pro`, `gemini-2.5-flash-lite`, `gemini-2.5-flash-image`, `gemini-2.0-flash`, `gemini-2.0-flash-lite`, `gemini-1.5-flash`, `gemini-1.5-pro` |
| **LM Studio** | Custom (OpenAI-compat) | `google/gemma-3-4b`, `qwen/qwen3-vl-30b`, `qwen/qwen3-coder-30b`, `text-embedding-embeddinggemma-300m-qat`, `text-embedding-nomic-embed-text-v1.5` |

LM Studio models reflect whatever is currently loaded in the local LM Studio instance.

## Setup

### Prerequisites

- Docker and Docker Compose
- API keys for your cloud providers
- [LM Studio](https://lmstudio.ai/) running locally (for local model access)

### Quick Start

1. Copy the example environment file and fill in your API keys:

   ```sh
   cp .env.example .env
   ```

2. Start the gateway:

   ```sh
   docker compose up -d
   ```

3. Test a request:

   ```sh
   curl -X POST http://localhost:8080/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "gemini/gemini-2.5-flash",
       "messages": [
         {"role": "user", "content": "Hello!"}
       ]
     }'
   ```

## Usage

All requests go through `http://localhost:8080/v1/chat/completions` using the format `provider/model` for the model field.

### Examples

```sh
# Azure OpenAI
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "azure/gpt-4o", "messages": [{"role": "user", "content": "Hello!"}]}'

# Google Gemini
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "gemini/gemini-2.5-flash", "messages": [{"role": "user", "content": "Hello!"}]}'

# LM Studio (local)
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "lm-studio/google/gemma-3-4b", "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Network Access

The gateway binds to `0.0.0.0:8080`, making it accessible from other machines on the local network. Use the host machine's IP address:

```sh
curl -X POST http://<host-ip>:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "gemini/gemini-2.5-flash", "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Configuration

- `config/config.json` -- Provider definitions, model lists, and network settings
- `.env` -- API keys and endpoints (not committed to git)
- `docker-compose.yml` -- Container configuration

### Notes

- **Custom providers** use `custom_provider_config` with `base_provider_type: "openai"` and require `request_path_overrides` to specify API paths.
- **`base_url` in custom providers** must be hardcoded in `config.json`. The `env.` variable reference syntax does not work for `base_url` (only for key `value` fields).
- **LM Studio** connects from the Docker container to the host via `host.docker.internal:1234`.
- **Env var changes** require `docker compose up -d` (not just `restart`) to take effect, since `restart` does not re-read the `.env` file.

## Management

```sh
# Start
docker compose up -d

# Restart (picks up config.json changes only)
docker compose restart

# Recreate (picks up .env changes)
docker compose up -d

# Logs
docker compose logs -f bifrost

# Stop
docker compose down
```

## Web UI

Bifrost includes a built-in web UI at [http://localhost:8080](http://localhost:8080) for monitoring and configuration.
