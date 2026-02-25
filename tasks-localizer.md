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
| `task-auditory` | Auditory cortex localizer | ses-02 or ses-03, ses-30 | 1/session |
| `task-tone` | Tonotopy mapping | ses-02, ses-03 | 1–2/session |
| `task-motor` | Motor cortex localizer | ses-30 | 2 |
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

### Data inventory
| Subject | Session | Runs |
|---------|---------|------|
| sub-03 | ses-02 | 3 |
| sub-03 | ses-03 | 3 |
| sub-04 | ses-02 | 3 |
| sub-04 | ses-03 | 3 |
| sub-05 | ses-02 | 3 |
| sub-05 | ses-03 | 3 |

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

### Data inventory
| Subject | Session | Runs |
|---------|---------|------|
| sub-03 | ses-03 | 6 |
| sub-04 | ses-04 | 6 |
| sub-05 | ses-02 | 4 |
| sub-05 | ses-03 | 3 |

---

## Motor Localizer (task-motor)

**Source:** Adapted from the motor localizer in Tang et al. (2023) / LeBel et al. (2023). The original protocol included a sixth "speak" (covert narrative) condition used to define Broca's area; the MMMData version omits this condition.

**Citations:**
- Tang, J., LeBel, A., Jain, S. et al. Semantic reconstruction of continuous language from non-invasive brain recordings. *Nature Neuroscience*, 26, 858–866 (2023). <https://doi.org/10.1038/s41593-023-01304-9>
- LeBel, A., et al. A natural language fMRI dataset for voxelwise encoding models. *Scientific Data*, 10, 555 (2023). <https://doi.org/10.1038/s41597-023-02437-z>

### Design
- Block design with 20-second blocks
- 5 conditions: foot movement, mouth movement, saccade (eye movement), hand movement, and rest
- 2 runs per subject, collected during ses-30 (final session)
- Implemented in PsychoPy (both Builder `.psyexp` and hand-coded `.py` versions exist)

### Experiment code
- PsychoPy Builder: `sourcedata/shared/experiment_code/localizer/other localizers/motor/motor.psyexp`
- Hand-coded: `sourcedata/shared/experiment_code/final_cued_recall/final_cued_recall_localizers/localizers/motor/`

### Data inventory
| Subject | Session | Runs | Events? |
|---------|---------|------|---------|
| sub-03 | ses-30 | 2 | Yes |
| sub-04 | ses-30 | 2 | Yes |
| sub-05 | ses-30 | 2 | Yes |

---

## Auditory Category Localizer (task-auditory)

**Source:** Adapted from the auditory cortex localizer in Tang et al. (2023) / LeBel et al. (2023). The original protocol used 10 repeats of a 1-minute stimulus (20 s music, 20 s speech, 20 s nature sounds) with repeatability-based ROI definition. The MMMData version uses a single continuous ~562 s auditory stimulus containing music, speech, and natural sounds.

**Citations:**
- Tang, J., LeBel, A., Jain, S. et al. Semantic reconstruction of continuous language from non-invasive brain recordings. *Nature Neuroscience*, 26, 858–866 (2023). <https://doi.org/10.1038/s41593-023-01304-9>
- LeBel, A., et al. A natural language fMRI dataset for voxelwise encoding models. *Scientific Data*, 10, 555 (2023). <https://doi.org/10.1038/s41597-023-02437-z>

### Design
- Single continuous auditory stimulus (~562 s) followed by a post-stimulus fixation period (~50 s)
- Stimulus content: music, speech, and natural sounds (single WAV file: `auditory_localizer_filtered.wav`)
- 1 run per session
- Collected during ses-02 or ses-03 (1st or 2nd localizer session) and again during ses-30 (final session)
- Implemented in PsychoPy (both Builder `.psyexp` and hand-coded `.py` versions exist)

### Experiment code
- PsychoPy Builder: `sourcedata/shared/experiment_code/localizer/other localizers/auditory/auditory.psyexp`
- Hand-coded: `sourcedata/shared/experiment_code/localizer/other localizers/auditory/localizer_auditory.py`

### Data inventory
| Subject | Session | Runs | Events? |
|---------|---------|------|---------|
| sub-03 | ses-02 | 1 | No |
| sub-03 | ses-30 | 1 | Yes |
| sub-04 | ses-02 | 1 | No |
| sub-04 | ses-30 | 1 | Yes |
| sub-05 | ses-03 | 1 | No |
| sub-05 | ses-30 | 1 | Yes |

Events for ses-30 were converted via `mmmdata/raw2bids_converters/localizer_events.py`. Events for ses-02/ses-03 are missing (behavioral timing CSVs not collected for those sessions).

---

## Tonotopy Mapping (task-tone)

**Source:** Custom in-house implementation. Follows a standard phase-encoding tonotopy paradigm using swept pure tones in ascending and descending frequency order. Written in PsychoPy. Not derived from any specific published protocol.

### Design
- 15 trials per run, 32 s each (28 s swept tone + 4 s ITI)
- Run 1: low-to-high frequency sweep (`pure_tones_low_to_high_filtered.wav`)
- Run 2: high-to-low frequency sweep (`pure_tones_high_to_low_filtered.wav`)
- 8 s leading-in silence, 12 s leading-out silence
- TR: 1.5 s, 320 volumes per run (~480 s total)
- Participants instructed to close their eyes and focus on the tones
- 1 run per session (some sessions have 2 runs with both sweep directions)

### Experiment code
- `sourcedata/shared/experiment_code/localizer/other localizers/tone/localizer_tone.py`
- Stimulus WAV files in `tone/` subdirectory

### Data inventory
| Subject | Session | Runs |
|---------|---------|------|
| sub-03 | ses-02 | 1 |
| sub-03 | ses-03 | 1 |
| sub-04 | ses-02 | 1 |
| sub-04 | ses-03 | 1 |
| sub-05 | ses-02 | 1 |
| sub-05 | ses-03 | 2 |

No events.tsv files exist. Timing is deterministic and can be reconstructed from experiment code parameters.

---

## Fixation (task-fixation)

Fixation baseline scan collected during ses-30 (final session). Participants maintained fixation on a central dot. One run per subject.

Used as calibration data for gaze reconstruction from BOLD signal (PEER / DeepMReye).

### Data inventory
| Subject | Session | Runs |
|---------|---------|------|
| sub-03 | ses-30 | 1 |
| sub-04 | ses-30 | 1 |
| sub-05 | ses-30 | 1 |
