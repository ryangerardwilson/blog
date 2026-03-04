# vlog

CLI for recording Linux desktop videos with webcam picture-in-picture, synced audio, trim UI, and direct publish to X + LinkedIn.

## Requirements

- Linux
- `ffmpeg`
- `ffplay` (for `vlog p -l` and `vlog a`)
- Webcam device (`/dev/video*`)
- Wayland sessions: `wf-recorder`
- X11 sessions: `ffmpeg` with `x11grab` and PulseAudio/PipeWire `pulse` input

## Commands

Start recording:

```bash
vlog r
```

Stop recording, trim, and publish:

```bash
vlog s
```

`vlog s` flow:
- Finalize recording
- Open trim TUI
- Prompt for accompanying post text
- If you type `v`, open `$EDITOR` to compose the post text
- Publish video + post text to X and LinkedIn

Webcam align preview (live):

```bash
vlog a
```

Play most recent recording (detached/background):

```bash
vlog p -l
```

Clear all saved recordings:

```bash
vlog c
```

Version / upgrade / help:

```bash
vlog -v
vlog -u
vlog -h
```

## Output and behavior

- Default output directory: `~/.cache/vlog/recordings`
- Output filename format: `vlog_YYYYMMDD_HHMMSS.mp4`
- Single active recording enforced
- Single CLI instance lock enforced
- Progress messages shown during start/stop/finalization
- Post-stop interactive trim editor is launched in TTY mode

## Config (XDG)

- Config file: `~/.config/vlog/config.json`
- Auto-created on first publish flow if missing.

Default config:

```json
{
  "publish": {
    "x": "x",
    "linkedin": "linkedin"
  }
}
```

Each publish command must accept:
- Arg 1: post text
- Arg 2: video file path

## Trim editor (after `vlog s`)

After recording is saved, a terminal trim UI opens so you can crop precisely by listening:

- `h` / `l`: seek cursor backward/forward by `0.1s`
- `H` / `L`: jump cursor to start/end
- `space`: play/pause from cursor
- `a` (while paused): discard everything before cursor (set left trim edge)
- `e` (while paused): discard everything after cursor (set right trim edge)
- `Enter`: apply trim and continue to publish prompt
- `q`: cancel trim (keep original finalized recording)

The timer display shows current cursor time and selected kept range.

## Recording pipeline

`vlog r` starts two captures:

1. Screen-only capture
  - Wayland: `wf-recorder`
  - X11: `ffmpeg x11grab`
2. Webcam + default audio capture (`pulse` source `default`, single process for A/V sync)

`vlog s` stops both captures, then finalizes into one MP4 by:

- Converting screen to grayscale
- Overlaying webcam at bottom-right edge (no padding)
- Using webcam-capture audio track (keeps webcam/audio sync)
- Applying audio post-processing: noise reduction + deeper/bassier voice EQ
- Web-optimized H.264/AAC output settings
- Opening trim TUI for optional manual crop
- Prompting for publication text
- Publishing to configured targets from `config.json`

## Video style

- Main screen: grayscale
- Webcam overlay: grayscale + boosted contrast/deeper blacks
- Webcam overlay width: 360px (scaled with aspect ratio preserved)
- Overlay position: bottom-right edge

## Audio style

- Input source is generic/default (`pulse` source `default`)
- Final audio is processed to reduce background noise
- Voice tone is shaped to be slightly bassier/deeper
- No automatic clipping/trimming is applied

## Install from releases

Installer script (repo `ryangerardwilson/vlog`):

```bash
curl -fsSL https://raw.githubusercontent.com/ryangerardwilson/vlog/main/install.sh | bash
```

Install a specific release:

```bash
curl -fsSL https://raw.githubusercontent.com/ryangerardwilson/vlog/main/install.sh | bash -s -- --version 0.1.0
```

After install:

```bash
vlog -h
```

## Release automation

GitHub Actions builds Linux x86_64 release artifacts on tags matching `v*`:

- `.github/workflows/release.yml`

## Files

- `main.py`: CLI entrypoint
- `install.sh`: installer/updater
- `.github/workflows/release.yml`: release build and publish workflow
- `.github/scripts/find-python-url.py`: helper for standalone Python URL resolution
