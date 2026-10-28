# WAV Audio Preflight

An original, standalone inspector for **local PCM16 RIFF/WAVE bytes and per-channel sample measurements**. It helps a recording reviewer hand another person concrete file facts. It does not listen to the file, convert it, repair it or grade its sound.

Open `index.html` in a modern browser. No installation, server, account, dependencies or network connection is required. The page contains its own styles and script; its content security policy blocks network connections and external assets. Reference links only navigate when you choose them.

## Use the actual file workflow

1. Download the labelled synthetic WAV from the page, or choose a supported local file with the normal file control.
2. Inspect the parsed format, frame count and duration. Read each channel independently; a quiet channel is not concealed by a combined average.
3. Expand the chunk structure if its offsets or skipped metadata matter to your handoff.
4. Download the JSON report. It records definitions, units, threshold, full calculated values and interpretation limits alongside the measurements.

The original bytes are never changed. The synthetic download does not silently load a different file into the inspector: select the downloaded file yourself. Its identity is recognized by an exact byte comparison, not by trusting its filename. Other files are labelled user-selected with origin unknown.

## Supported subset and rejection policy

This is deliberately a small byte parser, not a general audio decoder:

- At most **16 MiB = 16,777,216 bytes**, checked against `File.size` before `FileReader` allocation. At most **256 chunks**. The bounded sample loop yields to the browser between groups of 131,072 frames, so reset/new selection can interrupt it.
- Little-endian **RIFF/WAVE**, uncompressed format tag **1**, signed **16-bit** samples, **one or two channels**, integer sample rate **8,000 through 96,000 Hz**, inclusive.
- Exactly one `fmt ` chunk and one nonempty `data` chunk, in either order. The format payload is either 16 bytes or 18 bytes with `cbSize = 0`.
- Declared RIFF length plus 8 equals actual file length: no trailing bytes. Every chunk header, declared payload and odd-length padding byte must fit. The pad byte's value is ignored, not required to be zero.
- Block alignment is channels × 2; byte rate is sample rate × block alignment. Data length must be a positive exact multiple of block alignment. Empty data and partial frames are rejected.
- Other chunks are **opaque**: their outer lengths and padding are checked, but their contents, nested structure and semantic relationships are not validated or decoded. Chunk IDs are shown as hexadecimal bytes. Only `fmt ` and `data` determine the measurements; no 44-byte-header assumption.

RF64, RIFX/big-endian, compressed, float, extensible, other bit depths/channel counts/rates, extended format payloads outside the choices above, and files with more than 256 chunks are **unsupported by this tool**. That label does not mean an otherwise valid file is corrupt. Duplicate required chunks, inconsistent lengths/rates/alignment, missing required chunks and truncation instead receive specific structural errors. Rejection produces no partial report or export.

The file extension and MIME type are picker hints only. Neither establishes support. This tool does not validate waveform content against its claimed recording context.

## Measurements and units

For channel sample integers `x[0] ... x[N-1]`:

| Measurement | Definition |
| --- | --- |
| Frame count | `data_bytes / block_align_bytes`. A frame has one sample per channel. |
| Duration, seconds | `frames / sample_rate_hz` |
| Channel sample count | `N = frames` for each channel |
| Minimum / maximum | Smallest / largest signed stored integer |
| Normalized sample peak | `max(abs(x)) / 32768` |
| Normalized RMS | `sqrt(sum(x*x) / N) / 32768` |
| Exact-zero samples | Count of `x === 0` |
| Near-zero samples | Count of `abs(x) <= 32`, inclusive |
| Endpoint samples | Counts of `x === -32768` and `x === 32767`, separately |

The integer range is −32,768 to +32,767. Under the common divisor of 32,768, the negative endpoint has magnitude 1; the positive endpoint is `32767/32768 = 0.999969482421875`. There is no mixing of channels, average-of-RMS statistic, dBFS conversion or channel-position assumption.

