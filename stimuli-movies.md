---
title: Movie Stimuli
parent: Stimuli
grand_parent: Dataset Description
nav_order: 3
---

# Movie Stimuli (NAT Paradigm)

Short films (~4 minutes each) shown during naturalistic encoding sessions
(ses-19 through ses-28). The movie schedule is defined in
`movies/MMM movies - Sheet1.csv`.

- **Count**: ~40 unique titles across 10 sessions
- **Duration**: ~4 minutes each
- **Styles**: Animated, live-action, and stop-motion; with or without speech
- **Source**: Mix of short films and clips from longer films (sourced via a
  "memory search project" with collaborators Jordan & Kahlyn). Movie files are
  stored on Google Drive (links in `short films 4 minutes.rtf`).
- **Movie files**: 60 `.mov` files in `movies/movie_files/`, named
  `{Title}_trimmed_normalized_filtered.mov`
- **Movie cues**: 60 `.jpg` recall cue images in `movies/movie_cues/`, named
  `{Title}_cue.jpg` (see [Movie Cue Images](#movie-cue-images) below)

## Movie Cue Images

During free recall, participants are prompted with a movie title. If they cannot
recall which movie the title refers to, they are shown a **cue image** — a
single representative frame from the film. The cue is designed to orient the
participant without biasing their memory of the movie's content.

- **Count**: 60 images (one per movie)
- **Format**: JPG
- **Location**: `movies/movie_cues/{Title}_cue.jpg`

## Recall Conditions

Each movie in a session is assigned one of three recall conditions:

| Condition | Label | Description |
|-----------|-------|-------------|
| 1 | Same-session recall | Viewed and recalled within the same session |
| 2 | Next-session recall | Viewed this session, recalled in the following session |
| 3 | Multiple-repeat | Recurring film shown across many/all sessions |

Two films recur across all 10 sessions as condition 3: **"From Dad To Son"**
(animated, no speech) and **"The Bench"** (live-action, speech).

## Session Structure

Each session contains 6–8 movies:
- ~4 **condition 1** movies (same-session recall)
- ~2 **condition 2** movies (recalled next session)
- 2 **condition 3** movies (recurring)
- Sessions 2–10 also include 2 **carry-over** movies from the prior session
  (positions 7–8 at recall) that are recalled but not re-viewed

## Movie Schedule Metadata

`MMM movies - Sheet1.csv` columns:

| Column | Description |
|--------|-------------|
| `Session` | NAT session number (1–10, mapping to ses-19 through ses-28) |
| `Condition` | Recall condition: 1 = same-session, 2 = next-session, 3 = multi-repeat |
| `Movie name` | Title of the film |
| `Movie style` | Style description (e.g., "Animated, no speech", "Live-action, speech") |
| `Movie duration` | Duration (currently empty — all ~4 minutes) |
| `Cue position at Recall` | Order position (1–8) of this movie's recall cue; N/A for condition-2 movies |

Blank rows in the CSV separate sessions. Rows without a condition value are
carry-over movies from the prior session.

## Computational Features (viz2psy)

Movie frames and cue images are processed with
[viz2psy](https://github.com/hulacon/viz2psy), producing per-movie temporal
feature timeseries in `movies/viz2psy_scores/`.

For each movie, the output includes:
- **`{Title}_scores.csv`** — one row per sampled frame, indexed by `time` (seconds),
  with ~2,900 feature columns (memorability, emotion, CLIP embeddings, etc.)
- **`{Title}_scores.meta.json`** — feature definitions and provenance
- **`{Title}_scores_dashboard.html`** — interactive visualization of features over time
- **`{Title}_scores_frames/`** — extracted frame images (JPG)

Cue images are scored separately in a consolidated cue output file.

See the [viz2psy documentation](https://github.com/hulacon/viz2psy) for full
feature definitions.
