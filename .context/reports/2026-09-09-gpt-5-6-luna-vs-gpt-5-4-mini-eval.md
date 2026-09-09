# GPT-5.6 Luna vs GPT-5.4 Mini planner-eval attempt

- Attempt date: 2026-09-09
- Runtime commit: `34f544a3ed588a05c0f51598ff040997c0a8aeeb`
- Harness: Discord-agent canonical live-planner eval, OpenRouter profile
- Models: `openai/gpt-5.6-luna` and `openai/gpt-5.4-mini`

## Rate-card comparison

OpenRouter's public model catalog was queried immediately before the run. Its
prices are per million tokens and can change with provider routing or catalog
updates.

| Model | Input | Cache read | Output | Relative to GPT-5.4 Mini |
| --- | ---: | ---: | ---: | ---: |
| [GPT-5.6 Luna](https://openrouter.ai/openai/gpt-5.6-luna) | $0.20 | $0.02 | $1.20 | 3.75x lower on each common rate |
| [GPT-5.4 Mini](https://openrouter.ai/openai/gpt-5.4-mini) | $0.75 | $0.075 | $4.50 | baseline |

On the published rate card, Luna is 73.3% cheaper for prompt, cache-read, and
completion tokens. Luna also lists a $0.25/M cache-write rate; the Mini catalog
entry did not publish a cache-write rate, so that component is not compared.

## Live-eval result

No full evaluation was run. A serial, fixture-stubbed preflight was attempted
for each model using the same canonical scenario,
`crm_contact_info_lookup_001`, a 90-second per-scenario timeout, JSON-object
response format, and the harness's normal retry behavior.

Both models returned HTTP 403 before generating a provider draft or reporting
usage:

| Model | Provider draft | Parse success | Usage | Result |
| --- | --- | ---: | --- | --- |
| GPT-5.6 Luna | `parse_failed` | no | none | HTTP 403 |
| GPT-5.4 Mini | `parse_failed` | no | none | HTTP 403 |

The authorized OpenRouter key is not expired or exhausted, but a minimal
diagnostic request returned: `The request is prohibited due to a violation of
provider Terms Of Service.` The provider name was not supplied. This is an
account/provider access block, not evidence about either model's planner
quality, latency, or realized cost.

## Conclusion

Luna is materially cheaper on the currently published rate card, but there is
no valid behavioral comparison yet. Do not select it from this attempt. Restore
or replace the OpenRouter access path, then rerun both 27-scenario canonical
live-planner suites serially on the same commit and compare retained provider
draft pass rate, parse success, latency, reported usage, and routed-provider
cost.
