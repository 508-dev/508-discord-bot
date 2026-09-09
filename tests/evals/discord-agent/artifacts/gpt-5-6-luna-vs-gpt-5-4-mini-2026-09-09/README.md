# Direct OpenAI Luna/Mini audit snapshot

This tracked, immutable snapshot supports the findings in [the direct OpenAI
planner evaluation report](../../../../../.context/reports/2026-09-09-gpt-5-6-luna-vs-gpt-5-4-mini-eval.md). It is deliberately separate from
`tests/evals/discord-agent/reports/`, which remains ignored scratch output.

## Scope

- Evaluated runtime revision: `816db61555a09c4e0ccafdd7827b6ca43a797bf3`.
- Route: direct OpenAI (`https://api.openai.com/v1`) using the primary eval
  profile and `OPENAI_API_KEY`.
- Full sweeps: one serial 27-scenario canonical run for `gpt-5.6-luna` and one
  for `gpt-5.4-mini`, each with a 90-second timeout for each HTTP request.
- The full-sweep files preserve selected scenario checks, provider-draft
  results, selected latency, token usage, and raw synthetic model drafts.

The first request in an attempt uses JSON-object `response_format`. Any HTTP
error triggers an immediate second request without that format; a scenario
whose production result is `failed` or whose provider draft fails gets one
full retry. The retry replaces the first result only if both production and
provider-draft checks pass. Thus an affected scenario can make up to four
provider requests, while the published observation represents only the
selected scenario result. The `retries` metric counts full scenario retries,
not inner HTTP fallbacks.

The `api_key_configured` boolean was removed from every published observation.
No credentials, request headers, HTML viewer, CTRF output, or debug trace is
retained here. These are historical observations, not a claim that future
provider responses will be identical.

## Integrity check

From the repository root, verify the snapshot with:

```bash
sha256sum -c tests/evals/discord-agent/artifacts/gpt-5-6-luna-vs-gpt-5-4-mini-2026-09-09/SHA256SUMS
```
