# BIDSification Pipelines

Two automated pipelines convert raw sourcedata into BIDS format:

1. **dcm2bids** — DICOM images to NIfTI + JSON (anatomical, functional, fieldmap, diffusion)
2. **Behavioral** — CSVs, EDF files, and PhysioLog DICOMs to BIDS events, behavioral, and physio files

Both produce files in the same `sub-*/ses-*/` directory tree. Task names must match
across pipelines (e.g. the BOLD NIfTI `task-TBencoding` must have a corresponding
`task-TBencoding_events.tsv`). The pipelines are independent and can run in any order.

**Exception registry:** `sourcedata/sub-XX/sub-XX_sessions.tsv` is the canonical
source of truth for session-level anomalies. Companion sidecar:
`sourcedata/sessions.json`.

---

## Session Structure

| Session(s) | Type | Prefix | Description |
|------------|------|--------|-------------|
| ses-01 | Anatomy | INIT | Structural MRI, resting-state |
| ses-02, ses-03 | Localizer | — | prf, floc, tone, auditory (varies by subject) |
| ses-04 | TB first | TB | First cued-recall session |
| ses-05–17 | TB middle | TB | Standard cued-recall sessions |
| ses-18 | TB last | TB | Final cued-recall session |
| ses-19–28 | Naturalistic | NAT | Free-recall with movie encoding |
| ses-29 | Behavioral only | — | No imaging (voice memos + behavioral only) |
| ses-30 | Final | FIN | Final memory tests + makeup localizers |

**Task naming convention:** Prefixed by session type —
`TBencoding`, `TBretrieval`, `TBmath`, `TBresting`, `TB2AFC`,
`NATencoding`, `NATretrieval`, `NATmath`, `NATresting`,
`FINretrieval`, `FINresting`, `FIN2AFC`, `FINtimeline`,
`INITresting`, `prf`, `floc`, `tone`, `auditory`, `motor`, `fixation`.

---

## Source Data Inventory

| Data type | Files | Sessions | Converter | BIDS output |
|-----------|-------|----------|-----------|-------------|
| Cued-recall behavioral CSVs | ~390 | ses-04–18 | timed_events.py | `*_events.tsv` + `.json` |
| Cued-recall recognition CSVs | ~45 | ses-04–18 | behavioral_to_beh.py | `*_beh.tsv` + `.json` |
| Free-recall PsychoPy CSVs (encoding) | ~60 | ses-19–28 | psychopy_encoding.py | `*_events.tsv` + `.json` |
| Free-recall PsychoPy CSVs (retrieval) | ~30 | ses-19–28 | psychopy_retrieval.py | `*_events.tsv` + `.json` |
| Free-recall math CSVs | ~30 | ses-19–28 | timed_events.py | `*_events.tsv` + `.json` |
| Final session behavioral CSVs | ~15 | ses-30 | timed_events.py, localizer_events.py, behavioral_to_beh.py | `*_events.tsv` / `*_beh.tsv` + `.json` |
| EyeLink EDF files | 87 (73 viable) | ses-19–28 | edf_to_physio.py | `*_recording-eye_physio.tsv.gz` + `.json` |
| Siemens PhysioLog DICOMs | 611 (232 convertible) | ses-01–28, 30 | physio_dcm.py | `*_recording-scanner_physio.tsv.gz` + `.json` |
| Audio WAV recordings | ~78 | ses-19–28 | None | Stays in sourcedata (future human annotation) |
| Scan questionnaire (Excel) | 81 | ses-04–30 | generate_sessions_tsv.py | `sub-XX_sessions.tsv` |
| Demographics / VVIQ / Debriefing (Excel) | 3 each | enrollment, ses-30 | generate_phenotype.py | `participants.tsv`, `phenotype/*.tsv` |

---

## Behavioral Pipeline

### Converters

All converters live in `code/mmmdata/raw2bids_converters/`. The orchestrator
(`run_all.py`) reads `file_inventory.csv` and dispatches each file to the
appropriate converter.

| Converter | Type | Input | Output | Files |
|-----------|------|-------|--------|-------|
| timed_events.py | timed_events | Cued-recall/math CSVs + timing CSVs | events TSV | ~390 |
| psychopy_encoding.py | psychopy_encoding | PsychoPy encoding CSVs | events TSV | 60 |
| psychopy_retrieval.py | psychopy_retrieval | PsychoPy retrieval CSVs | events TSV | 30 |
| behavioral_to_beh.py | behavioral_to_beh | Recognition/timeline CSVs | beh TSV | 51 |
| localizer_events.py | localizer_events | Localizer CSVs | events TSV | 9 |
| edf_to_physio.py | edf_to_physio | EyeLink EDF files | physio TSV.GZ (3 cols: x, y, pupil) | 73 |
| physio_dcm.py | physio_dcm | Siemens PhysioLog DICOMs | physio TSV.GZ (cardiac, pulse, resp) | 232 |

