# Hermes Pantheon — Knowledge Vault

> **Purpose:** This directory is the operator's drop-point for any new trading knowledge — PDFs, YouTube transcripts, Obsidian Markdown notes, course exports, screenshots of charts. **TRITON AI** watches this directory and ingests new files into the relevant agent's training corpus.

## How to use

1. Treat this directory as your Obsidian vault root (or symlink your real Obsidian vault to it).
2. Drop any new knowledge artifact into this folder. Subfolders are encouraged for organization but TRITON ingests the entire tree.
3. TRITON fires automatically on the next Mega Cycle when the rotation lands on `obsidian_ingest`, OR on demand via `/triton ingest <path>`.

## Recommended folder layout

```
knowledge_vault/
├── methodology/
│   ├── order_flow/                # Bank Protocol, Masters of the Bank notes
│   ├── market_cipher/             # Crypto Face videos, Market Cipher manuals
│   ├── delta_volume_profile/      # Trade With Profile, DeltaTrend notes
│   ├── market_profile/            # Steidlmayer, Dalton, auction theory
│   ├── wyckoff/                   # Classical Wyckoff, SpacemanBTC
│   ├── markov_hmm/                # Statistical regime modeling notes
│   ├── algo_pro/                  # Algo Pro indicator manuals
│   ├── lux_algo/                  # LuxAlgo Premium guides
│   ├── quantpad/                  # QuantPad indicator references
│   └── jim_simons_rentech/        # Statistical-arb writeups
├── playbooks/                     # Personal setups, journal lessons
├── chart_library/                 # Annotated chart screenshots
├── transcripts/                   # YouTube / podcast transcripts
└── operator_notes/                # Free-form operator thoughts
```

## TRITON's ingestion behavior

For each new or modified file, TRITON:

1. Detects file type (PDF, MD, TXT, JSON, screenshot).
2. Parses with the appropriate tool (LlamaParse for PDF; vision for images).
3. Chunks by concept (not by token count alone).
4. **Tags** each chunk with: `methodology`, `instrument`, `timeframe`, `regime`, `source`, `recency`.
5. Routes chunks to the relevant agent's prompt corpus:
   - `methodology/order_flow/*` → SCYLLA AI
   - `methodology/market_cipher/*` → SIREN AI
   - `methodology/delta_volume_profile/*` → CHARYBDIS AI
   - `methodology/market_profile/*` → CHARYBDIS AI
   - `methodology/wyckoff/*` → MERMAID AI
   - `methodology/markov_hmm/*` → NAUTILUS AI
   - `methodology/jim_simons_rentech/*` → NAUTILUS AI
   - `methodology/algo_pro/*`, `lux_algo/*`, `quantpad/*` → SIREN AI
   - `playbooks/*`, `operator_notes/*` → ORCA AI (memory) + Council weighting
   - `chart_library/*` → SIREN AI (vision)
6. Records the ingestion in `knowledge_vault/.triton_index.jsonl` (atomic appended).

## Self-Learning Boundary

- TRITON can **propose** new heuristics, weights, or even draft Strategy Genomes from ingested knowledge.
- Strategy Genomes go to LOCHNESS → NAUTILUS RBI gauntlet → TIAMAT veto — they cannot reach live capital without surviving the entire pipeline.
- TRITON CANNOT modify `risk_governance.lock.yaml`. Any proposal touching risk is logged to `risk/proposals.jsonl` for operator review.

## Anti-Pattern Discipline

- `feedback_atomic_writes_shared_state` — `.triton_index.jsonl` uses atomic write.
- `feedback_silent_skip_pattern` — every file TRITON cannot parse MUST be logged with a named reason in the index.
- `feedback_speculative_write` — TRITON never marks a chunk as "ingested" without the index entry being persisted to disk first.
- `feedback_diagnostic_first` — if the same file fails to parse twice, TRITON spawns a parsing-diagnostic skill before the third attempt.
