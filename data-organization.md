---
title: Data Organization
parent: Dataset Description
has_children: true
nav_order: 5
---

# Data Organization

MMMData follows the [Brain Imaging Data Structure (BIDS) v1.9.0](https://bids-specification.readthedocs.io/)
standard. Data are organized into three tiers:

- **BIDS raw**: Converted NIfTI + JSON + events TSV files
- **Source data**: Original DICOMs, PsychoPy output, audio recordings — stored
  outside the BIDS tree at `/gpfs/projects/hulacon/shared/mmmsourcedata/` so
  the BIDS dataset can be shared without exposing PII (see sourcedata.md)
- **Derivatives**: Preprocessed outputs and analysis products (fMRIPrep,
  MRIQC, GLMsingle, hippocampal segmentations, stimulus features, …) — see
  [Derivatives & Preprocessing](derivatives.md)

## Per-Session Scan Inventories (`scans.tsv`)

Each subject/session directory contains a `sub-XX_ses-YY_scans.tsv` file
listing every primary NIfTI scan in that session. These are
[BIDS-standard scans files](https://bids-specification.readthedocs.io/en/stable/modality-agnostic-files.html#scans-file)
extended with custom columns:

| Column | Description |
|--------|-------------|
| `filename` | Relative path to the NIfTI file (BIDS required) |
| `acq_time` | Acquisition timestamp from JSON sidecar |
| `n_volumes` | Number of volumes (4th dimension; 1 for 3D scans) |
| `duration_s` | Scan duration in seconds |
| `n_events` | Row count of matching `_events.tsv` (n/a if none) |
| `physio_cardiac`, `physio_pulse`, `physio_respiratory` | Whether each physio channel exists |
| `eyetracking` | Whether eyetracking recording exists |
| `has_sbref`, `has_json`, `has_events_json` | Companion file presence |

Custom columns are documented in the BIDS-root `scans.json` sidecar (via
BIDS inheritance). Regenerate with:

```bash
python scripts/build_scans_tsv.py   # from the mmmdata repo, in the project env
```

## Catalog (the way to ask what exists)

`inventory/catalog.duckdb` is the queryable index of the dataset — raw BIDS
files, derivative trees, and QC state — keyed by BIDS entities
(`subject, session, task, run, suffix, space, res, desc, variant`) plus
pipeline version. It is the intended answer to "which sessions exist", "what
has fMRIPrep covered", "is X complete".

Rebuild it with the duckbrain catalog engine (~40 s):

```bash
python -m duckbrain.catalog rebuild --root /gpfs/projects/hulacon/shared/mmmdata
```

**Prefer a catalog query to any count written in these docs.** Numbers in
prose go stale silently; this page deliberately does not quote subject
counts, session counts, or per-pipeline completeness.

## Expectations & validation

`expectations/dataset.toml` (in the BIDS root) declares what the dataset
*should* contain — expected tasks, run counts, volume counts, event
structures, physio/eyetracking coverage, and roughly 100 documented
per-subject/session exceptions. It is ingested into the catalog by
`mmmdata/scripts/catalog_expectations.py`; query the `resolution` view to
compare declared expectations against what is actually present.

> **Superseded.** Two earlier mechanisms are no longer live and should not be
> used or cited:
>
> - **`dataset_expectations.toml`** (in this docs repo) was tombstoned
>   2026-08-17. Nothing parses it at runtime; its numbers predate 2026-04 and
>   several are contradicted by a later audit. It is retained only as the
>   historical record of the acquisition exceptions in their original prose.
>   Successor: `expectations/dataset.toml`.
> - **`inventory/manifest.db`** (SQLite) was the previous aggregate store,
>   last rebuilt in March 2026. It predates most current derivatives and is
>   not maintained. Successor: `inventory/catalog.duckdb`.

---

*Verified against the filesystem on 2026-08-20.*
