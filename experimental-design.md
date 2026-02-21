---
title: Experimental Design
parent: Dataset Description
has_children: true
nav_order: 2
---

# Experimental Design

Tasks fall into four categories, organized across a 30-session schedule. Task
labels in BIDS filenames are being updated to reflect the TB/NAT distinction
(e.g., `task-encoding` will become `task-TBencoding` or `task-NATencoding`).

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