The near-zero threshold 32 is an arbitrary review aid, **not** a silence, speech or intelligibility threshold. Its normalized magnitude is `0.0009765625`. A channel where every sample falls inside this interval may be intentional. Endpoint counts alone do not prove clipping or its cause. RMS is not calibrated loudness or LUFS, and sample peak is not true peak. None of these measures establishes equipment quality, hearing safety, transcript/translation accuracy or production readiness. Ask the recording owner about intended routing, duration and source context.

The UI rounds normalized ratios and seconds to nine decimal places. JSON preserves the calculated JavaScript numbers and integer counts. The 16 MiB bound keeps even the largest possible sum of squared PCM16 integers below `2**53`; that integer sum is exact, while square root and division use floating-point arithmetic. JSON numeric precision is not a claim about the recording's acoustic precision.

## Reproduce a known result

The deterministic download is a **76-byte synthetic file**, not a field recording: 44 bytes of basic headers and 32 data bytes, 8 stereo frames at 8,000 Hz, lasting 0.001 seconds.

```text
Channel 1: 0, 32767, -32768, 16384, -16384, 32, -32, 0
Channel 2: 0, 0, 0, 0, 0, 0, 0, 0
```

Channel 1's sum of squares is `2,684,291,073`. Its normalized peak is 1; its RMS is `sqrt(2684291073 / 8) / 32768 = 0.559010383750126` approximately. It has 2 exact zeros, 4 samples within ±32 inclusive, and one sample at each endpoint. Channel 2 has zero minimum, maximum, peak and RMS, 8 exact zeros, 8 near-zero samples and no endpoints. Each channel has 8 samples, not 16. The page calculates these through the same byte parser used for selected files; expected values here are for independent reproduction.

## Privacy, handoff and lifecycle

No automatic requests, telemetry, cookies, local/session storage, microphone, audio context, recording or playback. Files are read into memory, not sent elsewhere. Source filenames are rendered with `textContent`; non-ASCII/control characters and backslashes are escaped for a stable display. Metadata payloads are not rendered. Downloads use fixed safe names: `synthetic-pcm16-stereo.wav` and `wav-preflight-report.json`.

The report contains the escaped filename, file size, declared supported format, data location, frame/duration facts, per-channel measurements, chunk headers and explicit limits. It contains no audio samples or metadata payloads. Filename and measurements can still be sensitive; review them before sharing. JSON strings are data, not formulas; no spreadsheet CSV is exported or recommended for direct formula evaluation.

Opening the picker clears old results before selection. Canceling leaves no report. A new or rejected file and Reset disable the old export; reset also clears the file control. A generation token rejects older reads/calculations, and an active read is aborted when replaced. The page yields during calculations and checks the token again before displaying results. The view/export share one completed report object. Reload clears all session state. Browser-managed download history is outside this page's control; the original file is never written.

## Context and sources

Original AI-assisted resource by [PewTranslate](https://pewtranslate.com), © 2026 szj2ys. **PewTranslate live translation accounts are closed**; this independent local inspector requires none. [The sound-team setup guide](https://pewtranslate.com/en/guides/set-up-live-translation-for-your-church) provides a physical audio-input/rehearsal checklist, not a WAV specification or proof of a working production translation service.

Primary structure references: Microsoft's [RIFF description](https://learn.microsoft.com/en-us/windows/win32/xaudio2/resource-interchange-file-format--riff-) and [WAVEFORMATEX fields](https://learn.microsoft.com/en-us/windows/win32/api/mmeapi/ns-mmeapi-waveformatex). This limited inspector is not a standards certification. The sample formulas and interpretation limits above are part of this project's declared behavior.

## Distribution and local verification

Public files: `index.html`, `README.md`, `LICENSE` only. The `.verification/` directory is local test evidence and is excluded from publication. It contains synthetic fixtures, real browser downloads, functional checks and desktop/mobile captures; no personal recording is needed or bundled. Independent acceptance and any publication happen separately from development. MIT license; see `LICENSE`.
