# CASCADE Full Flag Reference (v0.9.6)

Same content and order as the README "All flags" section and each tool's `--help`.
Source of truth: the README on GitHub (https://github.com/FA387/cascade-mea) — if they disagree, run `--help`.

Flags are grouped by workflow step, in the same order as `--help`:

| Step | What it decides |
|---|---|
| **1. Setup** | Which file, which device, and where the results go |
| **2. Preprocessing** | Which part of the recording and which channels are trusted |
| **3. Analysis** | The parameters and assumptions of each calculation |
| **4. Output** | Extra figure formats and terminal messages |
| **5. Misc** | Information only; nothing is analysed |

Most runs only need step 1. Add step 2 filters for noisy recordings, and change step 3 only with a reason. Report any non-default value in your methods.

All three tools write to an `--output` folder. `cascade` and `cascade-info` read with `--input`; `cascade-ltpltd` compares two results with `--baseline` and `--measure`.

### cascade

#### 1. Setup — input and output

| Flag | Default | Description |
|---|---|---|
| `--manufacturer` | required | `3brain`, `maxwell`, `cl`, `mcs`, `axion`, or `finalspark` (v0.9.6+) (see [Supported manufacturers](README: Supported manufacturers)) |
| `--input` | auto-search | Path to the `.bxr`, `.h5`, `.csv` or FinalSpark `.hdf5` file. If omitted, the first `.bxr` or `.h5` file in the current folder is used |
| `--well` | first | Well or stream name. Axion: all wells are merged unless a well is given. FinalSpark: `0`–`3` = one site of 8 electrodes; none = all 32 |
| `--output` | `mea_out` | Output folder |

#### 2. Preprocessing — which data and which spikes to trust

**Recording window**

| Flag | Default | Description |
|---|---|---|
| `--start` | `0` | Analysis window start (seconds) |
| `--dur` | full | Analysis window duration (seconds) |

**Spike filtering**

| Flag | Default | Description |
|---|---|---|
| `--stim_blank` | off | (v0.9.6+) Remove spikes within N ms after each stimulation, all channels, before other filters (try `20`). Needs stim times: FinalSpark |
| `--ae` | off | Active-electrode filter: keep channels with ≥N spikes/min |
| `--max_rate` | off | Noise filter: remove if rate >N spikes/min (try `1000`) |
| `--max_rpv` | off | Noise filter: remove if >N% of ISIs <2 ms (try `1.0`) |
| `--max_amplitude` | off | Noise filter: remove if any spike exceeds N µV peak-to-peak (try `200`). All six loaders give µV; Maxwell stores a single-peak value rather than peak-to-peak, and only when the file contains amplitudes |

#### 3. Analysis — parameters and assumptions of each calculation

**Firing rate and connectivity**

| Flag | Default | Description |
|---|---|---|
| `--bin` | `1.0` | Bin size for firing rates, rate heatmaps and connectivity (seconds) |
| `--conn_thr` | `0.3` | Pearson \|r\| threshold for a functional connection |

**Network bursts**

| Flag | Default | Description |
|---|---|---|
| `--burst_min_spikes` | `5` | Minimum spikes per burst event |
| `--burst_max_gap` | `0.1` | Maximum inter-spike gap within a burst (seconds) |

**Synchrony**

| Flag | Default | Description |
|---|---|---|
| `--sync_metrics` | `core` | `none` / `core` / `xcorr` / `kreuz` / `index` / `all`. `core` = normalised correlogram area + Synchrony Index. The correlogram is cheap (FFT); `kreuz` and `index` are O(N²) in channels |
| `--sync_window` | `20.0` | Synchrony window in **ms** — the window around zero lag used for the area metrics. `5` for millisecond timescale, `100` for slow |
| `--sync_bin` | `1.0` | Cross-correlogram bin width in **ms**. Absolute, never inferred from sampling rate |
| `--sync_max_lag` | `500.0` | Maximum correlogram lag in **ms**. Must be wide enough to contain the peak or FWHH cannot be measured |
| `--sync_sigma` | = window | Gaussian σ in **ms** for the Synchrony Index |
| `--sync_max_channels` | `64` | Channel cap for the O(N²) measures; `0` = no cap |

**Avalanche detection**

| Flag | Default | Description |
|---|---|---|
| `--ava_size` | `2` | Minimum avalanche size (spikes). Single spikes are not avalanches. |
| `--ava_dur` | `1` | Minimum avalanche duration (bins). Default=1 allows synchronous bursts. |
| `--aval_dt` | inferred | Avalanche bin width (seconds); auto = mean ISI clipped to 1–50 ms |
| `--perc` | off | Population activity percentile threshold. `0.25` is the default of the Habibollahi et al. (2023) reference code (their paper reports 40% of the median) |

**Power-law fit (τ, α, DCC)**

| Flag | Default | Description |
|---|---|---|
| `--burst_min` | `12` | Upper xmin search limit (size exponent fit) |
| `--t_min` | `4` | Upper xmin search limit (duration exponent fit) |
| `--pl_xmin_S` | auto | Fix the size-fit xmin instead of searching. τ, DCC, SCE and the plot use it; the automatic fit is still written to the report |
| `--pl_xmin_D` | auto | Fix the duration-fit xmin instead of searching (α, β_fit, DCC) |
| `--pl_exclude` | off | Also trim the upper tail before fitting, with the `EXCLUDE` step of the Habibollahi et al. (2023) Python reference code (the MATLAB original differs). For comparison with that pipeline only: add `--simple_PL` to match its code exactly. The untrimmed fit is still reported — see `CHANGELOG.md` |
| `--skip_rare_sizes` | off | Exclude sizes seen only once from the log-log plot line (does not affect the MLE exponent) |
| `--skip_rare_durations` | off | Exclude durations seen only once from the log-log plot line (does not affect the MLE exponent) |

**Shape collapse**

| Flag | Default | Description |
|---|---|---|
| `--sc_Dmin` | `4` | Minimum duration (bins) in shape collapse |
| `--sc_Dmax` | `20` | Maximum duration (bins) in shape collapse |

**Branching ratio**

| Flag | Default | Description |
|---|---|---|
| `--br_tmax` | `2000` | Longest lag (ms) for the MR estimator. Should span several autocorrelation times; the report warns when it does not |

**Entropy and active area**

| Flag | Default | Description |
|---|---|---|
| `--entropy_win` | `10` | Window size for entropy and active-area curves (seconds) |

**Epoch comparison**

| Flag | Default | Description |
|---|---|---|
| `--epochA` | off | Epoch A as `start,duration` (e.g. `0,60` = 0–60 s). Both A and B required. |
| `--epochB` | off | Epoch B as `start,duration` (e.g. `120,60` = 120–180 s). Both A and B required. |

**Reproducing v0.8.0 (not for new analyses)** — see [Reproducing the published v0.8.0 results](SKILL.md: Reproducing published numbers)

| Flag | Default | Description |
|---|---|---|
| `--simple_PL` | off | Use the power-law normalisation of v0.8.0–v0.9.0 (biased below ~1,000 avalanches) |
| `--simple_BR` | off | Report the simple single-lag branching ratio of v0.8.0–v0.9.3 instead of the MR estimator (biased toward 0 under subsampling) |

Use both together to reproduce v0.8.0 exactly. No warning is printed; the criticality report names the method.

#### 4. Output — figures and terminal

| Flag | Default | Description |
|---|---|---|
| `--svg` / `--jpg` | off | Save figures as SVG/JPG in addition to PNG |
| `--verbose` | off | Print progress to terminal |

#### 5. Misc — information only

| Flag | Description |
|---|---|
| `--help` | Show all flags and exit |
| `--version` | Print the CASCADE version and exit |
| `--citation` | Print how to cite CASCADE (text and BibTeX) and exit |
| `--test` | Run the 16 built-in self-tests on synthetic data (no recording needed) |

### cascade-info

#### 1. Setup — input and output

| Flag | Default | Description |
|---|---|---|
| `--manufacturer` | required | `3brain`, `maxwell`, `cl`, `mcs`, `axion`, or `finalspark` (v0.9.6+) |
| `--input` | required | A recording file, **or a folder**: every file of the manufacturer's type in it is scanned (3brain `.bxr`; maxwell/cl/mcs `.h5`; axion `.csv`; finalspark `.hdf5`, raw file skipped) |
| `--well` | first | Well or stream name. Axion: all wells are merged unless a well is given. FinalSpark: `0`–`3` = one site of 8 electrodes; none = all 32 |
| `--sr` | — | Sampling rate in Hz (only for a plain CSV with a frame column instead of time) |
| `--output` | off | Output folder; saves `info_summary.tsv` (one row per file). Without it, results are only printed |

#### 2. Preprocessing — window and active channels

| Flag | Default | Description |
|---|---|---|
| `--start` | `0` s | Window start (seconds) |
| `--dur` | full | Window duration (seconds) |
| `--ae` | `0` | Count channels with ≥N spikes/min as active (counted only; nothing is removed) |

#### 5. Misc — information only

| Flag | Description |
|---|---|
| `--help` / `--version` / `--citation` | Show all flags / the version / how to cite CASCADE, and exit |

### cascade-ltpltd

#### 1. Setup — input and output

| Flag | Default | Description |
|---|---|---|
| `--baseline` | required | Path to `rates_*.tsv` from the baseline CASCADE run |
| `--measure` | required | Path(s) to one or more `rates_*.tsv` files to compare |
| `--output` | `ltpltd_out` | Output folder |

#### 3. Analysis — LTP/LTD classification

| Flag | Default | Description |
|---|---|---|
| `--threshold` | `20` | LTP/LTD threshold (% change from baseline) |

#### 4. Output — figures and terminal

| Flag | Default | Description |
|---|---|---|
| `--svg` / `--jpg` | off | Save figures as SVG/JPG |
| `--verbose` | off | Print per-electrode table to terminal |

#### 5. Misc — information only

| Flag | Description |
|---|---|
| `--help` / `--version` / `--citation` | Show all flags / the version / how to cite CASCADE, and exit |

---

## Recommended flag combinations

**Standard clean in-vitro:**
```
--ae 5
```

**Noisy in-vitro (recommended starting point):**
```
--ae 5 --max_rate 1000 --max_rpv 1.0 --max_amplitude 200
```

**25th-percentile avalanche threshold (Habibollahi et al. 2023 reference-code default):**
```
--ae 5 --perc 0.25
```

**Compare with the Habibollahi et al. (2023) reference pipeline (upper-tail trimming, old normalisation):**
```
--perc 0.25 --pl_exclude --simple_PL
```

**Reproduce the published CASCADE article (v0.8.0) sample results exactly:**
```
--ae 5 --perc 0.25 --ava_size 2 --ava_dur 1 --simple_PL --simple_BR
```
`--simple_PL` / `--simple_BR` are for reproducing old numbers only — never for new analyses.
