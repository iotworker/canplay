# CanPlay

[中文](./README.md) | English

CanPlay is a cross-platform desktop tool built with Qt for CAN log analysis, DBC decoding, signal filtering, and visual playback. It is designed for in-vehicle network debugging, offline log investigation, and signal inspection workflows, helping users move quickly from raw CAN frames to readable signals and observable trends.

![CanPlay UI Preview](./demo.png)

## Features

- Import CAN log files and inspect raw frames in a table view
- Load DBC files and decode signals
- Switch between graph view and data view
- Filter and control visibility by signal and CAN ID
- Replay selected signals as curves to inspect changes over time
- Suitable for offline analysis and extensible to real-time capture workflows

## Use Cases

- Analyze CAN logs exported from vehicles or ECUs
- Restore raw frames into readable physical signals with DBC files
- Quickly locate target CAN IDs or signals in large datasets
- Replay signal changes in the graph view to help identify abnormal behavior

## Interface Overview

CanPlay currently provides two main working views:

- `Graph View`: for waveform display, trend inspection, and playback of selected signals
- `Data View`: for browsing raw log frames, decoded results, and filtered data

The left panel manages selected signals, while the main area displays the corresponding curves. This layout is suitable for quickly switching between multiple observation targets during a single analysis session.

## Ubuntu Desktop Installation

The Linux package directory `CanPlay-<version>-linux-x86_64/` includes `install.sh`, which installs the CanPlay executable, desktop launcher, and icon. In practice, you can use a wildcard path to match the current version directory directly.

Runtime note:

- CanPlay currently requires an `X11` session and does not support `Wayland`

The script provides the following:

- Performs a user-level install by default
- Installs the app payload, command entry, and `.desktop` launcher into user or system locations

Example:

```bash
chmod +x ./CanPlay-*-linux-x86_64/install.sh
./CanPlay-*-linux-x86_64/install.sh
```

For a system-wide install:

```bash
sudo ./CanPlay-*-linux-x86_64/install.sh --system
```

Available options:

- `--user`: perform a user-level install (default)
- `--system`: perform a system-wide install into locations such as `/opt/canplay`
- `--help`: show help information

## Project Positioning

CanPlay focuses on desktop CAN data visualization and analysis with the following priorities:

- Built with Qt to keep a consistent experience across platforms such as Linux and Windows
- Optimized for engineering workflows around log import, signal decoding, filtering, and playback
- A lightweight and direct interaction model that reduces the cost of offline CAN data analysis
