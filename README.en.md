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
- Use `mosquitto` as the MQTT broker for scripting or integration tests
- Use Python's `cantools` for scripted DBC parsing or validation

## Interface Overview

CanPlay currently provides two main working views:

- `Graph View`: for waveform display, trend inspection, and playback of selected signals
- `Data View`: for browsing raw log frames, decoded results, and filtered data

The left panel manages selected signals, while the main area displays the corresponding curves. This layout is suitable for quickly switching between multiple observation targets during a single analysis session.

## MQTT Interface Guide

CanPlay can expose raw live CAN receive/transmit capability over MQTT during a live capture session. This is intended for scripts, test benches, and external control tools.

Prerequisites:

- Start an MQTT broker that CanPlay can reach
- Start live capture in CanPlay first
- Open `Live -> MQTT Connect...` and enter the broker `IP` and `Port`
- `mosquitto` is a convenient broker choice for local testing

After the connection succeeds, CanPlay subscribes to command topics and starts publishing captured CAN traffic in batches. When live capture stops, the MQTT connection is disconnected automatically.

### Topics

- `canplay/rx`: upstream batches of captured CAN frames
- `canplay/tx`: downstream single-frame transmit requests
- `canplay/cfg`: downstream runtime batch configuration updates
- `canplay/ack`: upstream command acknowledgement
- `canplay/st`: upstream runtime status

### Receive Live CAN Data

The `canplay/rx` payload is a JSON object:

```json
{
  "b": 1713752130123000,
  "f": [
    [0, 291, 0, "11223344"],
    [2000, 292, 5, "5566778899AABBCC"]
  ]
}
```

Field definitions:

- `b`: batch base timestamp in microseconds
- `f`: CAN frame list, each entry is `[delta_us, can_id, flags, data_hex]`
- `delta_us`: timestamp delta relative to `b`, in microseconds
- `can_id`: decimal CAN identifier
- `flags`: compact flags, where `1` means extended, `2` means RTR, and `4` means CAN FD
- `data_hex`: hexadecimal payload string without separators

### Send One CAN Frame

Publish a JSON array to `canplay/tx`:

```json
[291, 0, "11223344"]
```

Array format:

```text
[can_id, flags, data_hex]
```

Validation rules:

- Classic CAN payload length must not exceed 8 bytes
- CAN FD payload length must be one of `0-8, 12, 16, 20, 24, 32, 48, 64`
- If `flags` contains `2`, the request is RTR and `data_hex` must be empty

On success, CanPlay publishes this to `canplay/ack`:

```json
[1]
```

On failure, CanPlay publishes this to `canplay/ack`:

```json
[0, "error message"]
```

### Update Batch Publish Settings

Publish a JSON object to `canplay/cfg`:

```json
{
  "i": 50,
  "n": 100,
  "s": 16384
}
```

Field definitions:

- `i`: flush interval in milliseconds
- `n`: maximum frames per batch
- `s`: target maximum payload size per batch in bytes

CanPlay publishes one `canplay/rx` batch when any of these conditions is met:

- the flush interval `i` expires
- the buffered frame count reaches `n`
- the estimated payload size reaches `s`

### Status Topic

The `canplay/st` payload is:

```json
{
  "c": 1,
  "m": 1,
  "df": 0
}
```

Field definitions:

- `c`: live capture running flag, `1` means active
- `m`: MQTT connected flag, `1` means connected and `0` means disconnected
- `df`: number of received frames dropped because of MQTT-side buffer overflow

When MQTT is disconnected explicitly or live capture stops, CanPlay publishes a status update with `m = 0` so external systems can detect the offline state promptly.

## Ubuntu Desktop Installation

The Linux package directory `CanPlay-<version>-linux-x86_64/` includes `install.sh`, which installs the CanPlay executable, desktop launcher, and icon. In practice, you can use a wildcard path to match the current version directory directly.

The script provides the following:

- Performs a user-level install by default
- Installs the app payload, command entry, and `.desktop` launcher into the user directory

Example:

```bash
chmod +x ./CanPlay-*-linux-x86_64/install.sh
./CanPlay-*-linux-x86_64/install.sh
```

Use `--help` to show help information.

## Project Positioning

CanPlay focuses on desktop CAN data visualization and analysis with the following priorities:

- Built with Qt to keep a consistent experience across platforms such as Linux and Windows
- Optimized for engineering workflows around log import, signal decoding, filtering, and playback
- A lightweight and direct interaction model that reduces the cost of offline CAN data analysis
