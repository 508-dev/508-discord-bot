# GPT-5.6 Luna vs GPT-5.4 Mini planner evaluation

- Evaluation date: 2026-09-09
- Harness base revision: `34f544a3ed588a05c0f51598ff040997c0a8aeeb`
- Harness: Discord-agent canonical live-planner eval
- Route: direct OpenAI (`https://api.openai.com/v1`) using `OPENAI_API_KEY`
- Models: `gpt-5.6-luna` and `gpt-5.4-mini`
- Method: two serial 27-scenario runs, JSON-object responses, a 90-second
  per-scenario timeout, and normal harness retry behavior. The Luna run used
  `max_completion_tokens`, `reasoning_effort=low`, and `verbosity=low`;
  `temperature` was omitted.

## Result

Luna and Mini produced the same retained provider-draft pass rate and both had
complete parse and production success. Luna was slower, but it used fewer
output tokens and—using the contemporaneous OpenRouter rate card only as a
proxy—has an estimated selected-use cost 88.7% lower.

| Metric | GPT-5.6 Luna | GPT-5.4 Mini |
| --- | ---: | ---: |
| Production scenarios passed | 27 / 27 | 27 / 27 |
| Provider-draft parses | 27 / 27 | 27 / 27 |
| Retained provider-draft passes | 19 / 27 | 19 / 27 |
| Provider-draft failures | 8 | 8 |
| Average selected latency | 2,145.5 ms | 1,384.7 ms |
| Maximum selected latency | 4,415 ms | 2,031 ms |
| Total wall time | 73,779 ms | 51,889 ms |
| Selected input tokens | 32,630 | 32,630 |
| Selected cached-input tokens | 30,671 | 0 |
| Selected output tokens | 2,824 | 3,228 |
| Harness selected-use cost | not available | $0.0389985 |
| Harness retries | 9 | 10 |

Luna's selected average latency was 54.9% higher and its end-to-end suite time
was 42.2% longer. It generated 12.5% fewer selected output tokens.

## Cost interpretation

The direct OpenAI billing API was not available to this run, and no official
direct Luna price was added to the local catalog. For an apples-to-apples
comparison, the public [OpenRouter Luna](https://openrouter.ai/openai/gpt-5.6-luna)
and [OpenRouter Mini](https://openrouter.ai/openai/gpt-5.4-mini) rate cards
captured on the run date are used only as a rate proxy:

| Model | Input / M | Cache read / M | Output / M | Selected-use proxy |
| --- | ---: | ---: | ---: | ---: |
| GPT-5.6 Luna | $0.20 | $0.02 | $1.20 | $0.00439402 |
| GPT-5.4 Mini | $0.75 | $0.075 | $4.50 | $0.03899850 |

That proxy puts Luna at 8.88x lower selected-use cost (an 88.7% reduction).
It is not an invoice or a promise of direct-OpenAI billing. The harness records
usage for the selected scenario result; discarded retry attempts are billable
but excluded from these totals, so real run cost will be higher.

## What the behavioral result does and does not show

The provider-draft signals tie at 19/27. Both models had all eight parse-safe
draft failures retained by the harness, but the failures were not identical:
Luna missed two GitHub-query field assertions and the missing-email
clarification case, while Mini missed those two GitHub fields plus task
confirmation and user-account argument assertions. The remaining failures in
each run were intent-label differences.

All 27 production outcomes passed because the canonical suite is primarily a
policy and deterministic-routing regression suite (26 of its 27 fixtures have
a deterministic ownership path). This demonstrates that the Luna integration
preserves the production safety contract; it is not a broad quality benchmark
for arbitrary planning or resume extraction.

## Historical OpenRouter attempt

The initial OpenRouter preflight for both models returned HTTP 403 before any
provider draft or usage. The account-level message cited a provider Terms of
Service restriction and named no provider. That route remains blocked, but it
does not affect this direct-OpenAI evaluation.

## Decision and activation

Luna is the selected default for the direct OpenAI workflow configuration in
this workspace: it is protocol-compatible, preserves the evaluated agent
contract, and offers a large rate-card cost advantage in exchange for slower
responses. The ignored local `.env` now pins these model selectors to
`gpt-5.6-luna`:

- `OPENAI_MODEL` for job-requirement extraction and candidate reranking.
- `AGENT_FALLBACK_MODEL` for the Discord-agent and lead-classification fallback.
- `RESUME_AI_MODEL` for resume extraction.
- `AGENT_EVAL_OPENAI_MODEL` for future primary eval runs.

This local activation does not modify any deployment dashboard or other
environment. Mirror the same variables in a deployment only after accepting
the resume-workflow caveat above and monitoring its output quality.
