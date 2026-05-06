---
name: openai-models-cost-estimator
description: 'Estimate the cost of running a prompt against one or more OpenAI models. 
USE FOR: token counting, pricing prompts, comparing model costs, estimating token usage.'
---

# OpenAI Models Cost Estimator

You estimate the **input-output token cost** of running a user-supplied prompt against one or more OpenAI models. 

## Instructions on how to calculate token counts and costs:

1. **Tokenization**: Use the `tiktoken` library to tokenize the prompt text according to the model's encoding. 

    ```python
    # Requires: pip install --upgrade tiktoken -q
    import tiktoken
    encoding = tiktoken.encoding_for_model(model)
    ```

2. **Get Model Encoding**: Determine the correct encoding for each model. Use the appropriate encoding when counting tokens.

    ```python
    # Requires: pip install --upgrade tiktoken -q
    import tiktoken
    encoding = tiktoken.encoding_for_model(model)
    ```

3. **Token Counting**: Count the number of tokens in the prompt text for each model using the correct encoding.
    ```python
    # Requires: pip install --upgrade tiktoken -q
    import tiktoken
    def count_tokens(prompt_text, model):
        encoding = tiktoken.encoding_for_model(model)
        tokens = encoding.encode(prompt_text)
        return len(tokens)
    ```

4. **Cost calculation**: Multiply the token count by the model's input and output token prices (in USD) to get the total cost.
    Use browser tools to navigate, expand, and explore the pricing information for each model to retrieve the input and output token prices. 
    See [OpenAI Pricing](https://developers.openai.com/api/docs/pricing) for current token prices.

### Resolving pricing for legacy / non-flagship models

The main pricing page only lists current flagship + multimodal + specialized models. Legacy models (e.g. `gpt-5`, `gpt-5-mini`, `gpt-4.1`, `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `gpt-3.5-turbo`, `text-embedding-ada-002`) are NOT in the main table. Use this lookup order:

1. **Main pricing page** — https://developers.openai.com/api/docs/pricing
   Covers flagship (gpt-5.x), realtime, image, video, transcription, specialized, fine-tuning.
2. **Model detail page** — `https://developers.openai.com/api/docs/models/<model-id>`
   Each model has a "Pricing" section with `Input`, `Cached input`, `Output` per 1M tokens. This is the source of truth for legacy models.
3. **Catalog lookup** — https://developers.openai.com/api/docs/models/all
   Use this to find the canonical `<model-id>` (and deprecation status) when you're unsure of the exact ID, then fetch its detail page from step 2.
4. **Fully removed models** (no detail page) — check https://developers.openai.com/api/docs/deprecations or the historical `openai/openai-cookbook` repo. Flag the rate as "historical / no longer billable" in the output.
5. **Embeddings models** — detail pages list only an input price (no output cost). Treat as input-only.


## Dependencies
- `pip install --upgrade tiktoken -q` -- to perform tokenization according to OpenAI's models.
- `pip install --upgrade openai -q` -- to access OpenAI's API for pricing information.
- Access to OpenAI's pricing information for accurate cost estimation. This may require web scraping or API access to retrieve current token prices for each model.


