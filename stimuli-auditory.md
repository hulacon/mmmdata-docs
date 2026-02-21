---
title: Auditory Stimuli (Word Pool)
parent: Stimuli
grand_parent: Dataset Description
nav_order: 2
---

# Auditory Stimuli: Toronto Word Pool (twp1000)

- **Source**: Toronto Word Pool, a standard psycholinguistic word set used
  across many memory experiments
- **Count**: 1,000 words × 4 speakers = 4,000 audio files
- **Format**: MP3
- **Naming**: `{word}_{voice}.mp3` (e.g., `cabin_shimmer.mp3`)
- **Generation**: Text-to-speech via OpenAI API (GPT-4 Turbo model),
  generated November 2023

## Speakers

| Voice ID | Gender | OpenAI TTS voice |
|----------|--------|------------------|
| `echo`   | Male   | Echo             |
| `nova`   | Female | Nova             |
| `onyx`   | Male   | Onyx             |
| `shimmer`| Female | Shimmer          |

## Word Metadata

Two CSV files provide psycholinguistic properties for each word:

- **`twp1000.csv`** — the 1,000-word subset used in this study
- **`twp_all.csv`** — the full Toronto Word Pool

| Column | Description |
|--------|-------------|
| `itmno` | Item number (1-based index) |
| `word` | The word itself |
| `imagery` | Imagery rating (higher = more imageable) |
| `concreteness` | Concreteness rating (higher = more concrete) |
| `letters` | Number of letters |
| `frequency` | Word frequency (Kucera-Francis norms) |
| `foa` | First-order approximation to English, based on individual letter frequencies |
| `soa` | Second-order approximation to English, based on bigram frequencies |
| `onr` | Orthographic neighbor ratio (Landauer & Streeter, 1973): word frequency relative to combined frequency of orthographic neighbors |
| `dictcode` | Grammatical class code: 1=noun, 2=verb, 3=adjective, 4=adverb, 5=other |
| `noun` | Percent noun usage (0 or 100 for unambiguous words; rated for ambiguous words) |
| `canadian` | Flag for Canadian spelling variant (the final 13 entries in the full TWP are Canadian alternatives with duplicate item numbers) |
| `tmpno` | Temporary item number — present in `twp1000.csv` only |

The Toronto Word Pool (1,080 words) originated from Thorndike-Lorge (1944) norms
and has been used in hundreds of memory experiments. See the
[WordPools R package](https://cran.r-project.org/package=WordPools) for the
original dataset and documentation.
