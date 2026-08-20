---
title: Experimental Design
parent: Dataset Description
has_children: true
nav_order: 2
---

# Experimental Design

Tasks fall into four categories, organized across a 30-session schedule. BIDS
task labels carry a phase prefix — `INIT*` (baseline), `TB*` (trial-based),
`NAT*` (naturalistic), `FIN*` (final session) — with the localizer tasks
(`prf`, `floc`, `tone`, `auditory`, `motor`, `fixation`) unprefixed. This
relabeling is complete: no unprefixed `task-encoding` / `task-retrieval` /
`task-math` / `task-resting` files exist.

| Phase | Sessions | Paradigm |
|-------|----------|----------|
| Baseline | ses-01 | Anatomical, diffusion, resting-state |
| Localizers | ses-02, ses-03 | Functional localizer tasks |
| Trial-Based (TB) | ses-04 to ses-18 | Cued-recall memory paradigm |
| Naturalistic (NAT) | ses-19 to ses-28 | Movie-watching and free recall |
| Behavioral only | ses-29 | Out-of-scanner behavioral session |
| Final memory | ses-30 | Final memory tests + makeup localizers |

## Shared Tasks

Two tasks appear identically across both TB and NAT phases:

- **Resting state**: Fixation on a central point for 5 minutes.
- **Math distractor**: Series of math problems where all answers equal 1, 2, or 3.
