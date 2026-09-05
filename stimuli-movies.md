---
title: Movie Stimuli
parent: Stimuli
grand_parent: Dataset Description
nav_order: 3
---

# Movie Stimuli (NAT Paradigm)

Short films (~4 minutes each) shown during naturalistic encoding sessions
(ses-19 through ses-28). The movie schedule is defined in
`movies/MMM_movies_Sheet1.csv`.

- **Count**: 60 unique titles across 10 sessions
- **Duration**: ~4 minutes each
- **Styles**: Animated, live-action, and stop-motion; with or without speech
- **Source**: Mix of short films and clips from longer films (sourced via a
  "memory search project" with collaborators Jordan & Kahlyn). Movie files are
  stored on Google Drive (links in `short_films_4_minutes.rtf`).
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

> **Unverified:** the per-session counts above are approximate. Taken
> literally (~6 new films × 10 sessions + 2 recurring) they give 62 titles,
> but the stimulus registry has 60. The exact schedule is
> `MMM_movies_Sheet1.csv` / the stimulus registry, not this list.

## Movie Schedule Metadata

`MMM_movies_Sheet1.csv` columns:

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

## Movie Annotations

Hand-annotated event descriptions for each movie, stored as Excel workbooks in
`movies/movie_annotations/`. Each movie has one annotation file named
`{Title}_annotation_master_{initials}.xlsx`, where `{initials}` identifies the
annotator.

- **Count**: 62 annotation files. Coverage is **not** one-per-movie:
  `finders-fee` has no annotation file, and three files (Fargo, Migration,
  Paddington) describe movies never shown in the experiment and are
  deliberately excluded from the stimulus registry
- **Format**: XLSX
- **Location**: `movies/movie_annotations/`
- **Annotators**: Multiple team members (identified by initials: SL, JL, HK, DP, LH)

## Computational Features

Movie frames and cue images are processed with
[viz2psy](https://github.com/hulacon/viz2psy), producing per-movie temporal
feature timeseries.

Output lands in `derivatives/stimuli_features/movies/<stimulus-id>/`, one CSV
per model on a **0.5 s grid**, keyed by `stimulus_id`. Consumers read the
assembled tables — `psytwill/movies_frames_features.parquet` for the visual
grid, `movies_audio_frames_features.parquet` for acoustics, and the
transcript, caption, and annotation groups alongside them. See
[Computational Stimulus Features](stimuli_features.md).

The pre-0.6.0 per-movie `{Title}_scores.csv` under `movies/viz2psy_scores/`,
together with its `{Title}_scores_dashboard.html` files and
`{Title}_scores_frames/` directories and the consolidated cue-score file, was
**deleted 2026-08-23**. Those files were wide, indexed by `time` alone, and
used the column names viz2psy 0.6.0 renamed. See the legacy note on
[Computational Stimulus Features](stimuli_features.md).

See the [viz2psy documentation](https://github.com/hulacon/viz2psy) for full
feature definitions.
