---
title: Tasks
parent: Dataset Description
nav_order: 3
---

# Tasks

Tasks fall into four categories. Task labels in BIDS filenames are being updated
to reflect the TB/NAT distinction (e.g., `task-encoding` will become
`task-TBencoding` or `task-NATencoding`).

## Functional Localizer Tasks (ses-02, ses-03, sometimes ses-30)

| BIDS task label | Description |
|-----------------|-------------|
| `task-prf` | Population receptive field mapping (typically 3 runs) |
| `task-floc` | Functional localizer (3-6 runs) |
| `task-auditory` | Auditory stimulation |
| `task-tone` | Tonotopy (1-2 runs) |
| `task-motor` | Motor task (2 runs, ses-30) |
| `task-fixation` | Fixation baseline (ses-30) |

These labels are expected to remain unchanged.

### PRF
- 3 runs for each session
- Order to run (93=multibar, 94=wedgeringmash):
  - Run 1: 93, 94, 93
  - Run 2: 94, 93, 94
- Code adapted from http://kendrickkay.net/analyzePRF/
- Two run types were used: multibar and wedgering. The first, termed 'multibar', involves bars sweeping in multiple directions (same as RETBAR in the Human Connectome Project 7T Retinotopy Dataset). The second, termed 'wedgering', involves a combination of rotating wedges and expanding and contracting rings.
- A total of 6 runs (3 multibar and 3 wedgering) were collected across two functional localizer sessions. Each run lasted 300 s.
- The acquisition structure was [MWM] for the first localizer session, and [WMW] for the second localizer session. M indicates a multibar run and W indicates a wedgering run.
- Stimuli filled a circular region with diameter 10.6°. Throughout the stimulus presentation, a small semi-transparent dot (0.2° × 0.2°) was present at the center of the stimuli.
- Participants were instructed to maintain fixation on a central dot (switched randomly to one of three colors: black, white, or red, every 1–5 s) and press a button whenever the color of the dot changes.

### fLoc
- 3 runs for each session

### Motor Localizer
- 1 run for the 2nd localizer session and during the final cued recall session

### Auditory Category Localizer
- 1 run for the 1st localizer session and during the final cued recall session

### Tonotopy Mapping
- 1 run for each session
- Order to run (low to high or high to low):
  - Run 1: low to high
  - Run 2: high to low

## Trial-Based Memory Tasks (ses-04 to ses-18)

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

### Encoding
- Task: "How well are you able to associate the word with the image?" Select 1 if it was not at all, 2 for somewhat, and 3 for very well.
- Conditions: one pair presented once ("single"), one pair presented three times ("repeats"), sequence of pairs (x3) presented three times ("triplets"), and pairs presented three times every session ("super repeats")
- The image was shown on the screen for 3s, followed by a fixation dot for 1.5s. A total of 183 image-word pairs were presented in each session.

### Cued Recall
- Task: "Please retrieve the image/word associated with the word/image cue as vividly as possible, and rate the vividness of your memory." Select 1 for least vivid, 2 for somewhat vivid, and 3 for most vivid.
- Conditions: within session for if the participant saw that pairmate during that fMRI encoding session and across session for if the participant saw that pairmate in the previous fMRI session.
- Two of the four runs, the participant was given the word and had to retrieve the image, and vice versa for the other two runs.

### 2-AFC
- Shown two images on the screen and heard a word. "Which image was paired with the word?" 1 for high left image confidence, 2 for low left image confidence, 3 for right image low confidence, and 4 for right image high confidence.
- Conditions: image-word pairs also fell within the within session and across session conditions.

### Fixation Calibration Task
- Participant followed a fixation cross with their eyes for 60 seconds.

### Final Cued Recall Task
- Participant had 4 runs of a final cued recall session, the same as in the trial-based task, in which the participant was given either the word or image cue, and were instructed to recall the image or word that they saw during the 15 previous cued recall scans.

### Final Temporal Memory Task
- Participants were given a continuous scale and asked to select when they initially saw that pair during the encoding condition.

### Final 2AFC
- Across all 15 previous trial-based tasks.

## Naturalistic Memory Tasks (ses-19 to ses-28)

Movie-watching and free recall paradigm. During encoding, participants watch
movie clips. During retrieval, participants verbally recall what they remember
(audio recorded via microphone).

| Current label | Planned label | Description |
|---------------|---------------|-------------|
| `task-encoding` | `task-NATencoding` | Movie viewing |
| `task-retrieval` | `task-NATretrieval` | Verbal free recall (audio recorded) |
| `task-math` | `task-NATmath` | Math distractor task |
| `task-resting` | `task-NATresting` | Resting state between task blocks |

### Movie Viewing
- Watched 8 films of 2-6 minutes each
- Types of movies: various length and style (live-action, animated, stop motion, with or without speech)
- Conditions: 6 of the movies are new and 2 are repeated every session
- Participants are first shown the movie title for 2.5 seconds and then shown the movie.

### Free Recall
- Participants are prompted with a movie title for 7.5 seconds. During this time, the participant indicates if they are able to recall the movie associated with the title.
- If they are unable to recall the movie, they are then shown an image cue for 3 seconds.
- A fixation cross is then shown, and the participant speaks freely into a microphone for an unlimited amount of time to recall as many details as possible from the movie.
- Conditions: 4 are from that current session previously shown in the encoding task, 2 are repeated every session, and 2 are from the previous session.

## Resting State Task
- Focused on a fixation point for 5 minutes.

## Math Task
- Presented with a series of math problems where all the answers equaled 1, 2, or 3.

## Final Memory Session (ses-30)

Session 30 contains final memory tests that will receive unique task IDs (to be
determined). It may also include makeup localizer runs.

### Final Free Recall
- Outside the scanner behavioral task ~2 weeks after the last naturalistic task.
- Participants were asked to recall as many details as possible about the entire experiment.