**Total output:** 834 conversions producing 1,298 data files + 1,298 JSON sidecars = 2,596 files.

### File Inventory

`file_inventory.csv` is the manifest that drives the orchestrator. It maps every
source file to a BIDS destination and conversion type. Regenerate with:

```bash
cd /gpfs/projects/hulacon/shared/mmmdata/code/mmmdata/raw2bids_converters
/home/bhutch/.local/envs/viz2psy/bin/python3 generate_inventory.py
```

This walks `sourcedata/sub-*/ses-*/` and classifies files by regex. It also
reads `edf_triage.csv` and `physio_triage.csv` to determine which EDF and
PhysioLog files are convertible.

### How to Run

**Prerequisites:** The viz2psy venv (`/home/bhutch/.local/envs/viz2psy`) provides
`eyelinkio`, `pandas`, and other dependencies required by the full converter suite.

```bash
# From the mmmdata repo (code/mmmdata/)
# Dry-run (preview what would be converted)
sbatch scripts/behavioral_dryrun.sbatch

# Full conversion
sbatch scripts/behavioral_convert.sbatch

# Validation against reference data in derivatives/bids_validation/
sbatch scripts/validate_step8.sbatch
```

Both sbatch scripts invoke `run_all.py` with the viz2psy Python. The dry-run
takes ~15 minutes (DICOM parsing for physio); the full conversion takes ~45 minutes
(EDF processing dominates).

**Agent interface:** The BIDS agent in mmmdata-agents exposes
`run_behavioral_conversion` and `validate_conversion` tools
(`mmmdata-agents/src/tools/conversion_tools.py`).

### EDF Triage

Not all 87 EyeLink EDF files are usable. Quality is assessed by
`edf_triage.py` (in `code/mmmdata/scripts/`), which:

1. Extracts scanner trigger pulses (discrete input value 255, every 1.5s = TR)
2. Trims to the scan window (first trigger to last trigger)
3. Computes gaze and pupil viability separately within the scan window
4. Includes a file if gaze OR pupil reaches 60% valid samples

**Results:**

| | sub-03 | sub-04 | sub-05 | Total |
|---|:--:|:--:|:--:|:--:|
| gaze+pupil | 6 | 25 | 6 | 37 |
| pupil_only | 18 | 1 | 17 | 36 |
| exclude | 6 | 3 | 5 | 14 |
| Total | 30 | 29 | 28 | 87 |

**73 included** (37 gaze+pupil + 36 pupil_only), **14 excluded** (all retrieval
runs where both channels fall below 60%). Output:
`raw2bids_converters/edf_triage.csv`.

Key patterns:
- **sub-04:** Reliably good (25/29 gaze+pupil)
- **sub-03:** Systematic calibration issues — pupil tracked but gaze mapping failed (18 pupil_only)
- **sub-05:** Gaze lost after 500→1000 Hz switch at ses-23; pupil remains usable (17 pupil_only)

### Scanner Physio

Siemens PhysioLog DICOMs contain PMU waveform data embedded in DICOM private tag
`(7FE1,1010)`. Each DICOM has 5 sections (ECG, PULS, RESP, EXT, ACQUISITION_INFO).
`physio_dcm.py` extracts and resamples to uniform time series.

**Triage results** (`raw2bids_converters/physio_triage.csv`):

| Status | Count | Description |
|--------|-------|-------------|
| COMPLETE | 208 | Recording covers >=90% of scan duration |
| PARTIAL | 24 | 50–90% coverage |
| TRUNCATED | 7 | <50% coverage |
| INFO_ONLY | 372 | Metadata stubs only — no waveform data (connectivity issue) |

**232 convertible** (COMPLETE + PARTIAL). Each produces 3 files:
- `_recording-cardiac_physio.tsv.gz` + `.json` (ECG, 1000 Hz)
- `_recording-pulse_physio.tsv.gz` + `.json` (pulse ox, 500 Hz)
- `_recording-respiratory_physio.tsv.gz` + `.json` (resp belt, 125 Hz)

StartTime in JSON sidecars is relative to the first BOLD volume, computed from
ACQUISITION_INFO volume timing.

---

## dcm2bids Pipeline

### Overview

