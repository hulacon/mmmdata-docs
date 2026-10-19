# MMMData Shared Documentation

> **These docs are best viewed at [hulacon.github.io/mmmdata](https://hulacon.github.io/mmmdata/).**
> This repo contains the source markdown files only; the rendered, navigable version lives at the link above.

Canonical dataset documentation for the Multi-Modal Memory Dataset (MMMData) project.

This repo is consumed as a **git submodule** by:
- [hulacon/mmmdata](https://github.com/hulacon/mmmdata) — rendered on the Jekyll site at `docs/doc/shared/`
- [jhutchin/mmmdata-agents](https://github.com/jhutchin/mmmdata-agents) — read as plain markdown at `docs/shared/`

## Structure

```
Overview
├── study-overview.md              # Study design, participants, scanner
│
├── Experimental Design/
│   ├── experimental-design.md     # Phase overview, shared tasks
│   ├── session-schedule.md        # 30-session canonical schedule
│   ├── tasks-localizer.md         # PRF, fLoc, auditory, tonotopy, motor
│   ├── tasks-trial-based.md       # Paired-associate encoding & cued recall
│   ├── tasks-naturalistic.md      # Movie-watching & free recall
│   └── tasks-final-session.md     # ses-29 behavioral + ses-30 final tests
│
├── Stimuli/
│   ├── stimuli.md                 # Overview, directory structure, assignment
│   ├── stimuli-visual.md          # NSD shared1000 images + COCO metadata
│   ├── stimuli-auditory.md        # Toronto Word Pool (twp1000) audio
│   ├── stimuli-movies.md          # Short films, recall conditions, annotations
│   └── stimuli_features.md        # Computational features (viz2psy)
│
├── Data Acquisition/
│   ├── data-acquisition.md        # Modality summary table
│   ├── mri-sequences.md           # MRI acquisition parameters
│   └── behav-physio.md            # Behavioral, eye-tracking, physiological
│
├── Data Organization/
│   ├── data-organization.md       # BIDS overview
│   ├── file-organization.md       # BIDS directory tree & naming conventions
│   ├── sourcedata.md              # Source data structure
│   ├── derivatives.md             # Derivative trees & what each contains
│   ├── preprocessing-spaces.md    # Output spaces & preprocessing steps
│   └── bidsification.md           # BIDSification pipeline & conversion details
│
├── quality-assurance.md           # QA procedure, IQMs, run-level decision records
│
├── access.md                      # Data access & availability
└── contributors.md                # Project team & affiliations
```

## Contributing

Edit files here, commit, and push to GitHub. **That is the whole workflow** —
a GitHub Action (`.github/workflows/notify-consumers.yml`) fires on every push
to `main` and bumps the submodule pointer in both consumer repos
automatically. Wait a minute or two, then `git pull` in the consumer.

Two rules that matter:

- **Never commit a submodule bump by hand.** The Action owns those commits;
  a manual bump races it.
- **Never push from a submodule checkout** (`docs/shared/` or
  `docs/doc/shared/`). Commit in this clone and push from here.

## What belongs here

These pages describe **the dataset**: how the study was designed and run,
how the data were acquired, converted and preprocessed, and how they are
organised. They are public, and they are meant to stay correct as the
dataset changes. So:

- **No status, no plans.** Nothing here says what is done, in progress,
  blocked or planned, when something ran, or what was deleted. That goes
  stale silently and is the lab's working record, not the dataset's
  description; it lives in the lab's private repo. Describe a pipeline or
  derivative here once its output exists, in the present tense.
- **No results.** Findings from analyses of the data are not published here.
- **No coverage claims.** Which subjects and sessions exist, and what each
  pipeline has covered, is answered by the catalog
  (`inventory/catalog.duckdb`), not by prose. Durable structural facts (how
  many movie stimuli exist, what the task labels are) are fine.
- **Data described generically.** Example paths use `sub-##`, `ses-##`,
  `YYYY-MM-DD` rather than real filenames or acquisition times.
- **Pages for people running the code** (QC review, the MCP server, API
  docs, command-line tools) live in [mmmdata](https://github.com/hulacon/mmmdata)'s
  own `docs/`, under the site's *Code & Tools* section.

## Front matter

Each file includes Jekyll YAML front matter (title, parent, nav_order, and
optionally grand_parent and has_children) for rendering on the mmmdata site.
This is harmless when reading the files as plain markdown.
