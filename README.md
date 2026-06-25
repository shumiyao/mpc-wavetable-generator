# WaveBench Audio

💡 [Online Demo available at github pages](https://shumiyao.github.io/mpc-wavetable-generator/)

Turn your own recordings into [AKAI MPC 3.9](https://www.akaipro.com/)–compatible wavetables — no Serum, no Vital, no intermediate synth required. Drop in audio, shape it, export.

A single self-contained HTML file. No build step, no install, no server — everything runs locally in your browser.

## Why

MPC 3.9 introduced a Wavetable Oscillator, but turning a recording into a usable wavetable usually means routing it through a third-party wavetable editor first just to get the file format right. WaveBench Audio skips that step entirely: load one or more audio files, scan through them to build cycles, and export directly in the folder/file structure MPC expects.

## Features

- **Audio → Wavetable conversion** — turn any recording (voice, field recording, instrument, drum loop, anything) into a multi-frame wavetable
- **Multi-stage morphing** — load several audio files as "stages." The table morphs from one source to the next across its length, with an adjustable crossfade width and a per-stage "weight" controlling how long it stays in focus
- **Zero-crossing snap** — each frame's start point aligns to the nearest zero-crossing to suppress clicks
- **Loop smoothing** — blends each frame's tail back into its head so it loops cleanly at any pitch
- **Live audition** — play the current (uncommitted) table at a chosen pitch, or scan through the whole table in real time, before you export anything
- **MPC-correct export** — generates the exact folder layout and `format.json` that MPC 3.9 expects, bundled into a ZIP ready to copy onto your MPC's drive
- **Settings persistence** — your parameters and stage setup (trim points, weights — not the audio itself) autosave in the browser (localStorage, with a cookie fallback) and restore next time you open the file
- **JSON import/export** — save, share, or version your settings as a plain `.json` file
- Runs entirely client-side. Nothing is uploaded anywhere.

## Getting Started

1. Download `wavebench-audio.html` from this repo
2. Open it in any modern desktop browser (Chrome, Firefox, Safari, Edge)
3. That's it — no dependencies, no build, no server

## Usage

1. **Add a stage** — click "+ Add Stage" and drop in an audio file (WAV / MP3 / OGG / M4A)
2. **Trim** — drag the start/end handles to pick the portion of the recording to use
3. *(optional)* **Add more stages** — load additional files to morph between sources across the table; each stage's "weight" controls how long the table favors it
4. **Set conversion parameters** — samples per cycle, frame count, zero-crossing snap, loop smoothing, stage morph width, sample rate, bit depth
5. **Audition** — use "Play This Cycle" or "Scan Through Table" to hear the result before committing it
6. **Add to Table** — bakes the current settings into a named wavetable and adds it to the export list
7. **Export ZIP** — bundles every table in the list into the MPC folder structure, ready to transfer

## Installing the Result on Your MPC

1. Unzip the exported file
2. Copy the `Oscillators/Wavetables/<YourLibraryName>/` folder onto your MPC's internal drive or SSD, preserving that same path
3. On the MPC, select the Wavetable Oscillator on a track or pad and pick your library from the list

## Technical Notes — MPC 3.9 Wavetable Format

- Wavetables live in `<Drive>/Oscillators/Wavetables/<LibraryName>/`, containing only `.wav` files and a single `format.json` (no subfolders)
- Each `.wav` must be mono PCM, 16/24/32-bit, sample rate between 22,050 Hz and 96,000 Hz
- `format.json` describes the frame geometry for every file in the folder:
  ```json
  {
    "formatInfo": {
      "numSamplesPerSingleCycle": 2048,
      "numSingleCycles": 256
    }
  }
  ```
- `numSamplesPerSingleCycle` (samples per frame) must be between 512 and 16384
- `numSingleCycles` (frame count) must be between 2 and 2048
- A `.wav`'s total sample count must equal `numSamplesPerSingleCycle × numSingleCycles` exactly
- Max 512 files per library folder

WaveBench Audio enforces all of the above and flags out-of-range settings before you export.

## How It Works

- **Per-stage scanning** — for each output frame, the tool samples a short window from the stage's (trimmed) audio at the corresponding position in time
- **Zero-crossing snap** — the window's start is nudged to the nearest rising zero-crossing within a small search radius, which removes most clicking
- **Loop smoothing** — the tail of each frame is crossfaded toward its own first sample, so the last sample matches the first when the frame loops — eliminating the seam click that raw windowing would otherwise leave
- **Stage morphing** — frames are distributed across stages proportional to each stage's weight; near a stage boundary, frames crossfade between the previous stage's end and the next stage's start over an adjustable width

## Browser Support

Requires Web Audio API support for audition/playback — any current version of Chrome, Firefox, Safari, or Edge. Not specifically tested on mobile browsers.

## Known Limitations

- The wavetable format (`format.json` schema, folder layout, size limits) is based on community reverse-engineering of MPC 3.9, not official AKAI documentation, and could change in a future firmware update
- Audio decoding relies on the browser's built-in `decodeAudioData`, so exact supported input formats vary slightly by browser
- Settings autosave uses localStorage/cookies, which may not persist in sandboxed or private-browsing contexts — use JSON export for guaranteed portability
- Loop smoothing and zero-crossing snap reduce, but can't perfectly eliminate, artifacts on highly transient or noisy source material

## Credits

MPC 3.9 wavetable format details were reverse-engineered by the MPC community. This implementation follows the write-up at [dreyandersson.com](https://dreyandersson.com/blog/load-your-own-wavetables-mpc-3-9/).

[Claude.ai](https://claude.ai) did all the work to produce this webapp.

## License

[MIT](https://choosealicense.com/licenses/mit/)

## Contributing

Fork please. No permission or attribution necessary to do anything with this code.
