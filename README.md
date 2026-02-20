# MMMData Shared Documentation

Canonical dataset documentation for the Multi-Modal Memory Dataset (MMMData) project.

This repo is consumed as a **git submodule** by:
- [hulacon/mmmdata](https://github.com/hulacon/mmmdata) — rendered on the Jekyll site at `docs/doc/shared/`
- [jhutchin/mmmdata-agents](https://github.com/jhutchin/mmmdata-agents) — read as plain markdown at `docs/shared/`

## Files

| File | Description |
|------|-------------|
| `study-overview.md` | Study design, participants, scanner |
| `session-schedule.md` | 30-session canonical schedule and deviations |
| `tasks.md` | Task taxonomy with BIDS labels and detailed descriptions |
| `file-organization.md` | BIDS directory tree, naming conventions, expected files |
| `sourcedata.md` | Sourcedata structure and migration notes |
| `derivatives.md` | Derivatives pipelines and target workflow |
| `stimuli.md` | Stimulus materials (images, audio, movies) and metadata |
| `compliance-status.md` | BIDS compliance checklists and open questions |

## Contributing

Edit files here, commit, and push. Then update the submodule pointer in each consumer repo:

```bash
# In mmmdata
cd docs/doc/shared && git pull origin main && cd ../../..
git add docs/doc/shared && git commit -m "Update shared docs"

# In mmmdata-agents
cd docs/shared && git pull origin main && cd ../..
git add docs/shared && git commit -m "Update shared docs"
```

## Front matter

Each file includes Jekyll YAML front matter (title, parent, nav_order) for rendering on the mmmdata site. This is harmless when reading the files as plain markdown.
