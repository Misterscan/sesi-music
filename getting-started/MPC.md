# Native MPC production

Sesi Music uses std/mpc alongside std/audio and std/theory. Requires Sesi
1.9.1 or later. The MPC style preset renders a multi-section arrangement locally.
Existing presets and studio features remain available.

## Verified production API

- Import std/mpc as MPC, std/audio as Audio, std/theory as Music.
- Create MPC.drum_kit("Kit") and MPC.sequence("Song", bars, BPM, "4/4").
- kit.set_pad(number, options) accepts sound, level, pan, tune (semitones),
  and mute_group. Pads are 1–16. Native sounds include kick, snare, clap,
  rimshot, hat, open_hat, tom, crash, ride, perc, and 808.
- seq.add_track("Drums", "drum", kit) creates a drum track; use "melodic"
  for pitched instruments.
- track.step(step, pad, velocity, options) uses absolute zero-based
  sixteenth steps and velocity 1–127. Bar 2 starts at step 16.
- track.pattern(1, "x--- --x- x--- ----", {"start_step": 16, "velocity": 100})
  places a pattern at bar 2. Patterns do not automatically repeat.
- seq.set_swing(56) uses 50–75, with 50 straight. Convert the studio's
  normalized swing value with 50 + swing * 25.
- seq.set_vintage("mpc3000") adds sampler color; "mpc60" and "sp1200"
  are also supported. Omit this call for clean audio.

## Audio and Theory

Music.scale("C3", "minor") derives pitches; Music.bar(bars, BPM) returns
milliseconds. Audio.synth returns base64 WAV, NOT an MPC sample object.
Use Audio.save(path, note, ms, "triangle", options), then
kit.set_pad(number, {"sound": Audio.load(path)}).

For SoundFont instruments use Audio.sf2("sf2/GeneralUser-GS.sf2",
{"instrument": GM_ID}), render a one-shot with
Audio.mix(path, [[instrument("C4", 500)]], "sine"), then load it into a pad.
This requires FluidSynth.

Each melodic pad plays its loaded pitch. Use separate sampled pitches or
pad tune for melody. Supply numeric MIDI note metadata such as {"note": 48}
to track.step for C3. This metadata does not retune the audible sample.

Use track.set_volume, track.set_pan, and track.mute to honor mixer settings.
Track/master effects use track.add_effect or seq.add_effect, for example
track.add_effect("delay", {"time_ms": 240, "feedback": 0.3, "mix": 0.2}).
Matching pad mute_group values let closed hats choke open hats.

MPC.chop(existingWavPath, 16) creates a chopped pad kit.
MPC.sixteen_levels(padConfig, "tune", {"base_tune": -12}) returns pad
configurations to install using kit.set_pad(pad.pad, pad).
Use only sample files that actually exist.

## Arrangement and export

For new studio compositions build one full-length native sequence, then:

```sesi
seq.render_wav("output.wav")
seq.render_midi("output.mid")
```

The MIDI exporter accepts sequences, not multi-sequence song objects.
Do not invoke interactive playback from generated studio code.
MIDI includes notes and timing, not rendered samples or effects.
Keep existing Presets-based compositions in their original workflow when
revising them unless migration is requested.

Run the included three-library example:

```sh
npm run sesi examples/demo.sesi
```

It writes songs/mpc-session.wav, songs/mpc-session.mid, and a source one-shot.
The example has intro, verse, lift, hook, bridge, final hook, and outro
sections.
