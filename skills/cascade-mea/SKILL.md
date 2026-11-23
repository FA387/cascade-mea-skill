---
name: cascade-mea
description: Run CASCADE MEA (multi-electrode array) neurophysiology analysis on spike recordings. Use this skill whenever the user mentions MEA recordings, spike data, .bxr files, .h5 recording files, neuronal avalanches, criticality metrics, DCC, branching ratio, shape collapse, spike synchrony, LTP/LTD classification, burst detection, firing rate analysis, or functional connectivity from electrophysiology data — even if they don't say "CASCADE" explicitly. Also trigger when the user wants to inspect, interpret, or understand existing CASCADE output files such as criticality reports, TSV data files, or raster plots. This skill should be used proactively whenever MEA analysis of any kind is requested.
---

# CASCADE MEA Analysis

CASCADE (Criticality Avalanche Spike Cross-platform Analysis Detection Engine) analyses spike data from multi-electrode arrays (MEA) of six manufacturers: 3Brain, Maxwell Biosystems, Cortical Labs, Multi Channel Systems, Axion BioSystems and FinalSpark.

**Version:** this skill describes v0.9.6 (check with `cascade --version`). Published as Avila et al. 2026, *Bioinformatics* 42(8):btag570 — the article describes v0.8.0.
**Three tools:** `cascade-info` (inspect a recording), `cascade` (full pipeline), `cascade-ltpltd` (LTP/LTD)
**Not installed?** (`cascade: command not found`) → `references/installation.md`. If CASCADE is in a conda environment, activate it first (`conda activate cascade-mea`) or prefix each command with `conda run -n cascade-mea`.
**Help:** `--help` on any tool (there is no `-h`). Flags are grouped by step: 1 Setup, 2 Preprocessing, 3 Analysis, 4 Output, 5 Misc.
**Source code and README:** https://github.com/FA387/cascade-mea

**Large recordings and memory.** Synchrony needs about 1.5 GB of RAM per recorded hour. For runs over about 2 h, use `--sync_metrics none`, or analyse a window with `--start/--dur`. Run heavy jobs one at a time. On Linux, a hard memory cap stops a runaway job instead of freezing the computer:
```bash
systemd-run --user --scope -q -p MemoryMax=4G -p MemorySwapMax=0 cascade ...
```

---

## Reference files — read the one that fits, when it fits

| File | Read it when |
|---|---|
| `references/installation.md` | CASCADE is not installed, "command not found", new computer, updating, PyPI vs GitHub, Docker |
| `references/file-formats.md` | Which file/`--manufacturer` to use, what each device's file contains, export questions, "No wells/streams found", adding a new device |
| `references/noisy-recordings.md` | Dirty/noisy recording, artefacts, choosing `--ae`/`--max_rate`/`--max_rpv`/`--max_amplitude`, implausible results |
| `references/flags.md` | Any flag's meaning or default (all three tools, grouped by step) |
| `references/outputs.md` | What a specific output file or figure contains |
| `references/links.md` | GitHub, PyPI, paper, Zenodo sample data, manufacturer documentation, conda/Python downloads, method references to cite |

---

## Who am I helping?

Many CASCADE users are biologists, not programmers. Adapt:

- **New to the command line** (asks "how do I install", "what is a terminal", pastes Windows paths, gets "command not found"): go step by step — one command, wait for the output, check it, then the next. Explain each term in one line. Prefer copy-paste commands with their real paths filled in. If you can run terminal commands, run them yourself instead of asking them to.
- **Experienced user**: give the full command and the reasoning briefly.
- Either way, never just dump numbers — say what they mean and how confident to be (number of avalanches, branching-ratio checks, filters used).

## Read the situation first

Before doing anything, figure out which situation the user is in — you can usually tell from context without asking:

- **Not installed yet / new computer** → `references/installation.md`
- **Have a recording file, want analysis** → [Running Analysis]
- **Recording is noisy or results look wrong** → `references/noisy-recordings.md`
- **Already have CASCADE output files, want to understand them** → [Interpreting Results]
- **Have two CASCADE runs, want LTP/LTD** → [LTP/LTD Classification]
- **Need numbers that match the published article (v0.8.0)** → [Reproducing published numbers]
- **Writing a paper/thesis** → [Writing the methods section]
- **Something went wrong** → [Troubleshooting]

