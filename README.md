# hd-emg-visual-inspector-downloads
# HD-EMG Visual Inspector

A Python/Flet desktop application for inspecting high-density surface EMG recordings and recording channel-quality decisions (*interpolation* and *exclusion*) in an Excel workbook.

Developed by Taian Martins Vieira, LISiN — Politecnico di Torino.

## Status and downloads

**Version 0.1.0.** Standalone Windows and macOS builds have been built and tested by the developer, with successful testing also reported by colleagues. Testing is ongoing; a complete list of tested operating-system versions and processor architectures has not yet been established.

The first GitHub release is being prepared. For now, use the testing package shared directly by the developer. Once published, application downloads and the illustrated user guide will be available on the [Releases page](https://github.com/taian-vieira/hd-emg-visual-inspector/releases). While the repository is private, access requires a GitHub invitation.

## Run the desktop application

The packaged application includes its Python runtime and dependencies. Users do not need to install Python, Flet, Flutter, or development tools.

### Windows

1. Obtain the Windows ZIP and extract the **entire archive** to a local folder.
2. Open the application folder and launch its `.exe` file. Keep all accompanying folders and files alongside it; do not move only the executable.
3. Click **Open** and select the workbook in your data folder.

### macOS

1. Obtain the macOS ZIP and extract it in Finder.
2. Keep the complete `.app` bundle together; it may be copied to **Applications** or another local folder.
3. Launch the `.app`, click **Open**, and select the workbook in your data folder.

The current Mac build has not been signed with a Developer ID certificate and notarized for wider distribution. If macOS prevents it from opening, send the exact message to the developer so the cause can be checked.

### User guide and example data

Start with **User_Guide.pdf**, the illustrated guide supplied with the testing package. It explains workbook selection, trial inspection, channel decisions, saving, and keyboard shortcuts. The guide will also accompany the first GitHub release.

The separately shared example dataset contains a workbook with the full trial table, but recordings for **six trials only**. See the guide for the included subjects and trials. Selecting other rows is expected to report missing recordings. Preserve all subject, recording-folder, and file names, and use a copy of the example workbook when testing changes.

Application archives published on GitHub will contain no real participant workbooks or raw recordings. Example data are shared separately from GitHub.

## Workbook and recordings

Keep participant data outside the source repository. The workbook must be inside the directory containing the subject folders:

```text
data/
├── workbook.xlsx
└── SubjectCode/
    └── emg/
        └── DataFolder/
            └── DataFolder_ProbeID_EMG_raw.sig
```

The parent directory and workbook may have other names. Recording paths are resolved relative to the selected workbook, not to the application folder.

For each probe listed in `Sensors`, the application requires this exact pattern:

```text
<workbook directory>/<Subject Code>/emg/<Data Folder>/<Data Folder>_<probe>_EMG_raw.sig
```

For example, the synthetic values `Subject Code = ViTa`, `Data Folder = ViTa22052023_161753`, and `Sensors = M02BA1 M02BD4` require:

```text
data/ViTa/emg/ViTa22052023_161753/ViTa22052023_161753_M02BA1_EMG_raw.sig
data/ViTa/emg/ViTa22052023_161753/ViTa22052023_161753_M02BD4_EMG_raw.sig
```

The acquisition system creates the folder and file names automatically. Match folder names, probe identifiers, and capitalization exactly.

### Workbook columns

Use an `.xlsx` workbook. The application selects the sheet named exactly **EMG**, if present; otherwise, it uses the first worksheet. Put headers in row 1 and one trial per subsequent row.

All nine columns below must exist, in any order. Header matching ignores capitalization, leading/trailing whitespace, and repeated whitespace between words. Duplicate headers after normalization are rejected. Extra metadata columns are permitted and displayed in the details panel.

| Required header | Content |
| --- | --- |
| Subject Code | Text matching the subject folder, e.g. `ViTa`. Store numeric-looking codes as text. |
| Trial Number | Trial identifier. Rows without a subject code or trial number are skipped. |
| Trial Comment | Optional descriptive text; the column itself is required. |
| Data Folder | Text matching the recording subfolder, e.g. `ViTa22052023_161753`. |
| Sensors | Probe identifiers separated by spaces, e.g. `M02BA1 M02BD4`; their order determines concatenated channel numbering. |
| Electrodes Order | One-based acquisition-channel numbers in display order, entered as text. Supports space-separated numbers and MATLAB-style ranges such as `1:32` or `32:-1:1`. |
| Grid Columns | Positive integer number of grid columns. The electrode count must be divisible by this number. |
| EMG Chs to Interpolate | Blank or one-based acquisition-channel numbers, separated by spaces or commas, e.g. `3 7 12`. Ranges are not supported here. |
| EMG Chs to Exclude | Blank or one-based acquisition-channel numbers, separated by spaces or commas. Ranges are not supported here. |
| EMG Sync Events | Stored trigger events as JSON text, grouped by the probe identifiers used in `Sensors`. |

`Electrodes Order` fills each display column from top to bottom, then advances to the next column. Channel IDs must exist in the loaded recordings. With two 32-channel probes, the first probe supplies channels 1–32 and the second supplies 33–64. Decision columns use those acquisition-channel IDs, not positions in the reordered display. Keep interpolation and exclusion lists disjoint.

### Supported recording assumptions

The current reader expects headerless binary `.sig` files containing **little-endian unsigned 16-bit values**, arranged as successive samples of **32 channels per probe**. It assumes a sampling frequency of **2048 Hz** and applies the fixed conversion factor `2000 / 183 / (2**16 - 1)` mV per count after removing each channel's mean.

> [!NOTE] Only MEACs probes for now
> *The current version works only with MEACs (32-ch-based) probes. A future release will include also GAM (64-ch-based) probes. See https://recbioengineering.com/*

These acquisition settings are defined in `app_state.py`; additional workbook metadata does not override them. Other acquisition formats require explicit code adaptation and validation. Multiple probe files should be synchronized; if their lengths differ, only the length of the shortest file is read from each.

## Inspection workflow

1. Click **Open** and choose the workbook.
2. Select a subject and trial from **Load trial**.
3. Inspect the grid using **Processing** and **Visualization**: monopolar or differential signals, optional 10–400 Hz band-pass filtering, notch filtering off/50/60 Hz, amplitude scale, channel numbers, zoom, and pan.
4. In **monopolar** mode, use the channel controls to toggle interpolation or exclusion decisions. Assigning one removes the other for that channel.
5. Click **Save** to write pending decisions to the selected workbook.

Interpolation previews the mean of available surrounding grid electrodes, omitting neighbours marked for interpolation or exclusion. If no eligible neighbour exists, the original signal is retained. Differential signals are calculated between adjacent rows within each grid column.

**Save overwrites the opened workbook** and writes sorted, space-separated channel IDs into the two decision columns for edited trials. Keep a backup and close the workbook in Excel before saving. Save before opening another workbook or closing the application; decisions are not automatically saved. Trial switching within the same workbook retains pending decisions in memory.

Raw `.sig` files are never modified. Filtering and interpolation affect the display; the application does not export processed recordings.

### Keyboard shortcuts

| Shortcut | Action |
| --- | --- |
| **Ctrl+Z** (Windows) / **Cmd+Z** (macOS) | Toggle zoom; this is **not an undo command**. |
| **Ctrl+D** (Windows) / **Cmd+D** (macOS) | Toggle pan. |
| **D** | Select the differential spatial filter. |
| **M** | Select the monopolar spatial filter. |
| **↑ / ↓** | Change the amplitude scale. |
| **S** | Show or hide channel numbers. |

## Synchronization and batch processing

### Inspecting and correcting synchronization events

Open a trial and use the synchronization button to inspect the auxiliary trigger signal associated with each probe. Select the probe to display. Event markers and the event list show the identified changes in state. An optional EMG channel can be displayed alongside the auxiliary signal to help assess the recording.

Automatic detection uses a threshold to identify both rising and falling transitions; these are stored simply as events, without edge labels. Always inspect the results before using them to align recordings for export.

Mouse controls in the synchronization dialog:

- Left-click and drag horizontally to zoom in time. This does not add or remove events.
- Double-click to restore the full recording interval.
- Move the pointer over the plot to inspect its time position.
- Right-click near an event to remove it, or away from existing events to add an event at that instant. Zoom in when precise placement is needed.

Closing the dialog accepts its current event definitions and updates the active trial and workbook in memory. This also applies when dismissing the dialog with Escape or by clicking outside it, where allowed. Use the main Save button to write changes to the Excel workbook.

### Workbook synchronization information

The workbook requires an additional column named `EMG Sync Events`. Events are stored as JSON text, grouped by the probe identifiers used in `Sensors`. For example:

```json
{"M01D34": [2048, 122880], "M01D35": [2176, 123008]}
```

The values are zero-based sample indices: sample 0 is the first sample. An event marks the first sample after a state change. The dialog displays event times in seconds. The same external trigger can occur at different sample indices in different probe recordings.

A blank workbook cell means automatic detection should be attempted during batch processing. If detection finds no transitions, the report shows empty lists for the affected probes. Such recordings require inspection; when no events are stored, later batch runs attempt detection again.

### Batch synchronization detection

Use Batch Sync Aux in the Processing menu to process the trials in the loaded workbook. Trials without stored synchronization results are processed automatically; existing results are retained.

The bottom sheet reports the results and any unavailable directories or recordings. Its header remains fixed while the trial list scrolls. Use the load button beside a trial to open it for inspection and correction.

Batch detection updates the workbook in memory. Save the workbook to retain the results on disk.

### Batch export for DEMUSE decomposition

The decomposition export reorganizes recordings beneath a common export folder, with one subfolder per subject and the trial files directly inside each subject folder. This prepares the recordings for the DEMUSE batch-decomposition workflow using the supplied MATLAB CKC reader.

Each exported trial contains a binary `.sig` recording and a matching `.mat` metadata file. A MATLAB reader named `CKC_reader_<workbook_name>.m` is placed in the common export folder.

For trials with several probes, their channels are joined into one recording in the order specified in `Sensors`. Stored events determine the interval taken from each probe:

- Two or more events: begin at the first event and stop before the second event.
- One event: begin at that event and continue through the available recording.
- No events: begin at sample zero.

The exporter retains the same number of time samples from every probe, using the shortest available interval. Each probe can begin at a different original sample index. The alignment assumes that their first events represent the same external transition. Without usable events, retaining equal lengths alone does not establish synchronization.

Metadata describe acquisition scaling, sampling frequency, recording dimensions, subject and trial information, sensor identifiers, electrode arrangement, and channel interpolation and exclusion decisions.

The exported samples retain the original signal values within the selected intervals. The CKC reader preserves those signals; it does not interpolate channels. Interpolation decisions are supplied as metadata so that the reader can be amended to apply them if required. Channel exclusion is likewise recorded as metadata; do not assume that flagged channels have been removed from the binary recording. Display filters and spatial-filter settings are not applied by this raw-data export.

The interface prepares the files; DEMUSE performs the decomposition separately. The original acquisition files remain unchanged.

Recommended workflow:

1. Load the workbook and run batch synchronization detection.
2. Review the report and inspect trials needing correction.
3. Close each inspection dialog to accept its events, then save the workbook.
4. Run the decomposition export and review its report.
5. Use the supplied CKC reader for the exported recordings in DEMUSE.
