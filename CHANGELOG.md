# Changelog

Notable changes to HD-EMG Visual Inspector are recorded here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## Unreleased

## 0.2.0 - 2026-11-13

### Added

- Windows installer with a Start menu entry, optional desktop shortcut,
  and uninstaller.
- Support for BAM probes with 64 channels per probe, alongside
  MEACs probes with 32 channels per probe.
- Inspection and manual correction of synchronization events for
  individual probes.
- Batch synchronization-event detection with per-trial feedback.
- Direct trial loading from the batch-processing report.
- Batch export of recordings and metadata for DEMUSE, with a
  generated MATLAB CKC reader.

### Changed

- Updated signal rendering to use images.
- Improved Windows startup performance.
- Expanded documentation for synchronization and batch export.

### Fixed

- Addressed the blank interface encountered on the first Windows launch.