---
title: Trial-Based Memory Tasks
parent: Experimental Design
grand_parent: Dataset Description
nav_order: 3
---

# Trial-Based Memory Tasks (ses-04 to ses-18)

Paired-associate memory paradigm. During encoding, a single word is played
through headphones (from the Toronto Word Pool) while an image is presented on
screen (from NSD shared1000). Image–word pairings are randomly assigned per
subject. During cued recall, one pairmate is presented and the participant must
recall the other.

| Current label | Planned label | Description |
|---------------|---------------|-------------|
| `task-encoding` | `task-TBencoding` | Paired image + word encoding (2-3 runs per session) |
| `task-retrieval` | `task-TBretrieval` | Cued recall with one pairmate as cue (1-4 runs per session) |
| `task-math` | `task-TBmath` | Math distractor task |
| `task-resting` | `task-TBresting` | Resting state between task blocks |

## Encoding
- Task: "How well are you able to associate the word with the image?" Select 1 if it was not at all, 2 for somewhat, and 3 for very well.
- Conditions: one pair presented once ("single"), one pair presented three times ("repeats"), sequence of pairs (x3) presented three times ("triplets"), and pairs presented three times every session ("super repeats")
- The image was shown on the screen for 3s, followed by a fixation dot for 1.5s. A total of 183 image-word pairs were presented in each session.

## Cued Recall
- Task: "Please retrieve the image/word associated with the word/image cue as vividly as possible, and rate the vividness of your memory." Select 1 for least vivid, 2 for somewhat vivid, and 3 for most vivid.
- Conditions: within session for if the participant saw that pairmate during that fMRI encoding session and across session for if the participant saw that pairmate in the previous fMRI session.
- Two of the four runs, the participant was given the word and had to retrieve the image, and vice versa for the other two runs.

## 2-AFC
- Shown two images on the screen and heard a word. "Which image was paired with the word?" 1 for high left image confidence, 2 for low left image confidence, 3 for right image low confidence, and 4 for right image high confidence.
- Conditions: image-word pairs also fell within the within session and across session conditions.

## Fixation Calibration Task
- Participant followed a fixation cross with their eyes for 60 seconds.
