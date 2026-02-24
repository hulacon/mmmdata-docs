# Computational Feature Extraction (viz2psy)

For stimulus descriptions (images, audio, movies, metadata), see
[shared/stimuli.md](shared/stimuli.md).

---

Stimulus images and movie frames are scored with
[viz2psy](https://github.com/hulacon/viz2psy), which runs 11 computational
models covering memorability, emotion, semantics, scene categories, object
detection, and low-level statistics.

## Models

| Model | Output | What it measures |
|-------|--------|------------------|
| `resmem` | 1 score | Image memorability (0–1) |
| `emonet` | 20 scores | Emotion category probabilities |
| `clip` | 512 dims | Vision-language embeddings (CLIP ViT-B/32) |
| `caption` | 1 string | Natural language caption (BLIP) |
| `dinov2` | 768 dims | Self-supervised visual features (DINOv2 ViT-B/14) |
| `gist` | 512 dims | Spatial envelope descriptor (Gabor filters) |
| `places` | 467 scores | Scene categories (365) + SUN attributes (102) |
| `llstat` | 17 scores | Low-level stats (luminance, contrast, color, edges, frequency) |
| `saliency` | 576 dims | Fixation density prediction (24x24 grid, DeepGaze IIE) |
| `aesthetics` | 1 score | Aesthetic quality (1–10, LAION model) |
| `yolo` | 85 scores | Object detection counts (80 COCO classes + 5 summary stats) |

Running all models produces ~2,900 feature columns per image/frame.

## Output Files

```
stimuli/
├── shared1000/
│   ├── viz2psy_scores.csv              # All-models output (1000 rows × ~2900 cols)
│   └── viz2psy_scores.csv.meta.json    # Feature definitions and provenance
├── movies/
│   ├── viz2psy_cue_scores.csv          # Cue image features (60 rows)
│   ├── viz2psy_cue_scores.csv.meta.json
│   └── viz2psy_scores/                 # Per-movie frame-level features
│       ├── Adventure_Time_scores.csv
│       ├── Adventure_Time_scores.csv.meta.json
│       ├── Adventure_Time_frames/      # Extracted video frames (PNGs)
│       │   ├── frame_0.000.png
│       │   ├── frame_0.500.png
│       │   └── ...
│       ├── ...                         # (one set per movie)
│       └── movie_summary.csv           # Per-movie aggregates (mean, std, min, max)
```

## Running the Extraction

Extraction scripts live in the **mmmdata** repo (`code/mmmdata/scripts/`), not
mmmdata-agents, since they are data-processing infrastructure rather than agent
tooling.

```bash
# From the mmmdata repo (code/mmmdata/)
python scripts/run_viz2psy.py images --dry-run
python scripts/run_viz2psy.py movies --dry-run
python scripts/run_viz2psy.py cues --dry-run

# Run all models on images (requires GPU)
python scripts/run_viz2psy.py images

# Run on movies with custom frame interval
python scripts/run_viz2psy.py movies --frame-interval 1.0

# Run specific models only
python scripts/run_viz2psy.py images --models resmem emonet aesthetics
```

## Metadata Sidecar

Each output CSV is accompanied by a `.meta.json` sidecar documenting:
- viz2psy version and runtime
- Input source (image paths, video path, frame interval)
- Output dimensions (rows, columns)
- Per-model feature definitions (column names, embedding patterns, counts)

## Agent Interface

The `stimuli` agent provides tools for querying these features:

```bash
python -m src.cli --agent stimuli "What stimulus sets are available?"
python -m src.cli --agent stimuli "Show the most memorable images"
python -m src.cli --agent stimuli "What emotions are strongest in From-Dad-To-Son?"
```
