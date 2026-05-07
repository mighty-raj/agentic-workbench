# LLM Cost Estimator

Estimate input/output token cost of prompts across OpenAI models using `tiktoken`
tokenization and live pricing from [developers.openai.com](https://developers.openai.com/api/docs/pricing).
Compare per-call and at-scale costs before committing to a model.

## What's Included

### Agent

| Name            | Invoke with       | Purpose                                                    |
| --------------- | ----------------- | ---------------------------------------------------------- |
| llm-prompt-cost | `@llm-prompt-cost` | Tokenize a prompt, fetch live pricing, produce a cost table |

### Skill

`openai-models-cost-estimator` provides the domain knowledge for:

- **Tokenization** — count tokens via `tiktoken` with the correct encoding per model (`o200k_base`, `cl100k_base`).
- **Pricing resolution** — lookup order across the main pricing page, model detail pages, catalog, and deprecation docs.
- **Cost calculation** — multiply token counts by per-1M-token rates for input and output.

## Quick Start

### 1. Price an inline prompt

```
@llm-prompt-cost Estimate cost for: "Summarize this quarterly earnings report…"
```

### 2. Price a file (input only)

```
@llm-prompt-cost Estimate cost for - Input prompt: ``path/to/my-prompt.json``
```

### 3. Price input + expected output

```
@llm-prompt-cost Estimate cost Against gpt-4o for - Input prompt: ``prompts/request.json``, Output Response: ``prompts/response.json``
```

### 4. Target a specific model

```
@llm-prompt-cost How much would it cost to run this against gpt-5.4-mini?
<paste prompt text here>
```

## Default Model Set

When no model override is given, costs are compared across these models:

| #  | Model                    | Encoding     |
| -- | ------------------------ | ------------ |
| 1  | gpt-5.5                  | o200k_base   |
| 2  | gpt-5.5-pro              | o200k_base   |
| 3  | gpt-5.4                  | o200k_base   |
| 4  | gpt-5.4-pro              | o200k_base   |
| 5  | gpt-5.4-mini             | o200k_base   |
| 6  | gpt-5.4-nano             | o200k_base   |
| 7  | gpt-5                    | o200k_base   |
| 8  | gpt-5-mini               | o200k_base   |
| 9  | gpt-4.1                  | o200k_base   |
| 10 | gpt-4o                   | o200k_base   |
| 11 | gpt-4o-mini              | o200k_base   |
| 12 | gpt-4-turbo              | cl100k_base  |
| 13 | gpt-3.5-turbo            | cl100k_base  |
| 14 | text-embedding-ada-002   | cl100k_base  |

## How It Works

1. **Resolve input** — the agent reads file paths or accepts inline text.
2. **Tokenize** — uses `tiktoken` with the model-appropriate encoding to count input and output tokens.
3. **Fetch pricing** — retrieves current per-1M-token rates from the OpenAI pricing page and model detail pages.
4. **Produce table** — outputs a Markdown comparison table sorted by total cost (descending), with per-call and at-scale columns (`@50x`, `@100x`).

The agent is **read-only** — it prices prompts but never edits code or proposes changes.

## Dependencies

- `tiktoken` — installed automatically via `pip` for tokenization.
- Web access — the agent fetches live pricing from `developers.openai.com` at runtime.

## Installation

See the [root README](../../README.md#install-plugins-in-vs-code) for full installation options. The quickest path:

```jsonc
// settings.json
"chat.pluginLocations": {
  "C:/path/to/agentic-workbench/plugins/llm-cost-estimator": true
}
```

Reload VS Code — the `@llm-prompt-cost` agent and `openai-models-cost-estimator` skill will appear in Chat.
