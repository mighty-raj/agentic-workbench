# Cost Estimators

A bundled cost-estimation toolkit for AI workloads. Price prompts against OpenAI
models and size Microsoft Fabric Data Agent workloads — all from VS Code Chat.

This plugin combines two estimators behind a single install:

- **OpenAI prompt cost** — tokenize prompts with `tiktoken` and cost them against
  live pricing from [developers.openai.com](https://developers.openai.com/api/docs/pricing).
- **Fabric Data Agent CU cost** — compute Capacity Unit (CU) consumption per
  question, project monthly cost, and recommend the right Fabric SKU
  (F2 → F256) using the official
  [Fabric Data Agent consumption rates](https://learn.microsoft.com/en-us/fabric/fundamentals/data-agent-consumption).

## What's Included

### Agents

| Name             | Invoke with        | Purpose                                                                            |
| ---------------- | ------------------ | ---------------------------------------------------------------------------------- |
| llm-prompt-cost  | `@llm-prompt-cost` | Tokenize a prompt, fetch live OpenAI pricing, produce a cost-comparison table.     |
| fda-cost-agent   | `@fda-cost-agent`  | Estimate Fabric Data Agent CU consumption, monthly cost, and recommended SKU.      |

### Skills

| Name                          | Used by           | Purpose                                                                                                  |
| ----------------------------- | ----------------- | -------------------------------------------------------------------------------------------------------- |
| openai-models-cost-estimator  | `llm-prompt-cost` | Tokenization, pricing lookup, and per-1M-token cost math for OpenAI models.                              |
| fda-cost-estimation-skill     | `fda-cost-agent`  | Per-token CU rates, retry model, monthly aggregation, PAYG vs 1-year vs 3-year reservation comparison.   |

## Quick Start

### Price an OpenAI prompt

```text
@llm-prompt-cost Estimate cost for - Input prompt: ``prompts/request.json``, Output Response: ``prompts/response.json``
```

Or against a specific model:

```text
@llm-prompt-cost How much would it cost to run this against gpt-5.4-mini?
<paste prompt text here>
```

### Estimate Fabric Data Agent cost

By workload size:

```text
@fda-cost-agent Estimate Fabric Data Agent cost for - Requests/day: 5000
```

By user population:

```text
@fda-cost-agent Estimate Fabric Data Agent cost for - Users: 200, Queries/user/day: 25
```

With your own sample prompt:

```text
@fda-cost-agent Estimate Fabric Data Agent cost for - Requests/day: 5000, Input prompt: ``samples/q.txt``, Output Response: ``samples/a.txt``
```

## How It Works

### `llm-prompt-cost`

1. **Resolve input** — reads file paths or accepts inline text.
2. **Tokenize** — `tiktoken` with the model-appropriate encoding (`o200k_base`, `cl100k_base`).
3. **Fetch pricing** — live per-1M-token rates from the OpenAI pricing and model-detail pages.
4. **Produce table** — Markdown comparison sorted by total cost, with at-scale columns (`@50x`, `@100x`).

The agent is **read-only** — it prices prompts but never edits code.

### `fda-cost-agent`

1. **Fetch CU rates** — per-token CU-seconds for input/output from the Fabric consumption-rates page.
2. **Tokenize sample** (optional) — `tiktoken` with `o200k_base`; otherwise uses Microsoft's worked example.
3. **Per-turn CU** — convert tokens to CU-seconds for one LLM turn.
4. **Apply retry model** — Stage 1 (plan) + Stage 2 (reflect/retry, default 1 retry) + Stage 3 (format) ⇒ ~3 effective turns/question.
5. **Aggregate** — multiply by requests/day and project monthly CU + cost.
6. **Recommend SKU** — pick the smallest Fabric capacity (F2…F256) that fits, and compare PAYG vs 1-year vs 3-year reservation discounts.

## Dependencies

- `tiktoken` — installed automatically via `pip` for tokenization.
- Web access — both agents fetch live pricing/consumption data at runtime:
  - `developers.openai.com` (OpenAI pricing)
  - `learn.microsoft.com` (Fabric Data Agent consumption rates and SKU pricing)

## Installation

See the [root README](../../README.md#install-plugins-in-vs-code) for full
installation options. The quickest path is to register the folder locally:

```jsonc
// settings.json
"chat.pluginLocations": {
  "C:/path/to/agentic-workbench/plugins/cost-estimators": true
}
```

Reload VS Code — the `@llm-prompt-cost` and `@fda-cost-agent` agents, along with
their backing skills, will appear in Chat.
