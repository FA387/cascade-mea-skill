# CASCADE output files (v0.9.6)

Copied from the README "Output files" section (source of truth:
the README on GitHub (https://github.com/FA387/cascade-mea)). Use it to explain
what a file or figure shows. For how to interpret the criticality numbers, see SKILL.md.

## cascade output

```
mea_out/
├── raster_max_<well>.png              # spike raster — all electrode slots
├── raster_active_<well>.png           # spike raster — active electrodes only
├── rate_heatmap_max_<well>.png        # firing rate heatmap — all electrode slots
├── rate_heatmap_active_<well>.png     # firing rate heatmap — active electrodes only
├── population_rate_<well>.png         # population mean firing rate over time
├── bursts_timeline_<well>.png         # network burst events over time
├── avalanches_<well>_size.png         # avalanche size distribution
├── avalanches_<well>_duration.png     # avalanche duration distribution
├── loglog_size_<well>.png             # log-log size distribution with power-law fit
├── loglog_duration_<well>.png         # log-log duration distribution with power-law fit
├── shape_collapse_<well>.png          # avalanche shape collapse
├── wave_templates_<well>.png          # (3Brain with stored waveforms) mean spike waveform per channel
├── corr_heatmap_max_<well>.png        # correlation matrix — all electrode slots
├── corr_heatmap_active_<well>.png     # correlation matrix — active electrodes only
├── degree_hist_<well>.png             # node degree distribution
├── entropy_curve_<well>.png           # spatial entropy over time
├── active_area_curve_<well>.png       # active electrode fraction over time
├── epoch_compare_<well>.png           # (optional) epoch A vs B comparison
│
├── rates_<well>.tsv                   # per-channel firing rates (binned)
├── bursts_<well>.tsv                  # network burst event table
├── avalanches_<well>.tsv              # avalanche catalogue
├── corr_<well>.tsv                    # raw Pearson correlation matrix
├── synchrony_<well>.tsv               # synchrony metrics (Axion metric set)
├── crosscorrelogram_<well>.png        # pooled cross-correlogram
├── crosscorrelogram_norm_<well>.png   # auto-correlation-normalised correlogram
├── isi_metrics_<well>.tsv             # per-channel ISI statistics
├── active_area_<well>.tsv             # active electrode fraction per window
├── entropy_<well>.tsv                 # spatial entropy per window
├── degrees_<well>.tsv                 # node degree per channel
├── branching_mr_<well>.tsv            # MR estimator: r_k per lag with both fits
├── branching_mr_<well>.png            # r_k vs lag with the fitted exponentials
├── branching_points_<well>.tsv        # mean N(t+1) per N(t), for the --simple_BR fit
├── criticality_report_<well>.txt      # plain-text criticality metrics summary
└── cascade_run.log                    # command, CASCADE version, and run status
```

### Figures

**`raster_max_<well>.png`**
Spike raster plot with the Y-axis spanning all electrode slots on the array, including silent ones. Each dot is a single spike event. Useful for assessing spatial coverage and identifying which regions of the array are active.

**`raster_active_<well>.png`**
Spike raster restricted to electrodes that passed the active-electrode filter, one row per electrode, labelled with the real electrode IDs (since v0.9.6; before, rows were numbered 0…N−1). Provides a cleaner view of firing patterns when the array has many silent slots.

**`rate_heatmap_max_<well>.png`**
Firing rate heatmap with all electrode slots on the Y-axis and time on the X-axis. Colour encodes spikes per second in each bin. Shows how activity is distributed spatially across the full array over time.

**`rate_heatmap_active_<well>.png`**
Same heatmap restricted to active electrodes, remapped to contiguous row indices. Produces a denser, more readable plot when the proportion of active electrodes is low.

**`population_rate_<well>.png`**
Mean firing rate averaged across all active channels, plotted over time at the `--bin` resolution. Provides a single-trace overview of global activity dynamics and makes bursting epochs and silent periods immediately visible.

**`bursts_timeline_<well>.png`**
Timeline showing all detected network burst events as shaded intervals across the recording. Start time, duration, and inter-burst interval are visible at a glance. An empty plot indicates no bursts were detected above the minimum spike threshold.

**`avalanches_<well>_size.png` and `avalanches_<well>_duration.png`**
Histograms of avalanche sizes (total spikes per avalanche) and durations (number of Δt bins per avalanche). Show the range and frequency of avalanche events after size and duration thresholding. Reviewed before fitting power laws to confirm there are sufficient events.

**`loglog_size_<well>.png` and `loglog_duration_<well>.png`**
Log–log plots of the avalanche size and duration distributions with the fitted power-law line overlaid. The fitted exponents τ (size) and α (duration) and their respective xmin values are annotated. A straight line on a log–log scale is the primary visual signature of scale-free, critical dynamics.

**`shape_collapse_<well>.png`**
Avalanche profiles of different durations are rescaled by their predicted scaling exponent and overlaid. A tight collapse onto a single universal curve is a strong indicator of criticality beyond what the exponents alone can confirm. The shape collapse error (SCE) quantifies how well the profiles align.

**`corr_heatmap_max_<well>.png`**
Pearson correlation matrix across all electrode slots, with pairs above `--conn_thr` considered functionally connected. The full-grid layout makes it possible to see where connected and silent electrodes fall relative to each other on the array.

**`corr_heatmap_active_<well>.png`**
Correlation matrix restricted to active electrodes. Provides a cleaner view of the functional connectivity structure without silent electrode slots diluting the colour scale.

**`degree_hist_<well>.png`**
Histogram of node degree — the number of functional connections per electrode above the correlation threshold. Characterises whether connectivity is sparse or dense and whether hub electrodes with many connections are present.

**`entropy_curve_<well>.png`**
Normalised spatial entropy of the spike distribution computed in sliding windows of `--entropy_win` seconds. High entropy indicates spikes are distributed evenly across active electrodes; low entropy indicates spatially localised activity. Tracks changes in spatial organisation over the course of the recording.

**`active_area_curve_<well>.png`**
Fraction of active electrodes that fired in each time window. Reveals how broadly or focally the network recruits its active population over time and complements the entropy curve in characterising spatial dynamics.

**`wave_templates_<well>.png`**
Mean spike waveform per channel. Produced only for 3Brain `.bxr` files that store spike waveforms (`SpikeForms`); useful for checking that detected spikes look like action potentials rather than artefacts.

**`epoch_compare_<well>.png`**
Bar chart comparing population firing rate and active area fraction between epoch A and epoch B. Only produced when both `--epochA` and `--epochB` are provided. Provides a concise visual summary of the effect of an intervention or stimulation.

### Data files

**`rates_<well>.tsv`**
Per-channel firing rate in Hz, binned at the `--bin` interval. Each row is one channel–time-bin combination. This file is the primary input for `cascade-ltpltd` and for any downstream R-based analysis.

**`bursts_<well>.tsv`**
Table of detected network burst events: start time, end time and duration, in seconds (`burst_start_s`, `burst_end_s`, `duration_s`). An empty table indicates that no bursts exceeded the minimum spike threshold set by `--burst_min_spikes`.

**`avalanches_<well>.tsv`**
Catalogue of all avalanche events that passed the size and duration thresholds. Each row is one avalanche: its size in spikes (`avalanche_size`) and its duration in Δt bins (`duration_bins`). This is the dataset used for all criticality computations.

**`synchrony_<well>.tsv`**
Two-column metric/value table of the synchrony metrics, matching the Axion Neural Metric Tool set: `area_xcorr`, `area_xcorr_norm`, `fwhh_xcorr`, `fwhh_xcorr_norm` and, when requested, `kreuz_metric` and `synchrony_index`. Also records the parameters used (`sync_window_s`, `xcorr_bin_s`, `sync_index_sigma_s`), the pair counts, and `kreuz_subsampled` / `index_subsampled` flags so a subsampled result is never mistaken for a full one. Durations are in seconds. See the Synchrony metrics section for interpretation.

**`crosscorrelogram_<well>.png`** and **`crosscorrelogram_norm_<well>.png`**
Well-wide pooled inter-electrode cross-correlogram, standard and auto-correlation-normalised. Grey vertical lines mark the synchrony window and the shaded region is the area reported as `area_xcorr` / `area_xcorr_norm`. A tall narrow peak means high synchrony; a flat correlogram means none, in which case the FWHH value describes noise rather than network structure.

**`corr_<well>.tsv`**
Raw Pearson correlation matrix saved as a tab-separated square array. Rows and columns correspond to active channels in the order listed in `degrees_<well>.tsv`. Included for custom connectivity analyses beyond what CASCADE computes.

**`isi_metrics_<well>.tsv`**
Per-channel inter-spike interval (ISI) statistics: mean ISI, median ISI, coefficient of variation, mean firing rate, and refractory period violation fraction. Useful for characterising single-channel spiking regularity and for identifying channels with unusually high violation rates before applying noise filters.

**`active_area_<well>.tsv`**
Numerical counterpart of `active_area_curve_<well>.png`. Each row is one time window with start time, end time, and the fraction of active electrodes that fired. Used for quantitative comparisons of spatial recruitment across recordings or conditions.

**`entropy_<well>.tsv`**
Spatial entropy values per time window. Columns record window start, window end, entropy in bits (`H_bits`) and normalised entropy (`H_norm`). Numerical counterpart of `entropy_curve_<well>.png`, included for statistical comparisons across recordings.

**`degrees_<well>.tsv`**
Node degree for each active channel: the number of other channels with which it shares a correlation above `--conn_thr`. Includes channel identifiers for cross-referencing with the raster and heatmap outputs.

**`branching_mr_<well>.tsv`** / **`branching_mr_<well>.png`**
The multi-lag (MR) branching ratio of Wilting & Priesemann (2018), the default since v0.9.4: the regression slope r_k of a(t+k) on a(t) for every lag k, with the exponential fit r_k = b·m^k (m = BR) and an exponential+offset fit used as a drift check. Inspect the plot: r_k should decay exponentially to 0.

**`branching_points_<well>.tsv`**
For each population spike count N(t), the mean count in the next Δt bin and the number of bins it was averaged over (`Nt`, `E_Nt1_given_Nt`, `n_obs`): the input for the simple single-lag regression reported with `--simple_BR`. Included so the BR fit can be reproduced, visualised, or checked independently of the main pipeline.

**`criticality_report_<well>.txt`**
Plain-text summary of all criticality metrics: τ, α and their xmin values, β_pred, β_fit, DCC, the branching ratio (BR, with the MR fit's b, autocorrelation time τ, lag range and its three quality checks, plus the simple single-lag value for comparison), and the shape collapse error (SCE). Any `--simple_PL`, `--simple_BR`, `--pl_xmin_S/D` or `--pl_exclude` choice is stated in the report. This is the primary result file for reporting and manual inspection.

**`cascade_run.log`**
Records the exact command (and therefore every parameter), the CASCADE version, timestamp, Python version, platform, working directory, success or failure status, and the terminal output for each run. Report the CASCADE version with your results: criticality metrics differ between versions (see `CHANGELOG.md`).

---

## cascade-info output

Results are printed to the terminal. With `--output [output_dir]`, the same numbers are saved as `[output_dir]/info_summary.tsv`, one row per file (file, manufacturer, well, sampling rate, duration, window, total and active channels).

## cascade-ltpltd output

```
ltpltd_out/
├── ltpltd_[label].tsv               # per-electrode LTP/LTD call
├── ltpltd_[label]_bar.png           # bar chart (% change)
├── ltpltd_[label]_pie.png           # pie chart (LTP/LTD/NoChange proportions)
└── ltpltd_all_conditions.tsv        # (optional) all conditions combined
```
