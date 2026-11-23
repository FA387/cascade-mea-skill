# MEA file formats — what CASCADE reads from each manufacturer

Use this when someone is unsure which file to give CASCADE, which `--manufacturer` to use,
gets "No wells/streams found" / "not found" errors, or wants to add a new device.

CASCADE reads **detected spikes** (spike times, channel, optionally waveform/amplitude),
not raw voltage traces. The recording software must have run spike detection and saved
the spikes into the file. Every loader converts its file into one common object
(`MEARecording`: spike times in seconds per channel, sampling rate, channel list,
duration, optional per-spike amplitudes in µV), so all analysis is identical across devices.

## Quick table

| Device | `--manufacturer` | File | Spikes read from | Amplitudes for `--max_amplitude` |
|---|---|---|---|---|
| 3Brain BrainWave | `3brain` | `.bxr` | `/Well_X/SpikeTimes`, `SpikeChIdxs` | From `SpikeForms` waveforms (µV) — if present |
| Maxwell Biosystems | `maxwell` | `.h5` (`data.raw.h5`) | `/data_store/<stream>/spikes` (`frameno`, `channel`) | `amplitude` field — single-peak value, only if stored |
| Cortical Labs | `cl` | `.h5` | `/spikes` table (`timestamp`, `channel`) | 75-sample waveform snippets (µV) |
| Multi Channel Systems | `mcs` | `.h5` (MCS HDF5 v3) | `/Data/Recording_0/SegmentStream/` spike cutouts | Waveform cutouts, converted ADC → µV |
| Axion BioSystems | `axion` | `.csv` (spike list) | Time (s), Electrode, Amplitude (mV) columns | Amplitude column (mV → µV) |
| FinalSpark (v0.9.6+) | `finalspark` | `fsNNN_package.hdf5` | `/fsNNN_wholelife_events` (`time_of_event`, `electrode`) | `max_voltage_uv`, signed single peak (absolute value used) |

`.h5` is shared by three devices — ask which instrument recorded it, or try `cascade-info` with each.
A wrong guess stops with a clear message, never with numbers. Single file:
`Could not read X as --manufacturer mcs (...). This file looks like a 'cl' recording - try
--manufacturer cl.` (the suggestion appears when the file's own attributes identify the
device). Folder scan: `SKIPPED: … not readable as --manufacturer X`. Tested on all three
`.h5` samples × the three `.h5` manufacturers: only the right pairing loads.

---

## 3Brain BrainWave (`.bxr`)

Produced by BrainWave after spike detection (the `.bxr` result file, not the raw `.brw`).

```
/ (root attributes)   SamplingRate (Hz), ExperimentDateTimeUtc, PlateModel
/Well_A1/
    SpikeTimes     frame index of each spike
    SpikeChIdxs    channel index of each spike
    SpikeUnits     unit ID per spike (optional)
    SpikeForms     waveforms (optional) → amplitudes
    StoredChIdxs   channels stored
    TOC            [start_frame, end_frame] blocks
```
Wells: `Well_A1`, … — `--well Well_A1`. Default: first well.
Without `SpikeForms`, `--max_amplitude` prints a warning and is skipped.
Typical size: 4,096 electrodes (HD-MEA), most inactive → use `--ae`.

## Maxwell Biosystems (`.h5`, usually `data.raw.h5`)

Produced by MaxLab Live. Spikes must have been recorded (spike-only or raw + spikes).
```
/data_store/<stream>/spikes      structured: frameno, channel, amplitude (optional)
/data_store/<stream>/settings/   sampling (Hz), gain, lsb, mapping (channel → electrode)
/assay/inputs/                   record_time (s), electrodes, spike_only
```
Streams/wells: `data0000`, `data0001`, … — `--well data0000`. Default: first stream.
The `amplitude` field is a single-peak magnitude, not peak-to-peak — a µV threshold is
not exactly equivalent to other devices; calibrate per device.
Files can be very large (GBs) — inspecting is still fast because only spikes are read.

## Cortical Labs (`cl`, `.h5`)

Produced by the Cortical Labs system (standard recordings and game/closed-loop recordings).
CASCADE recognises these files by the `system_id` the device writes (it starts with `CL1-`).
```
/ (root attributes)  sampling_frequency (Hz), channel_count (64), duration_seconds, uV_per_sample_unit
/spikes              table: timestamp (frame), channel (0–63), samples (75-point waveform, µV)
/stims               stimulation times (not used)
/data_stream/...     game state etc. in game recordings (ignored)
```
One well: `default`. Stimulation artefacts are not removed by CASCADE — for closed-loop
recordings consider `--max_amplitude` and `--max_rpv`, and say that stim periods are included.

