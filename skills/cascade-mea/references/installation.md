# Installing CASCADE — step by step for non-programmers

Use this when someone has never installed CASCADE, or `cascade` is "command not found".
Go one step at a time: give one command, wait for its output, check it, then the next.
Never assume the person knows what a terminal, environment or path is — explain in one line.

If `cascade --version` already works, CASCADE is installed: skip to step 5.

---

## 0. What gets installed and why

| Piece | What it is | Why |
|---|---|---|
| **Conda** (via Miniforge) | A program that installs Python and scientific libraries into a separate "environment" | Keeps CASCADE's Python separate from anything else on the computer |
| **Environment `cascade-mea`** | A self-contained Python 3.10 with numpy, scipy, pandas, matplotlib, h5py | CASCADE needs exactly these |
| **CASCADE** | The three commands `cascade`, `cascade-info`, `cascade-ltpltd` | The analysis itself |

Disk space: about 1–2 GB. Internet is needed only during installation.

---

## 1. Open a terminal

| System | How |
|---|---|
| **Windows** | After step 2, open **"Miniforge Prompt"** from the Start menu (not the normal Command Prompt) |
| **macOS** | Applications → Utilities → **Terminal** |
| **Linux / WSL** | Open your terminal |

A terminal is a window where you type commands and press Enter.

---

## 2. Install Miniforge (conda) — once per computer

Download the installer for your system from https://github.com/conda-forge/miniforge (section "Download").

- **Windows:** run the `.exe`, accept the defaults, then open **Miniforge Prompt**.
- **macOS / Linux:** in a terminal:
  ```bash
  bash Miniforge3-$(uname)-$(uname -m).sh
  ```
  Answer `yes` to the questions, then close and reopen the terminal.

Check it worked:
```bash
conda --version
```
Expect something like `conda 24.x`. If "command not found": the terminal was not reopened, or (Windows) the normal Command Prompt was used instead of Miniforge Prompt.

Already have Anaconda, Miniconda or micromamba? Use it — skip this step (`micromamba` users: replace `conda` with `micromamba` below).

---

## 3. Get the CASCADE code

**Option A — no Git needed (easiest):**
1. Open https://github.com/FA387/cascade-mea
2. Green **Code** button → **Download ZIP**
3. Unzip it somewhere simple, e.g. `C:\CASCADE\cascade-mea` or `~/cascade-mea`

**Option B — with Git** (https://git-scm.com/downloads), which makes updating easier:
```bash
git clone https://github.com/FA387/cascade-mea.git
```

Then move into the folder (adjust the path):
```bash
cd cascade-mea
```
Check you are in the right place: `ls` (macOS/Linux) or `dir` (Windows) should list `pyproject.toml`, `environment.yml`, `README.md` and a `cascade` folder.

---

## 4. Create the environment and install CASCADE

```bash
conda env create -f environment.yml
conda activate cascade-mea
pip install .
```

- The first command takes a few minutes. It creates the `cascade-mea` environment.
- `conda activate cascade-mea` must be run **every time you open a new terminal** before using CASCADE. The prompt then starts with `(cascade-mea)`.
- `pip install .` (note the dot = "this folder") installs CASCADE from the downloaded code.

Optional, for the reference implementation of the Kreuz SPIKE-distance (only matters with `--sync_metrics kreuz` or `all`):
```bash
pip install ".[synchrony]"
```
If that fails on Windows with a compiler error, skip it — CASCADE falls back to its own implementation.

---

## 5. Check it works

```bash
cascade --version
cascade --test
```
- `--version` should print `CASCADE v0.9.6` (or newer). **If it prints `v0.8.0`, the old release was installed** — see "Troubleshooting" below.
- `--test` runs 16 self-tests on synthetic data (no recording needed; 14 before v0.9.6) and must end with `All 16 tests passed. CASCADE is installed correctly.`

Then inspect a real recording (see SKILL.md → Running Analysis).

---

## Install from PyPI or conda (shortcut)

```bash
conda create -n cascade-mea python=3.10
conda activate cascade-mea
pip install cascade-mea
cascade --version
```
Or with conda in one step:
```bash
conda create -n cascade-mea -c fa387 -c conda-forge cascade-mea
conda activate cascade-mea
```
PyPI: https://pypi.org/project/cascade-mea/ — conda: https://anaconda.org/fa387/cascade-mea . Both have **0.9.6** since 2026-11-20, tested with `cascade --test` in a fresh environment. If `cascade --version` says 0.8.0, update: `pip install -U cascade-mea` (or `conda update -c fa387 cascade-mea`).

---

## Updating to a newer version

- Downloaded ZIP: download the new ZIP, then in its folder: `conda activate cascade-mea` → `pip install .`
- Git: in the folder: `git pull` → `conda activate cascade-mea` → `pip install .`

Check with `cascade --version`.

Uninstall completely: `conda env remove -n cascade-mea` and delete the folder.

---

## Troubleshooting installation

| Symptom | Cause / fix |
|---|---|
| `cascade: command not found` / `'cascade' is not recognized` | Environment not active → `conda activate cascade-mea`. Or `pip install .` was never run. |
| `conda: command not found` | Miniforge not installed, terminal not reopened, or (Windows) not using Miniforge Prompt |
| `EnvironmentFileNotFound` / `environment.yml not found` | Not inside the `cascade-mea` folder → `cd` into it (step 3 check) |
| `CondaValueError: prefix already exists` | Environment exists already → `conda activate cascade-mea` and continue with `pip install .`; or recreate: `conda env remove -n cascade-mea` first |
| `cascade --version` says 0.8.0 | Old release → `pip install -U cascade-mea` (or `conda update -c fa387 cascade-mea`, or `pip install .` in the GitHub folder) |
| `Could not read … as --manufacturer X` | Not an installation problem: wrong device chosen for the file — use the suggested `--manufacturer` |
| `ModuleNotFoundError: No module named 'h5py'` (or numpy, …) | Wrong environment active, or pip used outside it → `conda activate cascade-mea`, then `pip install .` |
| A self-test FAILS | Copy the full output; reinstall (`conda env remove -n cascade-mea`, then step 4). If it still fails, open an issue: https://github.com/FA387/cascade-mea/issues |
| Path with spaces fails (e.g. `My Data`) | Put the path in quotes: `--input "D:\My Data\rec.bxr"` |
| Windows path in WSL | `D:\data\rec.bxr` becomes `/mnt/d/data/rec.bxr` |

Tested platforms: developed and tested on Linux (Ubuntu, incl. WSL on Windows). CASCADE is pure Python, so native Windows and macOS are expected to work with the steps above.

## Docker

There is no official Docker image: pip and conda already install CASCADE in one command. If you need one (e.g. for a computing cluster), open an issue on GitHub. If CASCADE later goes on Bioconda, BioContainers builds an image automatically (`quay.io/biocontainers/cascade-mea`).
