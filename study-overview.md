---
title: Study Overview
parent: Dataset Description
nav_order: 1
---

# Study Overview

The Multi-Modal Memory Dataset (MMMData) is a dense (many observations for a
smaller number of participants) fMRI study being conducted at the University of
Oregon. The goal is to better understand how the brain remembers sights and
sounds and how doing so in traditional, experimental settings (unrelated,
discrete series of trials) relates to doing so in more naturalistic settings
(i.e., watching and recalling a video).

- **Design**: Dense within-subject (many sessions per participant, few participants)
- **Target sample**: six to seven usable subjects; sub-01 and sub-02 are pilots
- **Sessions per subject**: Up to 30 scanning sessions
- **Scanner**: Siemens MAGNETOM Prisma 3T, University of Oregon
- **BIDS version**: 1.9.0

## Participants

| Subject | Age | Sex | Handedness | Vision |
|---------|-----|-----|------------|--------|
| sub-01, sub-02 | — | — | — | — |
| sub-03 | 20 | M | right | normal |
| sub-04 | 28 | M | right | normal |
| sub-05 | 22 | F | right | corrected |
| sub-06, sub-07, sub-08, sub-09 | — | — | — | — |

sub-01 and sub-02 are pilots (DICOMs in `mmmsourcedata/archive/pilot/` only)
and are not BIDSified; all other enrolled subjects are BIDSified as their data
arrive. Query the catalog for coverage.

Full demographics (including education, race, sleep, medication history) are
recorded in `participants.tsv`. Questionnaire data (VVIQ, final debriefing)
are in `phenotype/`.

> **Session coverage.** This page does not say which sessions or derivatives
> are complete for any subject. For any coverage question — which subjects
> and sessions exist, what a pipeline has processed —
> query the catalog (`inventory/catalog.duckdb`); see
> [Data Organization](data-organization.md). ses-29 (out-of-scanner
> behavioral) is not yet BIDS for any subject.

*Participant table verified against `participants.tsv` and the filesystem on
2026-08-20; the per-subject `Status` column was removed 2026-09-05 from the
project's settled records (coverage is a catalog question).*
