---
title: Final Sessions
parent: Experimental Design
grand_parent: Dataset Description
nav_order: 5
---

# Final Sessions (ses-29 and ses-30)

## Final Free Recall (ses-29)

BIDS task label: `task-FINrecall` (`beh/` only; no imaging)

- Outside the scanner behavioral task ~2 weeks after the last naturalistic task.
- Participants were asked to recall as many details as possible about the entire experiment.
- Audio recordings (M4A) stored in `mmmsourcedata/sub-##/ses-29/audio/`, outside the BIDS tree.

## Final Memory Tests (ses-30)

Session 30 contains final memory tests and makeup localizer runs as needed.
A T1w anatomical is also acquired this session (the others are in ses-01, two runs, and ses-28). The ses-30 task labels
(`FINretrieval`, `FINresting`, `FIN2AFC`, `FINtimeline`) are in use; see
[File Organization](file-organization.md).

### Final Cued Recall (`task-FINretrieval`)
- Participant had 4 runs of a final cued recall session, the same as in the trial-based task, in which the participant was given either the word or image cue, and were instructed to recall the image or word that they saw during the 15 previous trial-based sessions.

### Final Temporal Memory (`task-FINtimeline`)
- Participants saw an image and marked with the mouse, on a continuous timeline spanning the trial-based sessions (first to last), when they first saw it.

### Final 2-AFC (`task-FIN2AFC`)
- Across all 15 previous trial-based tasks.
