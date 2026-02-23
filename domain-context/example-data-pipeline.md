---
name: "Data Pipeline & ML"
triggers:
  paths:
    - "src/pipeline/"
    - "src/pipelines/"
    - "src/models/"
    - "src/features/"
    - "src/transforms/"
    - "notebooks/"
    - "dags/"
  keywords:
    - "pipeline"
    - "transform"
    - "feature"
    - "training"
    - "inference"
    - "model"
    - "dag"
    - "batch"
    - "etl"
---

## Overview

The data pipeline ingests raw data from external sources, transforms it through a series of typed stages, and produces features for ML model training and inference. Pipelines are defined as DAGs (directed acyclic graphs) with explicit dependencies between stages. Each stage is idempotent and can be retried independently.

## Key Patterns

- **DAG structure:** Pipelines are defined declaratively as a graph of stages. Each stage declares its inputs (upstream stages or raw sources) and outputs (typed datasets). The orchestrator resolves execution order from the DAG.
- **Data contracts:** Every stage boundary has a typed schema (Pydantic models or equivalent). Input validation runs at the start of each stage — if the upstream output doesn't match the expected schema, the stage fails fast with a clear error.
- **Idempotency:** All stages are idempotent. Output is written to a partitioned location keyed by run ID. Re-running a stage with the same inputs produces identical output. No append-only writes — always overwrite the partition.
- **Model versioning:** Trained models are stored with metadata (training data hash, hyperparameters, metrics). Model serving loads by version tag, not by "latest". Promoting a model to production is an explicit step, not automatic.
- **Feature store:** Computed features are stored in a feature store with point-in-time correctness. Training uses historical features (as of training timestamp), inference uses current features. This prevents data leakage.

## File Locations

| Path | Purpose |
|------|---------|
| `src/pipeline/` or `dags/` | Pipeline/DAG definitions |
| `src/transforms/` | Individual transformation stages |
| `src/features/` | Feature engineering and feature store access |
| `src/models/` | Model training, evaluation, and serving code |
| `notebooks/` | Exploration and analysis (not production code) |
| `config/` | Pipeline configs, hyperparameters, source credentials |
| `tests/` | Stage-level tests with fixture data |

## Notes

- Notebooks are for exploration only — never imported by production pipeline code.
- All external API calls (data sources) go through a retry wrapper with exponential backoff.
- Large datasets are processed in chunks to control memory usage. Chunk size is configurable per stage.
- Credentials for data sources live in environment variables or a secrets manager, never in config files.
- CI runs the full pipeline on a small fixture dataset to catch schema drift before deployment.
