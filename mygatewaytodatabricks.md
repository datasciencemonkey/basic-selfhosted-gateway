# LiteLLM Gateway to Databricks

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              YOUR APPLICATIONS                               │
│                                                                             │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│   │  Python  │    │   cURL   │    │   Node   │    │  Any App │             │
│   │  Client  │    │ Requests │    │    App   │    │ (OpenAI) │             │
│   └────┬─────┘    └────┬─────┘    └────┬─────┘    └────┬─────┘             │
│        │               │               │               │                    │
│        └───────────────┴───────────────┴───────────────┘                    │
│                                │                                            │
│                    OpenAI-compatible API                                    │
│                      POST /chat/completions                                 │
│                                │                                            │
└────────────────────────────────┼────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LITELLM PROXY SERVER                                │
│                         http://0.0.0.0:4000                                 │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        litellm_config.yaml                          │   │
│   │  ┌───────────────────────────────────────────────────────────────┐  │   │
│   │  │  model_list:                                                  │  │   │
│   │  │    - model_name: databricks-gpt-5-1                           │  │   │
│   │  │      litellm_params:                                          │  │   │
│   │  │        model: databricks/databricks-gpt-5-1                   │  │   │
│   │  │        api_base: $DATABRICKS_API_BASE                         │  │   │
│   │  │        api_key: $DATABRICKS_TOKEN                             │  │   │
│   │  └───────────────────────────────────────────────────────────────┘  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   Features:                                                                 │
│   ✓ OpenAI-compatible endpoint    ✓ Cost tracking                          │
│   ✓ API key management            ✓ Rate limiting                          │
│   ✓ Request logging               ✓ Load balancing                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                 │
                                 │  HTTPS
                                 │  Authorization: Bearer $DATABRICKS_TOKEN
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATABRICKS WORKSPACE                              │
│                   https://your-workspace.cloud.databricks.com               │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      MODEL SERVING ENDPOINTS                        │   │
│   │                        /serving-endpoints                           │   │
│   │                                                                     │   │
│   │   ┌─────────────────┐                                               │   │
│   │   │ databricks-gpt- │  ◄── Your deployed model                      │   │
│   │   │      5-1        │      (Foundation Model / Custom)              │   │
│   │   └─────────────────┘                                               │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Request Flow

1. **Application** sends OpenAI-compatible request to LiteLLM proxy
2. **LiteLLM Proxy** translates and routes request to Databricks
3. **Databricks** processes request through Model Serving endpoint
4. **Response** flows back through LiteLLM in OpenAI format

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABRICKS_API_BASE` | Full URL to serving endpoints | `https://adb-123.45.azuredatabricks.net/serving-endpoints` |
| `DATABRICKS_TOKEN` | Personal Access Token | `dapi...` |

## Quick Start

```bash
# 1. Set environment variables
export DATABRICKS_API_BASE="https://your-workspace.cloud.databricks.com/serving-endpoints"
export DATABRICKS_TOKEN="your-databricks-token"

# 2. Start the proxy
litellm --config litellm_config.yaml

# 3. Test it
curl -X POST 'http://0.0.0.0:4000/chat/completions' \
  -H 'Content-Type: application/json' \
  -H 'x-litellm-api-key: sk-1234' \
  -d '{
    "model": "databricks-gpt-5-1",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Benefits of Using LiteLLM as a Gateway

- **Unified API**: Use OpenAI SDK/format with any backend
- **Swap models easily**: Change providers without code changes
- **Observability**: Built-in logging and cost tracking
- **Access control**: API key management and rate limiting