## Multi Channel Systems (`.h5`, MCS HDF5 protocol v3)

Produced by Multi Channel Experimenter, or converted with Multi Channel DataManager.
The file must contain a **SegmentStream** with spike cutouts (i.e. spike detection was
run and spikes saved); raw `AnalogStream` alone is not enough.
```
/Data/Recording_0/AnalogStream/Stream_0/InfoChannel     channel labels, Tick (µs/sample), ConversionFactor, Exponent
/Data/Recording_0/SegmentStream/Stream_0/
    InfoSegment         segment → source channel
    SegmentData_<N>     waveform cutouts (ADC units → converted to µV)
    SegmentData_ts_<N>  spike timestamps (µs)
```
Stream: `Recording_0`. Typical: 60 electrodes.

## Axion BioSystems (`.csv` spike list)

Exported from AxIS / AxIS Navigator as a **spike list** CSV (`*_spike_list.csv`).
Layout: metadata key–value pairs in the left columns and spikes in the right columns
(`Time (s)`, `Electrode`, `Amplitude (mV)`) on the same rows.
Electrodes are named `WELL_ELECTRODE`, e.g. `B1_36`.
- `--well B1` analyses one well. **Without `--well`, all wells are merged into one
  network** — usually wrong for criticality because wells are separate cultures.
  `cascade-info` lists the wells.
- Amplitudes are converted mV → µV.

## FinalSpark Neuroplatform (`finalspark`, `.hdf5`) — v0.9.6+

A whole-life export (`fsNNN.zip`) holds one MEA for its whole life (days), as two HDF5
files plus FinalSpark's helpers (`README.md`, `demo.ipynb`, `demo_utils.py`,
`fsNNN_segment_index.parquet`, `pyproject.toml`). Both files are pandas "table" format;
CASCADE reads them with h5py, so PyTables is not needed.

| | `fsNNN_package.hdf5` (example: 210 MB) | `fsNNN_raw.hdf5` (example: 7.6 GB) |
|---|---|---|
| Tables | `events`, `stimulations`, `metadata`, `incubator_*` (CO2, O2, temperature, door; humidity and pressure may be missing) | `raw`, `metadata` |
| Spikes | **yes**: one row per spike, `time_of_event` (ns, UTC), `electrode`, `max_voltage_uv` | the same spikes as waveforms, not as a list |
| Voltage | peak only | 3 ms per spike (1 ms before, 2 ms after) at 30 kHz; nothing between spikes |
| CASCADE | **`--input` this file** | refused with a clear message (only the 2 KB header is read) |

- **No continuous recording exists.** The Intan RHS 32 headstages record at 30 kHz,
  but only threshold crossings (6 × median noise SD) are stored.
- **Hardware:** a custom 32-electrode MEA (HEPIA, Wertenbroek et al. 2021) with an
  air-liquid interface; 4 sites × 8 electrodes, one forebrain organoid (~500 µm) per site.
  The same electrodes record and stimulate.
- **Electrode IDs** = 32 × MEA + 8 × site + n (0–127 across FinalSpark's 4 MEAs), as in
  their `np-utils` code. Each export is one MEA, and the loader finds the base from the data.
  `--well 0`–`3` = one site; no `--well` = all 32.
- Time 0 = first event (as in FinalSpark's `demo_utils`); `t0_utc` is kept in metadata.
- **Stimulations** are in their own table, but the spikes they cause are in `events`.
  CASCADE doesn't remove them (see `noisy-recordings.md`).
- Tested on a 5.7-day export (2.6 million spikes): `cascade-info` on the whole life used
  0.3 GB of memory, `cascade` on a 1 h window 0.9 GB.
- Source: the export's README and the *Neuroplatform* paper (Jordan et al. 2024, Front. AI);
  links in `links.md`.

## Plain spike CSV (cascade-info only)

`cascade-info` can also inspect a generic CSV with a channel column (`ch`, `channel`,
`electrode`, …) and either a time column in seconds (`time_s`, `t`, …) or a frame column
(`frame`, `sample`, …) plus `--sr <Hz>`. The full `cascade` pipeline needs one of the six
manufacturer formats.

---

## Looking inside an unknown file

```bash
python -m cascade.utils.inspect_file dump "/path/to/file.h5"
```
Prints the HDF5 tree (groups, datasets, shapes, attributes). Use it to check that spikes
are present, or before writing a loader for a new device (see README "Adding a new
manufacturer": one new file in `cascade/loaders/`, registered in `LOADERS` and `EXTENSIONS` in `cascade/loaders/__init__.py`).