The dcm2bids pipeline converts DICOM images to BIDS NIfTI+JSON using the
[dcm2bids](https://unfmontreal.github.io/Dcm2Bids/) tool. Config generation is
automated via `mmmdata/src/python/dcm2bids_config/`:

- **Session definitions** (`session_defs.py`) — dataclass-based task/fmap definitions per session type
- **Config builder** (`config_builder.py`) — generates dcm2bids JSON configs
- **DICOM inspector** (`dicom_inspect.py`) — auto-detects fieldmap series numbers
- **Override system** (`overrides.py`) — per-subject TOML overrides for session idiosyncrasies
- **CLI** (`cli.py`) — command-line interface with dry-run, force, batch modes

### Session Schedule

Codified in `session_defs.py`:

```
ses-01  → anatomy
ses-02  → localizer
ses-03  → localizer
ses-04  → tb_first
ses-05–17 → tb_middle
ses-18  → tb_last
ses-19–28 → naturalistic
ses-30  → final
```

### Fieldmap Matching

Three scenarios for fieldmap-to-EPI assignment:

| Scenario | Condition | Strategy |
|----------|-----------|----------|
| A | SeriesDescription contains encoding/retrieval suffix | Match by SeriesDescription alone |
| B | Generic descriptions (e.g. `se_epi_ap`) | Match by SeriesNumber (auto-detected from DICOM dirs) |
| C | Extra fieldmaps (re-entry, protocol restart) | Explicit override in `overrides.toml` |

The CLI auto-detects scenario A vs B by inspecting DICOM directory names.

### Override System

Per-subject override files at `dcm2bids_configfiles/sub-XX/overrides.toml`
handle sessions that deviate from the standard schedule. Override capabilities:

- `template` — use a different session type template
- `tasks` — full custom task list (for localizer/final sessions)
- `fmap_series` — explicit fieldmap series numbers
- `exclude_tasks` / `extra_tasks` — modify task list
- `runs` — explicit run number lists (for split-group tasks)
- `run_protocols` — override DICOM protocol names for specific runs
- `run_series` — add SeriesNumber to BOLD/SBRef criteria

#### Sessions Requiring Overrides

**Sub-03:** ses-02 (localizer), ses-03 (localizer), ses-10 (re-entry → 3 fmap groups),
ses-30 (final: SeriesNumber disambiguation for FINretrieval runs 1-2)

**Sub-04:** ses-02 (localizer), ses-03 (localizer, no floc), ses-04 (6 prepended
fLOC runs split across fmap groups), ses-20 (`attempt2` protocol name),
ses-30 (final: standard structure)

**Sub-05:** ses-02 (localizer), ses-03 (localizer, re-entry), ses-05 (re-entry → 3
fmap groups), ses-19 (3 fmap groups with duplicate descriptions), ses-24
(duplicate run1 series, SeriesNumber needed), ses-30 (final: hybrid fmap matching)

### How to Run

```bash
# Generate configs for a subject (dry-run)
cd /gpfs/projects/hulacon/shared/mmmdata/code/mmmdata
python scripts/run_dcm2bids.py --subject sub-03 --session all --dry-run

# Generate configs (write to disk)
python scripts/run_dcm2bids.py --subject sub-03 --session all --force

# Generate + run dcm2bids conversion
python scripts/run_dcm2bids.py --subject sub-03 --session ses-06

# Submit as SLURM job
sbatch scripts/dcm2bids.sbatch  # (with SUBJECT/SESSION env vars)
```

**Agent interface:** The BIDS agent in mmmdata-agents exposes
`generate_dcm2bids_config` and `submit_dcm2bids` tools
(`mmmdata-agents/src/tools/conversion_tools.py`).

---

## Phenotype & Session Metadata

### Session metadata

`generate_sessions_tsv.py` (in `code/mmmdata/scripts/`) produces per-subject `sessions.tsv` files
from `mmm_scanlog.xlsx`. Merges the BySession sheet (scan dates, equipment) with
the scan_questionaire sheet (session-level responses) and the pipeline exception
registry. Output: `sourcedata/sub-XX/sub-XX_sessions.tsv` + `sourcedata/sessions.json`.

### Phenotype

`generate_phenotype.py` (in `code/mmmdata/scripts/`) converts Excel questionnaire data to BIDS format:

| Source | Output | Subjects |
|--------|--------|----------|
| Demographics Form.xlsx | `participants.tsv` + `.json` | 3 |
| VVIQ.xlsx | `phenotype/vviq.tsv` + `.json` | 3 |
| mmm_scanlog.xlsx (Final Debriefing) | `phenotype/final_debriefing.tsv` + `.json` | 3 |

### Supplementary scripts

- `copy_audio_and_transcriptions.py` — organizes voice memos and transcription CSVs into sourcedata
- `copy_psychopy_companions.py` — copies PsychoPy .log and .psydat files alongside .csv files

All supplementary scripts live in `code/mmmdata/scripts/`.

---

## Validation

### Behavioral pipeline

`validate_step8.py` (in `code/mmmdata/scripts/`) compares new BIDS output against 532 reference files
in `derivatives/bids_validation/eventfiles/` (produced by standalone scripts).

Comparison approach:
- **Category A** (same columns): full data comparison with +/-0.001s tolerance
- **Category B** (shared columns): compare overlapping columns only
- **Category C** (different structure): row count + manual review flag

Results: all files match or show expected design differences (fewer but more
semantically focused events in the new converters).

### dcm2bids pipeline

Validation by regression diff: regenerate all configs with the automated system
and compare against existing hand-generated configs in
`derivatives/bids_validation/`. Output JSON should be functionally equivalent.

---

## BIDS Output Structure

```
sub-XX/
  ses-YY/
    anat/    ← dcm2bids (T1w, T2w)
    func/    ← dcm2bids (BOLD NIfTI) + behavioral (events/physio TSVs)
      sub-XX_ses-YY_task-TBencoding_run-01_bold.nii.gz        ← dcm2bids
      sub-XX_ses-YY_task-TBencoding_run-01_events.tsv          ← behavioral
      sub-XX_ses-YY_task-TBencoding_run-01_events.json         ← behavioral
      sub-XX_ses-YY_task-TBencoding_run-01_recording-eye_physio.tsv.gz     ← behavioral
      sub-XX_ses-YY_task-TBencoding_run-01_recording-cardiac_physio.tsv.gz ← behavioral
      sub-XX_ses-YY_task-TBencoding_run-01_recording-pulse_physio.tsv.gz   ← behavioral
      sub-XX_ses-YY_task-TBencoding_run-01_recording-respiratory_physio.tsv.gz ← behavioral
    beh/     ← behavioral (recognition, timeline)
      sub-XX_ses-YY_task-TB2AFC_run-01_beh.tsv
    dwi/     ← dcm2bids
    fmap/    ← dcm2bids
  sub-XX_sessions.tsv           ← generate_sessions_tsv.py
participants.tsv                ← generate_phenotype.py
phenotype/
  vviq.tsv                      ← generate_phenotype.py
  final_debriefing.tsv          ← generate_phenotype.py
```

---

## Protocol-to-Task Mapping

Used by both `physio_dcm.py` (for PhysioLog DICOMs) and the dcm2bids config
system (for imaging DICOMs).

| DICOM protocol pattern | BIDS task | Session type |
|------------------------|-----------|-------------|
| `cued_recall_encoding_runN` | TBencoding | ses-04–18 |
| `cued_recall_retrieval_runN` | TBretrieval | ses-04–18 |
| `cued_recall_math` | TBmath | ses-04–18 |
| `cued_recall_resting` | TBresting | ses-04–18 |
| `free_recall_encoding_runN` | NATencoding | ses-19–28 |
| `free_recall_retrieval_runN` | NATretrieval | ses-19–28 |
| `free_recall_math` | NATmath | ses-19–28 |
| `free_recall_resting` | NATresting | ses-19–28 |
| `Resting_baseline` | INITresting | ses-01 |
| `Resting` | FINresting | ses-30 |
| `fixation_calibration` | fixation | ses-30 |
| `final_cued_recall_runN` | FINretrieval | ses-30 |
| `localizer_prf_runN` | prf | ses-02–03 |
| `localizer_floc_runN` | floc | ses-02–04 |
| `localizer_tone` | tone | ses-02–03 |
| `localizer_auditory*` | auditory | ses-02–03, ses-30 |
| `localizer_motor_runN` | motor | ses-30 |

---

## Known Edge Cases

- **sub-04/ses-22:** EDF filename `s4s4s1r` is a typo (should be encoding run 1)
- **sub-03/ses-24:** Three EDFs mislabeled as retrieval; two are actually encoding (mapped via `EDF_ANOMALIES` in `generate_inventory.py`)
- **sub-04/ses-20:** Retry protocol `free_recall_retrieval_run1_attempt2` — handled via `run_protocols` override
- **sub-05/ses-24:** Duplicate `free_recall_retrieval_run1` series — needs SeriesNumber override
- **PhysioLog stubs:** 372 of 611 PhysioLog DICOMs are info-only (no waveform data due to connectivity issues) — automatically detected and skipped
- **Truncated PhysioLogs:** 7 files with <50% coverage — converted but partial

---

## Key Paths

| Resource | Path |
|----------|------|
| BIDS root | `/gpfs/projects/hulacon/shared/mmmdata/` |
| Sourcedata | `sourcedata/sub-XX/ses-YY/` |
| Behavioral converters | `code/mmmdata/raw2bids_converters/` |
| dcm2bids config system | `code/mmmdata/src/python/dcm2bids_config/` |
| dcm2bids generated configs | `code/dcm2bids_configfiles/sub-XX/` |
| Pipeline scripts | `code/mmmdata/scripts/` |
| Agent code | `code/mmmdata-agents/` |
| Validation reference | `derivatives/bids_validation/eventfiles/` |
| viz2psy venv | `/home/bhutch/.local/envs/viz2psy` |
