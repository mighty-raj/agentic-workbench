---
name: fda-cost-estimation-skill
description: 'Estimate the Capacity Unit (CU) consumption and monthly Azure cost of running user queries
through Microsoft Fabric Data Agent, and recommend the right Fabric SKU.
USE FOR: Fabric Data Agent cost estimation, CU consumption calculation, Fabric SKU sizing,
F2/F4/F8/F16/F32/F64/F128/F256 capacity comparison, AI Query billing, LlmPlugin cost,
reservation discount comparison (PAYG vs 1-year vs 3-year).'
---

# Fabric Data Agent Cost Estimator

You estimate the **Capacity Unit (CU) consumption and Cost** of running user queries through Microsoft Fabric Data Agent, and recommend the right Fabric SKU based on a target requests/day workload.

Use web-fetch/browser tools to navigate the consumption rate/pricing page. 

## Calculation pipeline

Follow these seven steps in order. Each step produces an input for the next.

### Step 1 — Fetch per-token CU consumption rates

Get the per-token CU-seconds for input and output from the [Microsoft Fabric Data Agent consumption rates](https://learn.microsoft.com/en-us/fabric/fundamentals/data-agent-consumption) page.

### Step 2 — Tokenize the sample prompt (optional)

Only when the user provides input/output prompt files. Count tokens with `tiktoken` using the `o200k_base` encoding:

```python
# Requires: pip install --upgrade tiktoken -q
import tiktoken
def count_tokens(prompt_text, model):
    encoding = tiktoken.encoding_for_model(model)
    tokens = encoding.encode(prompt_text)
    return len(tokens)
```

If no sample is provided, use Microsoft's worked example from the [consumption rates page](https://learn.microsoft.com/en-us/fabric/fundamentals/data-agent-consumption).

### Step 3 — Compute per-turn CU consumption

Convert tokens to CU-seconds for a single LLM turn:

### Step 4 — Apply the retry model (effective turns per question)

A single user question triggers multiple LLM turns inside the Data Agent:

| Turn | Stage | What happens |
|---|---|---|
| 1 | **Stage 1 — Plan** | LLM generates SQL/DAX from the user's natural-language question |
| 2..N | **Stage 2 — Reflect/retry** | If the SQL fails (wrong column, type mismatch, etc.), the LLM regenerates with error context. Each retry **carries prior conversation history**, so input tokens grow. |
| N+1 | **Stage 3 — Format** | LLM turns the result rows into a natural-language answer |

**Default turn model:**

- Base turns = **2** (Stage 1 + Stage 3 — no retry)
- Stage-2 retries assumed = **1** (typical real-world default)
- Effective turns per question = **3**

```python
def effective_cu_per_question(per_turn_cu, stage2_retries=1):
    base_turns = 2  # Stage 1 plan + Stage 3 format
    effective_turns = base_turns + stage2_retries
    return per_turn_cu * effective_turns
```

⚠ **The Stage-2 retry count is highly variable and can significantly impact the estimation — it is the single largest source of error in the CU calculation.** It depends on schema quality:

| Schema quality | Typical retries | Effective turns/Q |
|---|---:|---:|
| 🟢 Clean (well-named columns, descriptions, narrow tables) | 0 | 2 |
| 🟡 Typical (default) | 1 | 3 |
| 🔴 Wide tables, cryptic column names, vague questions | 2–3 | 4–5 |

Always disclose the assumption with a ⚠ flag and offer it as a sensitivity scenario in the recommendations footer.

### Step 5 — Max questions per day (the headline capacity metric)

Convert effective CU-seconds per question to CU-hours, then divide by the SKU's daily CU-hour budget:

```python
def max_questions_per_day(sku_cus, effective_cu_seconds_per_q):
    cu_hours_per_day = sku_cus * 24
    cu_hours_per_q = effective_cu_seconds_per_q / 3600
    return int(cu_hours_per_day / cu_hours_per_q)
```

### Step 6 — Fetch SKU pricing and compute monthly cost

Live-fetch Fabric capacity hourly pricing from the [Azure Fabric pricing page](https://azure.microsoft.com/en-us/pricing/details/microsoft-fabric/) for PAYG, 1-year reserved, and 3-year reserved. Convert hourly to monthly using the standard 730-hour convention:

```python
def monthly_cost_usd(hourly_rate_usd):
    return hourly_rate_usd * 730  # standard month-hours convention
```

If a price isn't found for a SKU/region/reservation combination, populate that cell with `n/a` and flag it in the footer — **do not fabricate a price**.

### Step 7 — Recommend the smallest SKU that fits

A SKU is "recommended" when it is the **smallest SKU** whose `Max Questions/day` ≥ user's `requests/day` × **1.15** (a 15% headroom safety factor for traffic variability).

```python
def recommend_sku(requests_per_day, sku_capacities):
    target = requests_per_day * 1.15
    for sku, max_q in sorted(sku_capacities.items(), key=lambda x: x[1]):
        if max_q >= target:
            return sku
    return None  # workload exceeds all SKUs in default set
```


## Source URLs

| Source | URL | What to extract |
|---|---|---|
| **Consumption rates** (authoritative) | https://learn.microsoft.com/en-us/fabric/fundamentals/data-agent-consumption | Per-token CU rates (100/400), background-job smoothing behavior, AI Query / LlmPlugin operation names, Stage-2 separate billing rule |
| **Capacity SKUs** | https://learn.microsoft.com/en-us/fabric/enterprise/licenses | SKU → CU mapping (F2 = 2 CU, ..., F2048 = 2048 CU) |
| **Pricing** (live) | https://azure.microsoft.com/en-us/pricing/details/microsoft-fabric/ | $/hour per SKU, per region, for PAYG / 1-yr / 3-yr reservations |


Always cite the **consumption-rates source URL** in every output footer.

## Hard rules

### Data integrity rules

- **No fabrication.** If pricing for a SKU/region/reservation isn't on the Azure pricing page, populate `n/a` and say so. If tokenization fails, say so.
- **One fetch per source per call.** Don't loop or retry beyond a single sensible attempt per pricing-page section.

### Disclosure rules

- **Always disclose the Stage-2 retry assumption** with a ⚠ flag — it is highly variable and the single largest source of estimation error.
- **Always state what's excluded** (Stage-2 query execution, storage, system-prompt overhead) — scope honesty builds trust.

### Modeling rules

- **Background-job smoothing applies** — Data Agent is classified as a background job in Fabric. Don't model spikes; total daily CU supply vs total daily CU demand is the right comparison.

## Dependencies

- `pip install --upgrade tiktoken -q` — for tokenization (reuses the same library as `openai-models-cost-estimator`).
- Web access — fetch live Fabric capacity pricing from `azure.microsoft.com/pricing/details/microsoft-fabric/` at runtime.