---

## Running Analysis

### Gather what you need

You need three things before running. Infer from context when possible; only ask what's missing:

1. **File path** — accept Windows paths; if commands run in WSL, convert `D:\...` to `/mnt/d/...`
2. **Manufacturer** — infer from extension when unambiguous (`.bxr` → `3brain`, `.csv` → `axion`, `fsNNN_package.hdf5` → `finalspark`); for `.h5` ask since Maxwell, MCS, and Cortical Labs (`cl`) all share the extension
3. **Output folder** — default to `mea_out` next to the input file; confirm unless user already specified

Don't ask about optional filters upfront — wait to see what the inspection reveals.

### Inspect before committing

Always run `cascade-info` first. This takes seconds and tells you whether the recording is worth full analysis, how many channels are active, and whether noise filters are needed:

```bash
cascade-info \
  --manufacturer [manufacturer] \
  --input "[wsl_path]" \
  --ae 5
```

`--input` can also be a **folder**: every file of that manufacturer's type in it is scanned (3brain `.bxr`; maxwell/cl/mcs `.h5`; axion `.csv`; finalspark `.hdf5`). Add `--output [dir]` to save `[dir]/info_summary.tsv` (one row per file). `--ae` here only *counts* active channels; nothing is removed.

Report back: duration, total channels, active channels. If active channels are very few (< 5), suggest lowering or removing `--ae` before the full run.

### Run the full pipeline

Choose the right preset based on what the inspection told you:

**Standard in-vitro (clean signal, many active channels):**
```bash
cascade \
  --manufacturer [manufacturer] \
  --input "[wsl_path]" \
  --output "[output_dir]" \
  --ae 5 --verbose
```

**Noisy recording (in-vivo, or user reports artifacts):**
```bash
cascade \
  --manufacturer [manufacturer] \
  --input "[wsl_path]" \
  --output "[output_dir]" \
  --ae 5 --max_rate 1000 --max_rpv 1.0 --max_amplitude 200 --perc 0.25 --verbose
```
Don't apply every filter blindly: add the ones that match what the first run showed, check how many channels each removed (`--verbose`), and keep the same settings across recordings that will be compared. How to choose them: `references/noisy-recordings.md`.

**Add epoch comparison** (pre vs. post stimulation) to either preset — both are needed; one alone is skipped with a warning:
```bash
  --epochA 0,60 --epochB 120,60
```

**Multi-well Axion recordings:** pass `--well A1` (or whichever well). Without `--well`, Axion merges **all wells** into one network — usually wrong for criticality, since wells are separate cultures. `cascade-info` lists the wells.

**FinalSpark:** `--input` is the `fsNNN_package.hdf5` file (the spikes), never `fsNNN_raw.hdf5`. Pass `--well 0`–`3` for one site of 8 electrodes (one organoid each); without it, all 32 electrodes are used. A whole-life export spans days, so use `--start/--dur` windows or `--sync_metrics none` (see memory above). **Stimulation-evoked spikes and artefacts are in the spike table:** in our test export about 20 % of spikes fell within 20 ms after a stimulation, against about 0.6 % expected by chance. Recommend `--stim_blank 20` (v0.9.6+) to remove them; without it `cascade` prints a `[WARN]` with the share (see `references/noisy-recordings.md`).

**Synchrony** is computed by default (`--sync_metrics core`: normalised cross-correlogram area + Synchrony Index). Use `--sync_metrics all` for the full Axion Neural Metric Tool set (slow, O(N²) in channels).

For less common parameters (bin size, connectivity threshold, shape collapse range, power-law xmin, branching-ratio lag window, etc.), see `references/flags.md`.

---

## Interpreting Results

When the user has CASCADE output, read the files and explain what the numbers mean — don't just list them back. The most important file is `criticality_report_<well>.txt`.

**Criticality metrics — interpret these together, not in isolation:**

