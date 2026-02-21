# MMMData Shared Documentation

> **These docs are best viewed at [hulacon.github.io/mmmdata](https://hulacon.github.io/mmmdata/).**
> This repo contains the source markdown files; the rendered, navigable version lives at the link above.

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
│   └── stimuli-movies.md          # Short films, recall conditions, schedule
│
├── Data Acquisition/
│   ├── data-acquisition.md        # Modality summary table
│   ├── mri-sequences.md           # MRI acquisition parameters
│   └── behav-physio.md            # Behavioral, eye-tracking, physiological
│
├── Data Organization/
│   ├── data-organization.md       # BIDS overview
│   ├── file-organization.md       # BIDS directory tree & naming conventions
│   ├── sourcedata.md              # Source data structure & migration
│   └── derivatives.md             # Preprocessing pipelines & derivatives
│
├── compliance-status.md           # BIDS compliance checklists
├── access.md                      # Data access & availability
└── contributors.md                # Project team & affiliations
```

## Contributing

Edit files here, commit, and push. Then update the submodule pointer in each consumer repo:

```bash
# In mmmdata
cd docs/doc/shared && git pull origin main && cd ../../..
git add docs/doc/shared && git commit -m "Update shared docs"

# In mmmdata-agents
cd docs/shared && git pull origin main && cd ../..
git add docs/shared && git commit -m "Update shared docs"
```

## Front matter

Each file includes Jekyll YAML front matter (title, parent, nav_order, and
optionally grand_parent and has_children) for rendering on the mmmdata site.
This is harmless when reading the files as plain markdown.
