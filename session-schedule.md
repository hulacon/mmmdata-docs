---
title: Session Schedule
parent: Experimental Design
grand_parent: Dataset Description
nav_order: 1
---

# Session Schedule

Each subject follows a canonical session schedule. Minor deviations are expected
(e.g., a localizer task collected in a different session than planned).

| Session(s) | Phase | Purpose |
|------------|-------|---------|
| **ses-01** | Baseline | Anatomical (T1w, T2w), diffusion (DWI), resting-state fMRI |
| **ses-02, ses-03** | Localizers | Functional localizer tasks (PRF, fLoc, auditory, tonotopy; motor too under the regularised protocol — see [Localizer Tasks](tasks-localizer.md)). No anatomy. |
| **ses-04 to ses-18** | Trial-Based (TB) | Repeated cued-recall memory paradigm: encoding, retrieval, math, resting (ses-18 has no encoding run) |
| **ses-19 to ses-28** | Naturalistic (NAT) | Repeated naturalistic memory paradigm: encoding, retrieval, math, resting |
| **ses-28** | Anatomy repeat | Also includes a second anatomical + DWI acquisition |
| **ses-29** | Behavioral only | Out-of-scanner behavioral session (no imaging) |
| **ses-30** | Final memory | Final memory tests (`FIN*` task labels, see [Final Sessions](tasks-final-session.md)), plus makeup localizers if needed |

## Known Deviations from Canonical Schedule

- Localizer tasks (ses-02/ses-03) are sometimes swapped across subjects (e.g.,
  auditory in ses-02 for one subject but ses-03 for another).
- Session 30 may include localizer tasks that were missed earlier.
- Localizer runs are split differently across sessions per subject (e.g.,
  all floc runs in one session for one subject, split over ses-02 and ses-03
  for another), though the total run count per localizer task is the same.
- Future subjects may have similar minor deviations.
