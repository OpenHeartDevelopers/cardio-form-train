# cardio-form-train

Tools to package, validate and evaluate model weights for
[cardio-form](https://github.com/OpenHeartDevelopers/cardio-form).

> **Status: early.** The repository holds the project skeleton only. No
> command is available yet.

## Purpose

cardio-form downloads its model weights from GitHub releases. This repository
helps a lab member turn a trained nnUNet model into a weights bundle that
cardio-form accepts, and measure the bundle before anyone publishes it.

Planned commands:

| Command | Does | Status |
|---|---|---|
| `package` | Builds release zips and `weights-manifest.json` from an nnUNet results folder. Picks the fold with the best held-out Dice, re-saves the checkpoint for numpy 1.x, removes dataset names, and validates the zip against cardio-form's archive contract. | planned |
| `evaluate` | Segments a benchmark set with one or more model versions and writes a Dice table per label. | planned |

This repository does **not** upload weights. A maintainer of cardio-form
uploads a validated bundle and registers it (see "Publish weights" below).

Scope today: the nnUNet 2D segmentation models (`segment_sax`,
`segment_lax_2ch`, `segment_lax_4ch`). The 3D reconstruction networks are out
of scope.

## Install

You need two environments:

| Environment | Use | numpy |
|---|---|---|
| **publish** | `package`, `evaluate` | 1.24.4, the same as cardio-form users |
| **train** | nnUNet training on the GPU server | the server's own (2.x) |

The `package` command must run in the publish environment. It loads
checkpoints saved under numpy 2.x and saves them again under numpy 1.x, so
cardio-form users can load them.

### Publish environment

1. Create a conda environment named `cardioform-train` from cardio-form's
   `environment-cpu.yaml` (CPU is enough).
2. Install this package. pip also installs cardio-form at the tag pinned in
   `pyproject.toml` (`v0.4.3`):

```bash
conda activate cardioform-train
pip install -e .
```

This does not change the conda packages: cardio-form declares only
`pycemrg`, and `pycemrg` needs only `click`, `pyyaml` and `tqdm`.

### Train environment

Use the nnUNet environment that exists on the GPU server. The training
procedure is not documented yet.

## Configuration

Server paths go in `config/server.yaml`. Git ignores this file. A template
with placeholders will be at `config/server.example.yaml`.

## Rules for this public repository

- Do not commit cohort names, dataset IDs that identify a cohort, or server
  paths. Put them in `config/server.yaml`.
- Do not commit weights (`*.pth`, `*.zip`) or images (`*.nii`, `*.nii.gz`).
  Git ignores them.
- Use `--audit-string` with `package` (when available) to stop a bundle that
  still holds a cohort name.

## Publish weights (maintainers)

1. Get a bundle that passes `package`, plus its `evaluate` table.
2. Create a draft release with a tag that ends in `-models`, for example
   `v0.3.0-models`, and upload the zips and `weights-manifest.json`.
3. Run the registration workflow:

   ```bash
   gh workflow run register-weights.yml --repo OpenHeartDevelopers/cardio-form -f tag=v0.3.0-models
   ```

4. Review the pull request that the workflow opens, merge it, then publish the
   release.

## Layout

- `src/cardio_form_train/` — stateless library code.
- `config/` — the server configuration (git-ignored) and its template.
- `outputs/` — bundles, results and logs (git-ignored).
