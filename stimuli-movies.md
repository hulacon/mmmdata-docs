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
  `{Title}_cue.jpg`

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
