---
title: Data Organization
parent: Dataset Description
has_children: true
nav_order: 5
---

# Data Organization

MMMData follows the [Brain Imaging Data Structure (BIDS) v1.9.0](https://bids-specification.readthedocs.io/)
standard. Data are organized into three tiers:

- **BIDS raw**: Converted NIfTI + JSON + events TSV files
- **Source data**: Original DICOMs, PsychoPy output, audio recordings
- **Derivatives**: Preprocessed outputs (fMRIPrep, MRIQC)
