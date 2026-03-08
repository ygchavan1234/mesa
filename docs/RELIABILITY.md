# Reliability Report: Stress Testing the Infrastructure

Generative Agents inherently hit API rate limits ("429 Resource Exhausted"). This document outlines our "Senior Move" handling logic to ensure research continuity.

## 1. litellm Retry Backoff
Inside `run.py->setup_environment()`, we configure `litellm` specifically for high-throughput multi-agent execution:
- `litellm.num_retries = 2` (Prevents runaway cost loops)
- `litellm.suppress_debug_info = True` (Silences the noisy "Provider Feedback" text dump)

## 2. The Stress Test Victory Panel
When the generative model infrastructure hits a hard 429 quota block, the simulation does not return a raw Python stack trace. Instead, it exits gracefully with a "Stress Test Victory" panel via the `rich` library.

This communicates clearly to the researcher that the **code logic outpaced the API tier infrastructure**, not the other way around.

## 3. The `finally` Dump Protocol
All reasoning steps exist primarily in memory (`model.history_log`).

We utilize a strict `try...except...finally` execution context. If the script is severed at Step 8 by an API quota error, the `finally` block ensures Steps 1-8 are written securely to `logs/full_simulation_audit.json`. No data is lost.

### Mesa 4.0 Integration
The finally block specifically calls `model.dataregistry.flush()`, leveraging the 2026 Mesa 'Push' architecture. This ensures that even in an interrupted state, all agent observations and price signals are serialized into Parquet/JSON formats for post-mortem analysis.
