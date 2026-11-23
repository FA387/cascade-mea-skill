# Dirty or noisy recordings — choosing filters

Use this when a recording looks noisy, has artefacts, odd channels, or the results look
implausible. Goal: remove channels that record noise, keep real neurons, and report exactly
what was removed.

## The four channel filters (step 2 "Preprocessing" in `--help`)

They remove **whole channels** for the whole recording (never single spikes) and run in this order:

| Order | Flag | Removes a channel if… | Catches | Starting value |
|---|---|---|---|---|
| 1 | `--ae N` | it fires < N spikes/min | silent / dead electrodes | `5` |
| 2 | `--max_rate N` | it fires > N spikes/min | electrical noise counted as spikes | `1000` (≈ 17 Hz) |
| 2 | `--max_rpv N` | > N % of its inter-spike intervals are < 2 ms | noise (real neurons can't fire that fast) | `1.0` |
| 3 | `--max_amplitude N` | any single spike > N µV peak-to-peak | mechanical knocks, interference, stimulation artefacts | `200` |

All default to off (0). `--verbose` prints how many channels each filter removed and their IDs.

## Symptom → what to try

| What you see | Likely cause | Try |
|---|---|---|
| Raster has a few channels that are solid lines / far busier than the rest | noise channels | `--max_rate 1000` |
| `isi_metrics_<well>.tsv`: `refractory_viol_frac` > 0.01 on some channels (column is a fraction: 0.01 = 1 %) | noise or poorly sorted multi-unit | `--max_rpv 1.0` |
| Sudden vertical stripes across many channels at once in the raster | artefact events (movement, handling, stimulation) | `--max_amplitude 200`; if stimulation, also consider analysing a window without stim (`--start/--dur`) |
| Very large spikes on a few channels, or the user mentions bumps/knocks | mechanical artefacts | `--max_amplitude 200` |
| Thousands of channels, most nearly silent (HD-MEA, 3Brain/Maxwell) | inactive electrodes dilute the population | `--ae 5` |
| Avalanches dominated by tiny noise events; τ, BR look implausible | background noise in the population count | `--perc 0.25` (step 3; avalanche threshold, not a channel filter) |
| Branching-ratio checks fail (`exp+offset fit … DISAGREES — possible drift`) | non-stationary activity (drift, bursts of artefact) | filter as above, or analyse a stable window with `--start/--dur` |

## Recommended workflow

1. **Inspect:** `cascade-info --manufacturer X --input file --ae 5` → total vs active channels.
2. **First run, light:** `cascade ... --ae 5 --verbose`. Look at `raster_active_<well>.png`,
   `isi_metrics_<well>.tsv`, and the number of avalanches.
   (ISI metrics are computed on the channels kept — run without the noise filters first
   to see the noisy ones.)
3. **Add filters only for problems you saw**, one at a time, with `--verbose`:
   `--max_rate 1000 --max_rpv 1.0`, then `--max_amplitude 200` if artefacts remain.
4. **Sanity check what was removed:** if a filter removes more than ~10–20 % of active
   channels, the threshold is probably too strict for this culture/device — loosen it
   (e.g. `--max_amplitude 300`, `--max_rate 1500`) rather than trusting the result.
5. **Use the same filter settings for every recording you compare** (baseline vs treatment,
   culture A vs B). Different filters between conditions make the comparison invalid.
6. **Report the settings** (they are in `cascade_run.log`): "channels with < 5 spikes/min,
   > 1000 spikes/min, > 1 % ISIs < 2 ms, or any spike > 200 µV were excluded (N of M removed)."

Full noisy preset:
```bash
cascade --manufacturer X --input file --output out \
        --ae 5 --max_rate 1000 --max_rpv 1.0 --max_amplitude 200 --verbose
```

## Things to know about `--max_amplitude`

- One large spike removes the channel for the whole recording (no per-spike rejection, no
  time windows).
- Removing even a few channels changes the spike pool, so τ, α, DCC and shape collapse can
  shift. Measured on the published 3Brain sample: 200 µV removed 3 of 142 active channels
  (~2 %) and τ moved 1.570 → 1.517, DCC 2.561 → 2.400, β_opt 0.875 → 2.325. Always re-check
  `shape_collapse_<well>.png` after enabling it.
- On the published MCS sample the largest true spike was ~163 µV, so 200 µV removed nothing;
  on the Axion sample it removed 0 of 70. 200 µV is a starting point, not a universal value.
- Device differences: all loaders give µV, but **Maxwell stores a single-peak value** (not
  peak-to-peak) and only if the file has amplitudes. A file without amplitude data skips the
  filter with a warning — the run continues.

## What filtering cannot fix

- A recording with too few active channels or < 50 avalanches: no filter makes the
  power-law fit reliable — say so.
- Stimulation in closed-loop recordings (e.g. Cortical Labs game recordings): stim periods stay in the data unless
  a window is chosen with `--start/--dur`.
- **FinalSpark stimulation:** stims are logged separately, but the spikes they cause are in
  the events table. In our 5.7-day test export (0.8 million stims, 2.6 million spikes), the share of spikes after a stim was:

  | Time after stim | Share of spikes |
  |---|---|
  | 0–2 ms (mostly artefact) | 2.2 % |
  | 2–5 ms | 11.4 % (peak) |
  | 5–20 ms | 6.7 % |
  | 20–100 ms | 8.4 % |
  | > 100 ms (spontaneous) | 71.3 % |

  So about **20 % fall within 20 ms**, against 0.6 % expected by chance (35×). 0–2 ms is likely
  artefact; 2–50 ms may be real evoked spikes, which timing can't separate. Either way it is
  not spontaneous activity and can inflate avalanches. It's an experiment effect, not a CASCADE
  or math error. **Since v0.9.6: `--stim_blank N`** removes every spike within N ms after each
  stim, on all channels, before the channel filters (`5` mainly artefacts, `20` also evoked
  responses). Without it, `cascade` prints a `[WARN]` with the share of spikes within 20 ms after a
  stim; `cascade-info` prints the number of stims. Use `--stim_blank 20` for spontaneous-activity
  analysis of FinalSpark data, keep it the same across compared recordings, and report it.
  The share varies a lot by period: one hour on one site had only 0.2 %.
- Spike sorting quality: CASCADE uses the spikes the acquisition software detected.
