# HD-EMG Visual Inspector

A desktop application for inspecting high-density surface EMG recordings, recording channel-quality decisions, reviewing synchronization events, and preparing recordings for DEMUSE decomposition.

Developed by **Taian Martins Vieira**, **LISiN — Politecnico di Torino**.

This public repository provides application downloads and user documentation. The development source repository is maintained separately and is private.

**Platform availability:** Version 0.2.0 is currently available for Windows.
The macOS package for this version is being prepared and will be added
to this release once tested.

## Main features

- Support for **MEACs probes (32 channels per probe)** and **BAM probes (64 channels per probe)**: see *https://recbioengineering.com/*.
- Workbook-based organization of subjects, trials, probe identifiers, and electrode arrangements.
- Monopolar and differential visualization, band-pass and notch filtering, amplitude scaling, zoom, and pan.
- Interactive channel interpolation and exclusion decisions, saved to the workbook.
- Inspection and manual correction of synchronization events for each probe.
- Batch synchronization-event detection with per-trial feedback.
- Batch export of recordings and metadata, with a MATLAB CKC reader for use in DEMUSE.

## Downloads and release status

The updated release is being prepared. Published application packages and the illustrated user guide will be available on the [Releases page](https://github.com/taian-vieira/hd-emg-visual-inspector-downloads/releases).

Choose the download for your operating system and check its release notes for supported versions and processor architectures. Standalone Windows and macOS builds have been tested by the developer and colleagues; a complete compatibility matrix has not yet been established.

**Python, Flet, Flutter, and development tools are not required to run the packaged application.**

GitHub also displays automatically generated **Source code** archives. In this repository, these contain documentation rather than the application. Download the named Windows or macOS package instead.

## Installation and first launch

Follow the instructions supplied with the selected release. Windows installers and macOS disk images are planned; their availability will be stated in the release notes.

For portable ZIP packages:

- **Windows:** extract the entire ZIP to a local folder and launch the application `.exe`. Keep all accompanying files and folders together; do not move only the executable.
- **macOS:** extract the ZIP in Finder and copy the complete `.app` bundle to **Applications**, then open it.

After launching, click **Open** and select the workbook in your data directory.

Unsigned packages may trigger operating-system security warnings. If an application cannot be opened, report the exact message and the package version. Do not disable system-wide security protections.

## User guide and example data

Consult the illustrated user guide supplied with the release for the interface and inspection workflow. The sections below describe the workbook, recording layout, shortcuts, and synchronization/export features.

Example data, when provided, are shared separately. The existing example workbook contains a full trial table, but recordings are supplied for **six trials only**. Selecting other rows is expected to report missing recordings. Preserve subject, recording-folder, and file names, and work on a copy of the example workbook.

**Real participant workbooks and raw recordings are not included in this repository or its release packages.**

## Workbook and recordings

Keep participant data in a separate data directory, outside the application installation folder and this repository. The workbook must be inside the directory containing the subject folders:

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

All ten columns below must exist, in any order. Header matching ignores capitalization, leading/trailing whitespace, and repeated whitespace between words. Duplicate headers after normalization are rejected. Extra metadata columns are permitted and displayed in the details panel.

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

`Electrodes Order` fills each display column from top to bottom, then advances to the next column. Channel IDs must exist in the loaded recordings. With two MEACs probes, the first probe supplies channels 1–32 and the second supplies 33–64. With two BAM probes, the first supplies channels 1–64 and the second supplies 65–128. Decision columns use those acquisition-channel IDs, not positions in the reordered display. Keep interpolation and exclusion lists disjoint.

### Supported probes and recording formats

The application supports **MEACs probes with 32 channels each** and **BAM probes with 64 channels each**. Use the original acquisition files and the matching sensor identifiers and electrode arrangement in the workbook.

Support for these probes does not imply support for arbitrary 32- or 64-channel binary files. Other acquisition systems or file formats require specific reader support and validation.

For recordings from several probes, inspect their synchronization events before exporting aligned data. Equal recording lengths alone do not establish synchronization.

## Inspection workflow

1. Click **Open** and choose the workbook.
2. Select a subject and trial from **Load trial**.
3. Inspect the grid using **Processing** and **Visualization**: monopolar or differential signals, optional 10–400 Hz band-pass filtering, notch filtering off/50/60 Hz, amplitude scale, channel numbers, zoom, and pan.
4. In **monopolar** mode, use the channel controls to toggle interpolation or exclusion decisions. Assigning one removes the other for that channel.
5. Click **Save** to write pending decisions to the selected workbook.

Interpolation previews the mean of available surrounding grid electrodes, omitting neighbours marked for interpolation or exclusion. If no eligible neighbour exists, the original signal is retained. Differential signals are calculated between adjacent rows within each grid column.

**Save overwrites the opened workbook** and writes sorted, space-separated channel IDs into the two decision columns for edited trials. Keep a backup and close the workbook in Excel before saving. Save before opening another workbook or closing the application; decisions are not automatically saved. Trial switching within the same workbook retains pending decisions in memory.

Original `.sig` files are never modified. Filtering and interpolation affect the display. The separate batch-export workflow described below writes new recording files and metadata; it does not apply the display filters or interpolation to the exported samples.

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

The required `EMG Sync Events` column listed above stores synchronization information. Events are stored as JSON text, grouped by the probe identifiers used in `Sensors`. For example:

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

## Feedback and support

Report problems through this repository's [Issues page](https://github.com/taian-vieira/hd-emg-visual-inspector-downloads/issues). Include:

- Application version and downloaded package name.
- Operating system and processor architecture, if known.
- Steps that reproduce the problem and the exact error message.
- Probe type and number of probes, when relevant.

Public issues and attachments are visible to everyone. Do not upload participant workbooks, raw recordings, identifying information, or screenshots containing such information. Use synthetic examples where possible.

## Release history

See the [Changelog](CHANGELOG.md) for changes in each version.

**Platform availability:** Version 0.2.0 is currently available for Windows.
The macOS package for this version is being prepared and will be added
to this release once tested.

## Licence

HD-EMG Visual Inspector license

Copyright (C) 2026
LISiN (Laboratorio di Ingegneria del Sistema Neuromuscolare e della Riabilitazione Motoria) - Politecnico di Torino, Torino, Italy
ReC Bioengineering Laboratories S.r.l. - Via Boston 22/7, Turin, Italy
All Rights Reserved.

Unauthorized copying and use of these software, files, documentation and libraries, via any medium is strictly prohibited.
Proprietary and confidential.

The above copyright notice, this list of conditions and the following disclaimer this must be included in all the materials provided with this distribution of the software.

Author: Taian Martins Vieira <taian.martins@polito.it>, November 2026
LISiN (Laboratorio di Ingegneria del Sistema Neuromuscolare e della Riabilitazione Motoria), Politecnico di Torino, Torino, Italy
ReC Bioengineering Laboratories S.r.l. - Via Boston 22/7, Turin, Italy

THIS SOFTWARE IS PROVIDED BY LISiN AND REC BIOENGINEERING LABORATORIES S.r.l. ''AS IS'' AND NO EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL LISiN AND REC BIOENGINEERING LABORATORIES BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
