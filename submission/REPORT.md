# Day 18 Lakehouse Lab Report

Student repo: `Day18-Track2-2A202601211-NguyenVietLinh`

## Executive Summary

The lightweight path is complete. All 8 notebooks in `notebooks/` were executed with output preserved, screenshots were exported to `submission/screenshots/`, and the mechanical grading gates passed:

```text
make run-all: 8/8 notebooks passed
make test:    24 passed
```

The Spark path is not yet executed. The 4 notebooks in `notebooks-spark/` require a Spark runtime. This machine currently has no local `java`, no `.venv` `pyspark`/`delta`, and the Docker Spark image was not downloaded because the download was explicitly stopped. To finish Spark evidence, run `make spark-up`, `make spark-smoke`, `make spark-data`, then execute the 4 Spark notebooks after allowing Docker to pull `quay.io/jupyter/pyspark-notebook:spark-3.5.0`.

## Deliverables

- Executed notebooks: `notebooks/01_delta_basics.ipynb` through `notebooks/08_agents_provenance.ipynb`
- Screenshots: `submission/screenshots/NB01.png` through `submission/screenshots/NB08.png`
- Reflection: `submission/REFLECTION.md`
- Note for submission: `.gitignore` ignores `notebooks/*.ipynb` and `notebooks-spark/*.ipynb`; use `git add -f` for executed notebooks if submitting through git.

## Rubric Evidence

| NB | Criterion | Evidence | Interpretation |
|---|---|---|---|
| 1 | Delta transaction log, schema enforcement, schema evolution | `_delta_log/ has JSON commits`; bad `age=str` write blocked; `tier` added by `schema_mode=merge`; DuckDB sees 2 tier groups | Delta gives ACID table metadata and schema control. Schema evolution is opt-in, so accidental shape drift is blocked but planned changes are possible. |
| 2 | Small files, OPTIMIZE/Z-order | Files before: 200; files after OPTIMIZE+ZORDER: 55; speedup: `12.2x`; pruning: `55.0x` | The wall-clock result exceeds the 3x target, and the deterministic file-pruning metric exceeds the 10x fallback. The important reading is that Z-order makes file stats useful for point filters. |
| 3 | Time travel, MERGE, RESTORE | MERGE 100K rows succeeded; final history has 5 versions including RESTORE; `score < 0` after restore is 0 | The table keeps a usable audit trail. RESTORE creates a new version, so rollback is itself auditable rather than an invisible filesystem edit. |
| 4 | Medallion Bronze -> Silver -> Gold | Bronze: 200,000 rows; Silver: 190,052 rows; dedup dropped 9,948 rows; Gold: 8 dates x 3 models = 24 rows | Silver is smaller than Bronze because retries are deduplicated. Gold has p50/p95 latency, token totals, error rate, and cost for more than the 7-day target. |
| 5 | Iceberg catalog, hidden partitioning, metadata, evolution | Filter on `ts` reads 1 of 10 files; pruning ratio `10x`; metadata is 278.5% of toy table size; rename preserved `field_id=4`; specs `[1, 2]` coexist | Hidden partitioning prevents users from forgetting a partition predicate. Field IDs make rename metadata-only, and spec evolution lets old and new layouts remain readable. |
| 6 | Maintenance jobs | Compaction: 200 -> 11 files; clustering opens 1/10 files for point query; vacuum reclaimed bytes; 3 Delta orphans removed; checkpoint written; Iceberg expired to 3 snapshots and stranded manifests swept | Maintenance is not optional operations polish. The measured surprise is that expiry alone does not clean all physical files; orphan cleanup is a separate job. |
| 7 | Multimodal/vector storage | Random-read amplification: `200x`; int8 recall@10: `0.904`; topic fidelity: `1.000`; storage saved: `83%`; lakehouse erased hits: 0, external index erased hits: 8 | Inline blobs are fine for projection scans but bad for random frame reads because row groups are large. Vector DBs must be treated as derived indexes because delete propagation can fail. |
| 8 | Agent trajectories and provenance | Silver: 1,578 steps partitioned by `agent_version`; replay at pinned version matched exactly; 5 agent turns produced 1 catalog read; 4 Art. 10 buckets present; 334 UNCLASSIFIED rows excluded | Agent training must pin table versions for reproducibility. Provenance is not a label afterthought; it is a partitionable control used to exclude ungoverned training data. |

## Spark Notebook Status

The repo includes four Spark equivalents for NB1-NB4 under `notebooks-spark/`. They are currently not executed because Spark is not available locally:

```text
notebooks-spark/01_delta_basics.ipynb:      0 output cells
notebooks-spark/02_optimize_zorder.ipynb:   0 output cells
notebooks-spark/03_time_travel.ipynb:       0 output cells
notebooks-spark/04_medallion.ipynb:         0 output cells
```

The required runtime options are:

```bash
make spark-up
make spark-smoke
make spark-data
```

Then execute the four notebooks in Jupyter at `http://localhost:8888` with token `lakehouse`, or run them headlessly inside the container. This step requires downloading the Spark notebook Docker image, which was stopped on purpose.

## Strongest Findings

NB2 proves the practical value of clustering: the 12.2x speedup is useful, but the stronger evidence is `55.0x` pruning because it is less dependent on laptop noise.

NB6 is the most operationally important notebook. It shows that compaction, clustering, vacuum, orphan cleanup, and checkpointing are separate jobs. Expiring metadata without sweeping stranded files can leave storage bills unchanged.

NB7 exposes a common RAG lifecycle bug. The lakehouse table honored deletion immediately, while the external vector index still returned 8 erased documents. That makes delete propagation a correctness and compliance requirement, not just a sync optimization.

NB8 ties lakehouse mechanics to AI governance. A training run that pins Delta version 0 can be replayed exactly even after more rollouts arrive in version 1, and provenance buckets make trainable vs excluded corpus rows explicit.

## Reproducibility Notes

Commands already run successfully:

```bash
make run-all
make test
```

The executed lightweight notebooks and screenshots are ready. The Spark notebooks require a runtime download before they can be completed with real output.