| Metric | What it tells you | Critical value |
|--------|-------------------|----------------|
| **τ** (tau) | Avalanche size power-law exponent | ~1.5 |
| **α** (alpha) | Avalanche duration power-law exponent | ~2.0 |
| **DCC** | `\|β_pred − β_fit\|` — how far from the critical scaling relation | → 0 |
| **BR (m)** | Branching ratio, multi-lag MR estimator (Wilting & Priesemann 2018): 1.0 = critical, < 1 = subcritical, > 1 = supercritical | ~1.0 |
| **SCE** | Shape Collapse Error — `\|β_opt − β_pred\|` | → 0 |

Interpret the pattern, not just individual values. For example: "DCC of 0.06 and BR of 0.98 together suggest the network is close to criticality — the slight subcriticality is consistent with typical in-vitro cultures at this density."

**Branching ratio — read the checks before trusting m.** The report prints, under "Branching Ratio":
- `b` (subsampling factor) and the autocorrelation time `tau` in ms;
- three checks: `r_k > 0`, `exp+offset fit` (`consistent`, or `DISAGREES — possible drift`), `window >= 3 tau`. If a check fails, say so and treat m as unreliable (non-stationary activity, or raise `--br_tmax`);
- the old single-lag value (`--simple_BR`) for comparison. It is biased toward 0 under subsampling, which every MEA is — never report it as the BR for new work.

**Avalanche fit:** `xmin_S` / `xmin_D` are the fitted lower cut-offs. If `--pl_xmin_S/D` or `--pl_exclude` was used, the report also shows the automatic fit — mention both.

**Also check:**
- `bursts_<well>.tsv` — burst count, mean duration, inter-burst interval
- `isi_metrics_<well>.tsv` — channels with >1% refractory period violations are likely noisy; flag them if filtering wasn't applied
- `avalanches_<well>.tsv` — if fewer than 50 avalanches, warn that power-law fitting is unreliable (below ~1,000 the exponent is still sensitive)
- `synchrony_<well>.tsv` — `area_xcorr_norm` and `synchrony_index` (0–1); higher = more spike-timing synchrony
- `cascade_run.log` — the exact command, CASCADE version, and SUCCESS/FAILED

**Key figures to mention** (by name so the user knows where to look):
- `raster_active_<well>.png` — overall activity pattern
- `loglog_size_<well>.png` and `loglog_duration_<well>.png` — how straight the power-law lines are
- `shape_collapse_<well>.png` — quality of the criticality evidence
- `branching_mr_<well>.png` — the r_k decay used for the BR fit
- `corr_heatmap_active_<well>.png` — functional connectivity structure
- `crosscorrelogram_norm_<well>.png` — pooled spike-timing synchrony

---

## LTP/LTD Classification

When the user has firing rate files (`rates_*.tsv`) from two CASCADE runs:

```bash
cascade-ltpltd \
  --baseline "[baseline_rates.tsv]" \
  --measure "[post_stim_rates.tsv]" \
  --output "[output_dir]"
```

Multiple conditions: pass multiple `--measure` files. Adjust threshold with `--threshold 30` (default is 20% change). (`--out` from v0.8.0 still works, but `--output` is the documented name.)

Outputs per condition: `ltpltd_[label].tsv` (per-electrode call), `_bar.png` (% change per electrode), `_pie.png` (LTP/LTD/NoChange proportions). Interpret the proportions and what the LTP/LTD balance suggests about the stimulation effect. Note it is a **firing-rate** comparison per electrode, not a criticality comparison — say so if the user expects the latter.

---

## Reproducing published numbers

v0.9.x corrected two calculations, so **a default run can give different τ, α, β_pred, DCC, SCE and BR than v0.8.0**:
- power-law normalisation (since 0.9.1) → `--simple_PL` restores the old one
- branching ratio (since 0.9.4, MR estimator) → `--simple_BR` restores the old single-lag one

