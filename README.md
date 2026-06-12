# failure-lantern

Find the crash before the pager goes off.

Most log aggregators are great at storage and terrible at triage. When a service dies at 3:14 AM, you don't need a high-level summary of "system health" or a pretty dashboard. You need to know which specific event started the cascade. 

Failure Lantern is a triage utility designed to surface the first observable fault in a sea of secondary errors. It combines deterministic pattern matching with local LLM assistance to identify root causes in noisy application logs.

### The Problem with Noise

When a database connection fails, every dependent service starts screaming. You end up with 10,000 log lines in sixty seconds. Most of those lines are useless "Service Unavailable" messages. Somewhere in that pile is the actual culprit: a malformed config change or a silent credential expiration. 

Founders and lead engineers don't want AI-generated summaries that hallucinate system architecture. They want to know why production is dead.

### How it works

Failure Lantern uses a two-stage approach to clear the hay:

*   **Deterministic Clustering:** It uses regex-based pattern matching to group known stack traces and repetitive warnings. If the same error appears 500 times, it occupies one line in the triage view.
*   **Timestamp Correlation:** The tool maps errors across different log streams (e.g., Nginx, Auth service, and Postgres) to find the chronological "Patient Zero."
*   **Local LLM Heuristics:** For the "weird" errors that don't match known patterns, it uses a local model (like Llama 3 or Mistral) to analyze the specific error context and suggest why it might be failing.
*   **Failure Chain Mapping:** It attempts to link events. For example: `Config Change -> 401 Unauthorized -> Service Exit`.

### MVP Features

1.  **Log Ingestion:** Stream logs directly from `stdout` or upload a flat `.log` file.
2.  **Cluster Engine:** Automatically collapses identical errors and counts occurrences.
3.  **Trace Grouping:** Gathers related stack traces to show the full execution path of a crash.
4.  **Candidate Selection:** Produces a list of the top three most likely root causes based on timing and severity.

### Usage

The tool is designed to run locally. No data leaves your machine. 

```bash
cat production_crash.log | python3 lantern.py --cluster --analyze
```

### Why I built this

I spend a lot of time investigating production failures. The most frustrating part of incident response is the "information gathering" phase where three different people are grepping logs and coming to three different conclusions. Failure Lantern provides a single, filtered view of the disaster. 

It doesn't replace a senior engineer. It just gives that engineer the right data so they can stop digging and start fixing.
