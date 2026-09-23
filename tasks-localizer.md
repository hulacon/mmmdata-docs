---
title: Localizer Tasks
parent: Experimental Design
grand_parent: Dataset Description
nav_order: 2
---

# Functional Localizer Tasks

Collected during ses-02, ses-03, and ses-30 (final session).

| BIDS task label | Description | Sessions | Runs |
|-----------------|-------------|----------|------|
| `task-prf` | Population receptive field mapping | ses-02, ses-03 | 3/session |
| `task-floc` | Functional localizer (category-selective) | ses-02, ses-03, ses-04 | 3–6/session |
| `task-auditory` | Auditory cortex localizer | first cohort: ses-02 or ses-03, and ses-30; regularised protocol: ses-02 and ses-03 | 1/session |
| `task-tone` | Tonotopy mapping | ses-02, ses-03 | 1/session (a make-up run exists where an earlier session's run was aborted) |
| `task-motor` | Motor cortex localizer | first cohort: ses-30; regularised protocol: ses-02 and ses-03 | first cohort: 2; regularised: 1/session |
| `task-fixation` | Fixation baseline | ses-30 | 1 |

These labels are expected to remain unchanged.

---

## PRF (task-prf)

**Source:** Experiment code adapted from the analyzePRF toolbox — <https://kendrickkay.net/analyzePRF/>

**Citation:** Kay, K. N., Winawer, J., Mezer, A., & Wandell, B. A. (2013). Compressive spatial summation in human visual cortex. *Journal of Neurophysiology*, 110(2), 481–494.

**Model:** Compressive Spatial Summation (CSS) — extends the standard 2D isotropic Gaussian pRF model (Dumoulin & Wandell, 2008) with a static power-law nonlinearity after spatial summation. Fitted parameters per voxel: center position (x, y), pRF size (sigma), compressive exponent, and gain.

### Design
- 3 runs per session, 300 s each
- Two stimulus types:
  - **Multibar** (exp 93): bars sweeping in 8 directions (same as RETBAR in the HCP 7T Retinotopy Dataset)
  - **Wedgering** (exp 94): combination of rotating wedges and expanding/contracting rings
- Acquisition order: [MWM] for ses-02, [WMW] for ses-03 (M = multibar, W = wedgering)
- A total of 6 runs (3 multibar, 3 wedgering) collected across two localizer sessions
- Stimuli filled a circular region with diameter 10.6°
- A small semi-transparent fixation dot (0.2° × 0.2°) was present at center throughout
- Participants maintained fixation on a central dot that switched randomly between three colors (black, white, red) every 1–5 s and pressed a button at each color change
- Stimulus frame rate: 15 Hz; pre-generated aperture matrices stored in MATLAB workspace

**Fitted maps:** `derivatives/prf/` — see [Derivatives](derivatives.md#prf-maps-prf).

### Data inventory
Per-subject run counts are not listed here; query the catalog (`inventory/catalog.duckdb`, see [Data Organization](data-organization.md)).

---

## fLoc (task-floc)

**Source:** Stanford VPNL fLoc localizer — <https://vpnl.stanford.edu/fLoc/>

**Citation:** Stigliani, A., Weiner, K. S., & Grill-Spector, K. (2015). Temporal processing capacity in high-level visual cortex is domain specific. *Journal of Neuroscience*, 35(36), 12412–12424.

### Design
- Miniblock format: 8 images per 4-second block (500 ms stimulus duration each)
- 5 stimulus domains, each with 2 subcategories:
  - **Characters:** pseudowords, numbers
  - **Bodies:** whole bodies, limbs
  - **Faces:** adults, children
  - **Places:** corridors, buildings (houses)
  - **Objects:** vehicles (cars), instruments
- Baseline condition: fixation
- Behavioral task: oddball detection (scrambled image among category images)
- 75 blocks per run (~300 s total)
- 3–6 runs per session; sessions with 6 runs used two stimulus sets
- Code written in MATLAB/Psychtoolbox-3

### Experiment code
- Original fLoc code: `sourcedata/shared/experiment_code/localizer/floc/` (Stigliani et al. V2.0, August 2015)
- Updated version: `sourcedata/shared/experiment_code/localizer/floc_new/` (V3.0, August 2017; 12 stimuli per block)
- Block-level timing in `.par` files; trial-level timing in detailed script files
- BIDS `_events.tsv` files for fLoc are block-level (written 2026-08-21; onsets corrected 2026-09-08)
- **Timing origin:** the fLoc program starts on a scanner pulse, shows a 12 s countdown (`et_run_fLoc.m`, `countDown = 12`; 8 TRs at 1.5 s), and only then starts the clock its trial times are logged on. BIDS onsets therefore equal the logged times **plus 12 s**; a run is 208 volumes = 12 s countdown + 300 s of blocks. The logged scanner-trigger pulses sit at a fixed phase of the task clock and cannot reveal this whole-TR offset — only a fit to the BOLD can (see `mmmdata/src/python/raw2bids_converters/floc_events.py`).

### Data inventory
Per-subject run counts are not listed here; query the catalog (`inventory/catalog.duckdb`, see [Data Organization](data-organization.md)).

---

## Motor Localizer (task-motor)

**Source:** Adapted from the motor localizer in Tang et al. (2023) / LeBel et al. (2023), including the "speak" (covert narrative) condition used to define Broca's area.

> *Corrected 2026-09-08.* This section previously said the MMMData version omits the "speak" condition and listed five conditions. It does not: the stimulus program's condition list and on-screen instructions both carry six, and every run presents five `speak` blocks. Verified against `localizer_motor.py` and all motor timing CSVs.

**Citations:**
- Tang, J., LeBel, A., Jain, S. et al. Semantic reconstruction of continuous language from non-invasive brain recordings. *Nature Neuroscience*, 26, 858–866 (2023). <https://doi.org/10.1038/s41593-023-01304-9>
- LeBel, A., et al. A natural language fMRI dataset for voxelwise encoding models. *Scientific Data*, 10, 555 (2023). <https://doi.org/10.1038/s41597-023-02437-z>

### Design
- Block design, 20-second blocks; 30 blocks = 600.0 s per run, which exactly fills the acquisition
- 6 conditions x 5 blocks each:

  | Condition | On-screen cue | Instruction given to the participant |
  |---|---|---|
  | `hand` | hand | make small finger-drumming movements |
  | `foot` | foot | make small foot and toe movements |
  | `mouth` | mouth | make small nonsense vocalizations (e.g. "balabalabala") |
  | `speak` | speak | self-generate a narrative without vocalization |
  | `saccade` | saccade | look around for the duration of the task |
  | `rest` | rest | rest with eyes open |

- The instruction wording above is the on-screen text of the program that produced every motor run. A second program (the PsychoPy Builder version) exists in the experiment code with **different** wording for the same six cues — notably a *silent* lip protrusion for `mouth` rather than a vocalized one — but no run in the dataset is attributable to it. Whether any cohort was given that wording verbally is an open question (mmmdata-agents `docs/OPEN-QUESTIONS.md` Q19); until it is answered, treat `mouth` as vocalized, and see the caveat below.
- **The block order is frozen.** The program shuffles the condition list under a hardcoded seed, so every run of every subject presents the identical 30-block sequence. Two runs in a session are not independent orderings, and order effects are perfectly confounded across subjects. Do not treat the sequence as randomised.
- **Timing origin:** the program waits on the scanner sync key and resets its clock immediately afterwards — no countdown, no lead-in, no `launchScan`. The task clock is the scan clock, so onsets need no shift.
- Run counts differ by cohort: the first cohort ran two runs in the final session (BOLD carries a `run` entity); under the regularised protocol motor moved to the localizer sessions at one run per session (BOLD carries **no** `run` entity, though the source CSV filename still numbers runs per subject). Query the catalog for what exists.
- Implemented in PsychoPy. Two versions exist in sourcedata and **only the hand-coded one produced the data** — the Builder `.psyexp`/`_lastrun.py` writes no timing CSV.

### Experiment code
- **Hand-coded (this is what ran):** `mmmsourcedata/shared/experiment_code/final_cued_recall/final_cued_recall_localizers/localizers/motor/localizer_motor.py` — waits on the sync key, resets the clock, presents the seeded block list, and writes the `*_timing.csv` the converter reads.
- PsychoPy Builder (**not** the version that ran): `mmmsourcedata/shared/experiment_code/localizer/other localizers/motor/motor.psyexp` and its generated `.py`. Neither writes a timing CSV, and the Builder condition list is drawn with replacement rather than seeded — do not read the design off it.
- Converter: `mmmdata/src/python/raw2bids_converters/localizer_events.py` (`convert_motor`)

(Other paths in this file still carry the pre-migration `sourcedata/` prefix; they now live under `mmmsourcedata/`.)

### Data inventory
Per-subject run counts are not listed here; query the catalog (`inventory/catalog.duckdb`, see [Data Organization](data-organization.md)).

---

## Auditory Category Localizer (task-auditory)

**Source:** Adapted from the auditory cortex localizer in Tang et al. (2023) / LeBel et al. (2023). The original protocol used 10 repeats of a 1-minute stimulus (20 s music, 20 s speech, 20 s nature sounds) with repeatability-based ROI definition. The MMMData version uses a single continuous auditory stimulus containing music, speech, and natural sounds (a 611 s file, presented in ~562 s; see Design).

**Citations:**
- Tang, J., LeBel, A., Jain, S. et al. Semantic reconstruction of continuous language from non-invasive brain recordings. *Nature Neuroscience*, 26, 858–866 (2023). <https://doi.org/10.1038/s41593-023-01304-9>
- LeBel, A., et al. A natural language fMRI dataset for voxelwise encoding models. *Scientific Data*, 10, 555 (2023). <https://doi.org/10.1038/s41597-023-02437-z>

> *Corrected 2026-09-23.* This section previously gave the stimulus as ~611 s (and before that ~562 s) and described one program. Both durations are real: the file is 611.3 s, but it was **played in ~561.65 s** (see "Playback rate" below). And two program versions produced the data, with different trial windows; only one of them writes a timing record.

### Design
- Single continuous auditory stimulus (music, speech and natural sounds in one WAV, `auditory_localizer_filtered.wav`, 44.1 kHz), followed by silence with a fixation dot to the end of the trial window
- 1 run per session; no lead-in: the program waits on the scanner sync key (`apostrophe`), zeroes its clock there, and starts playback within the first second
- The stimulus is one block with no internal condition structure in the events. The music / speech / nature segments exist only inside the audio file, and their timing is not recorded anywhere in the dataset, so a category contrast needs segment timings derived from the file (then time-scaled, see below)
- **Playback rate.** The file is 44.1 kHz, but every timing record logs the sound lasting 561.64–561.69 s, which is the file length × 44.1/48 (611.32 s → 561.65 s). The tone task shows the same ratio. So the audio device most likely played the 44.1 kHz samples at 48 kHz: ~8.8 % faster than the file, and pitched up ~1.5 semitones. This is inferred from timing records in two tasks, not checked against a recording of the audio output. **Anything derived from the WAV (segment times, acoustic features) must be scaled by 44100/48000 = 0.91875 before it is aligned to the BOLD.**
- Two program versions, told apart by the record they leave:

  | Version | Used for | Trial window | After the trial | Timing record |
  |---|---|---|---|---|
  | `localizer/other localizers/auditory/localizer_auditory.py` | the first cohort's ses-02/ses-03 run | 600 s | 12 s lead-out (outside the 600 s acquisition) | **none** — the script raises a `KeyError` (`info['tone']`, a leftover from the tone script it was copied from) at the point it builds the CSV, so it never writes one |
  | `final_cued_recall/final_cued_recall_localizers/localizers/auditory/localizer_auditory.py` | ses-30, and the regularised protocol's ses-02/ses-03 | 612 s | nothing | `localizer_auditory_subj#_sess#_run#_<datetime>_timing.csv` with `stim_start`, `stim_end`, `stim_fixation_start`, `stim_fixation_end` |

- The acquisition matches the trial window: 400 volumes (600 s) for the first version, 408 volumes (612 s) for the second. So a run holds the whole ~562 s stimulus plus 38–50 s of silence
- Because the first version writes no record, events for those runs can only be inferred: the stimulus starts at the trigger plus an unmeasured latency (0.23–0.90 s in the runs that do have a record) and lasts ~561.65 s. A sidecar for such a run has to say its timing is inferred
- Implemented in PsychoPy (a Builder `.psyexp` also exists; like motor's, it is not what ran)

### Experiment code
- First-cohort localizer sessions: `mmmsourcedata/shared/experiment_code/localizer/other localizers/auditory/localizer_auditory.py`
- Final session and regularised protocol: `mmmsourcedata/shared/experiment_code/final_cued_recall/final_cued_recall_localizers/localizers/auditory/localizer_auditory.py`
- PsychoPy Builder (not what ran): `mmmsourcedata/shared/experiment_code/localizer/other localizers/auditory/auditory.psyexp`
- Converter: `mmmdata/src/python/raw2bids_converters/localizer_events.py` (`convert_auditory`)

### Data inventory
Per-subject run counts are not listed here; query the catalog (`inventory/catalog.duckdb`, see [Data Organization](data-organization.md)).

---

## Tonotopy Mapping (task-tone)

**Source:** PsychoPy implementation by Futing Zou, based on the phase-encoded tonotopy paradigm from Da Costa et al. (2011, 2013).

**Citations:**
- Da Costa, S., van der Zwaag, W., Marques, J. P., Frackowiak, R. S. J., Clarke, S., & Saenz, M. (2011). Human primary auditory cortex follows the shape of Heschl's gyrus. *Journal of Neuroscience*, 31(40), 14067–14075. <https://doi.org/10.1523/JNEUROSCI.2000-11.2011>
- Da Costa, S., van der Zwaag, W., Miller, L. M., Clarke, S., & Saenz, M. (2013). Tuning in to sound: Frequency-selective attentional filter in human primary auditory cortex. *Journal of Neuroscience*, 33(5), 1858–1863. <https://doi.org/10.1523/JNEUROSCI.4405-12.2013>

> *Corrected 2026-09-23.* This section previously gave an 8 s lead-in, a 28 s sweep, and "run 1 low-to-high, run 2 high-to-low" as if the run number were a within-session index. The lead-in is commented out in the program. The 28 s sweep plays in ~25.75 s. And the "run number" is typed in by the operator, who in practice entered the localizer-session number.

### Design
- 15 trials per run, 32 s each: a swept-tone WAV, then silence with a fixation dot to the 32 s boundary. The 32 s cycle is set by the program's trial timer, so the phase-encoding period is exactly 32 s
- **No lead-in.** `leading_in_time = 8.0` is defined, but the call that uses it is commented out. The program waits on the scanner sync key, zeroes its clock there, and trial 1 starts at the trigger. 15 × 32 s = 480.0 s = 320 volumes × 1.5 s, so the design fills the acquisition exactly. The 12 s lead-out runs after the last trial, outside the acquisition
- **Sweep duration and frequencies.** The WAVs are 28.01 s at 44.1 kHz. Every timing record logs the sweep lasting 25.74–25.77 s, which is 28.01 s × 44.1/48. So the tones were most likely played ~8.8 % fast, as in the auditory localizer: every frequency is ×1.088 the file's (~+1.5 semitones), and each sweep lasts ~25.75 s, followed by ~6.25 s of silence. A tonotopic analysis that maps response phase to frequency must use the presented frequencies and sweep time, not the file's
- **Sweep direction** is chosen by the "Run #" the operator types at launch: 1 → `pure_tones_low_to_high_filtered.wav`, 2 → `pure_tones_high_to_low_filtered.wav`. Any other value raises an error. Operators entered 1 in the first localizer session and 2 in the second, so in practice **direction is a session-level property: ses-02 low-to-high, ses-03 high-to-low**. It is carried as a `direction` column in the events, so it survives concatenation across runs
- The timing CSV is named `localizer_tone_subj#_sess#_run#_timing.csv` and written with no existence check, so a second run launched with the same fields **silently overwrites** the first run's record
- Participants were told to close their eyes and focus on the tones
- TR 1.5 s, 320 volumes per run

### Experiment code
- `mmmsourcedata/shared/experiment_code/localizer/other localizers/tone?/localizer_tone.py` (the directory name really ends in `?`)
- Stimulus WAV files in its `tone/` subdirectory (unfiltered and `_filtered` variants; the program plays the `_filtered` ones)
- Converter: `mmmdata/src/python/raw2bids_converters/localizer_events.py`

### Data inventory
Per-subject run counts are not listed here; query the catalog (`inventory/catalog.duckdb`, see [Data Organization](data-organization.md)).

---

## Fixation (task-fixation)

Fixation baseline scan collected during ses-30 (final session). Participants maintained fixation on a central dot. One run per subject.

Used as calibration data for gaze reconstruction from BOLD signal (PEER / DeepMReye).

`_events.tsv` files for fixation runs were written 2026-08-21.

### Data inventory
Per-subject run counts are not listed here; query the catalog (`inventory/catalog.duckdb`, see [Data Organization](data-organization.md)).