Add **both** to reproduce v0.8.0 exactly (the article's sample-data command line was `--ae 5 --perc 0.25 --ava_size 2 --ava_dur 1`). They print no warning; the criticality report names the method used. Use them only to regenerate old numbers — never for new analyses. To compare with the Habibollahi et al. (2023) reference pipeline: `--perc 0.25 --pl_exclude --simple_PL`.

### Does this change the article?

No for its conclusions. The main text and tables print no τ, α, β, DCC, BR or SCE values. The counts in Table 1 (active electrodes, spikes, avalanches) are unchanged, because electrode filtering and avalanche detection did not change. The only printed exponents are in Supplementary Fig. 1C–D: τ = 1.752, α = 1.974 with v0.8.0, and 1.770, 1.990 with v0.9.6, about 1 %. The README section "Differences from the published article" says the same. Keep it in proportion: don't present the updates as large.

---

## Writing the methods section

When the user needs a methods paragraph:
1. Read `cascade_run.log` in the output folder: it has the exact command line and the CASCADE version.
2. List every **non-default** flag (compare with `references/flags.md`) — filters, `--perc`, `--aval_dt`, `--ava_size/--ava_dur`, `--pl_*`, `--br_tmax`, `--sync_*`, window.
3. Write it in plain prose, e.g.: "Spike data were analysed with CASCADE v0.9.6 (Avila et al., 2026). Channels firing < 5 spikes/min or with > 1 % of ISIs < 2 ms were excluded. Avalanches were detected in bins of [dt] ms (inferred from the mean ISI) … Power-law exponents were estimated by discrete truncated maximum likelihood with xmin chosen by KS minimisation (Clauset et al., 2009); the branching ratio was estimated with the multi-lag MR estimator (Wilting & Priesemann, 2018)."
4. Cite the paper plus the method references for the metrics actually reported (`references/links.md`).
5. If `--simple_PL`/`--simple_BR` were used, say so explicitly and why (reproducing earlier results).

---

## Troubleshooting

- **No avalanches / fewer than 50** — Lower `--ava_size` (default 2) or `--ava_dur` (default 1), or reduce `--perc`. Need ≥50 for reliable fitting.
- **No active channels after filtering** — Lower or remove `--ae`; the network may simply have low activity.
- **Connectivity analysis skipped** — Requires ≥2 active channels; lower `--ae`.
- **`--max_amplitude` removes nothing / warns** — All six loaders support it in µV, but Maxwell only if the file stores amplitudes (single-peak value, not peak-to-peak). Files without amplitude data skip the filter with a warning.
- **Wrong manufacturer for `.h5`** — Maxwell, MCS, and Cortical Labs (`cl`) all use `.h5`. A wrong guess stops with `Could not read … as --manufacturer X` and, when it can tell, `This file looks like a 'cl' recording - try --manufacturer cl` (folder scan: `SKIPPED`). It never produces numbers silently — rerun with the suggested value. See `references/file-formats.md`.
- **FinalSpark: "only holds 3 ms snippets"** — the user passed `fsNNN_raw.hdf5`. Use `fsNNN_package.hdf5` from the same export. A folder scan reads the package file and lists the raw file as `SKIPPED`, which is expected.
- **`--manufacturer finalspark` not accepted** — CASCADE is older than 0.9.6 (`cascade --version`). Update: `pip install -U cascade-mea` or `conda update -c fa387 cascade-mea`.
- **Installation problems** (`command not found`, `ModuleNotFoundError`, version shows 0.8.0) — see `references/installation.md`.
- **BR check failed** (`exp+offset fit … DISAGREES — possible drift`, `r_k` negative at long lags) — activity is likely non-stationary; report m with that caveat or shorten the window. `window >= 3 tau` failed → raise `--br_tmax`.
- **Numbers differ from an older run** — expected since v0.9.1/0.9.4; see [Reproducing published numbers]. The article itself prints only τ/α in Supplementary Fig. 1C–D.
- **Verify installation:** `cascade --test` (runs 16 tests on synthetic data, no file needed; 14 before v0.9.6)
- **Citation:** `cascade --citation` prints the article reference and BibTeX.

---

## Manufacturer quick reference

| File extension | Device | `--manufacturer` |
|----------------|--------|-----------------|
| `.bxr` | 3Brain BrainWave | `3brain` |
| `.h5` | Maxwell Biosystems | `maxwell` |
| `.h5` | Cortical Labs | `cl` |
| `.h5` | Multi Channel Systems | `mcs` |
| `.csv` | Axion BioSystems (spike-list export) | `axion` |
| `fsNNN_package.hdf5` | FinalSpark Neuroplatform (v0.9.6+) | `finalspark` |

For the full list of tunable parameters for all three commands, read `references/flags.md`.
