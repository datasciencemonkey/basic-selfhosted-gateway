# LiteLLM Proxy for Databricks

A simple LiteLLM proxy configuration to access Databricks model serving endpoints via an OpenAI-compatible API.

## Setup

1. Install LiteLLM:
```bash
pip install 'litellm[proxy]'
```

2. Set environment variables:
```bash
export DATABRICKS_API_BASE="https://your-workspace.cloud.databricks.com/serving-endpoints"
export DATABRICKS_TOKEN="your-databricks-token"
```

3. Start the proxy:
```bash
litellm --config litellm_config.yaml
```

## Usage

```bash
curl -X POST 'http://localhost:4000/chat/completions' \
  -H 'Content-Type: application/json' \
  -H 'x-litellm-api-key: sk-1234' \
  -d '{
    "model": "databricks-gpt-5-1",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Files

- `litellm_config.yaml` - Proxy configuration
- `mygatewaytodatabricks.md` - Architecture diagram

