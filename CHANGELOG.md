# AURA release notes

## 3.6.0

Pre-beta robustness pass - a four-front edge-case audit (Link lifecycle,
host/audio, UI abuse, file/data inputs) with the real breakers fixed.

Audio / host:
- NaN/Inf sanitized at the analysis boundary: one broken upstream block
  used to freeze LUFS, spectrum, and stem readings until reload.
- releaseResources() now parks the analysis thread before resetting, so a
  transport stop/unload can't race the gating-block vector (rare crash).
- FFT state resets on sample-rate change (was carrying stale-rate windows).
- Stem peak ballistics no longer scale with host buffer size; smoothed
  EMAs self-heal from a NaN.
- Offline bounces above ~10x realtime drop whole hops instead of splicing
  discontinuous windows; rendered audio was and remains bit-identical.
- Mono main-bus tracks are accepted (folded to both channels) instead of
  AURA hiding from the insert list.

AURA Link:
- A bypassed/frozen/deactivated master no longer holds the election
  forever - the heartbeat is gated on recent audio, so a live master
  takes over within ~3 s.
- Vanished or slot-reused senders mark their stem inactive immediately
  (generation + liveness checked on the drain thread), so Auto-Mix stops
  advising on stale or mis-identified audio - works with the editor closed.
- New senders flush buffered audio on assignment (no replay of old audio).
- All-10-slots-in-use shows an explicit notice instead of a false
  "SENDING"; duplicate same-type senders get distinct labels; a
  user-typed label is never clobbered by auto-fill.

UI / files:
- Fixed a use-after-free when a reference load completed as the editor
  closed; reference results delivered on the message thread.
- Reopening the editor during a Full-Track run no longer desyncs the
  toggle (which could silently restart and wipe the run).
- Unreadable/corrupt/empty dropped files show an explicit error; long
  offline analyses show status; dropping during analysis no longer kills
  a Full-Track run and does nothing.
- 64-bit file lengths (8-hour / high-SR files no longer truncate to
  garbage); mid-file read failures reported instead of analyzed as silence.
- editor_zoom = nan/inf in project state no longer collapses the window.
- UpdateChecker: bounded manifest read (no thread pinned by a stalled or
  10 MB response); prerelease versions sort below their final release;
  over-long release notes truncated.
- Header brand pulse dot actually animates; clip-threshold widgets stay in
  step when the host restores state with Settings open.

Known limitations (documented, not bugs): two projects open in one DAW
process share the Link registry (analyze one song at a time); analysis is
meaningless during an offline bounce.

## 3.5.0

The trust release - stem measurement rebuilt so what AURA shows matches
what your EQ and your ears say.

- Stem spectrum: 4096-point FFT per stem (was 1024 - at that resolution
  a 200 Hz high-pass had ~4 bins below the cut and window leakage drew
  phantom low-frequency energy). Curves now agree with Pro-Q/EQ Eight.
- Stem analysis is TRUE STEREO end to end: per-channel FFTs power-combined
  for the display, and levels measured as louder-channel peak + channel
  power RMS. The old mono fold cancelled side content, under-reading wide
  stems (vocals, big synths) by up to 10 dB - the primary source of
  "turn UP something already blasting" Auto-Mix advice. AURA Link now
  streams stereo; sidechain measurement fixed identically.
- Percussive recommendations use a ~2 s rec-peak (hold + 6 dB/s decay)
  instead of the 30 s display hold that pinned readings to the loudest
  hit of half a minute ago.
- Multi-solo: repeated S presses build a solo group on the spectrum
  (kick + sub + bass to hunt low-end clashes); SHOW ALL restores.
- Glass & Glow visual pass: vignette backdrop, gradient spectrum fill,
  circular Mix Score dial, sharper high-quality logo rendering, embedded
  brand typography (Space Grotesk), breathing status accents.
- Window: free corner-drag resizing (aspect-locked, 40-150%); +/- buttons
  still step presets; first launch opens at 75%.
- Cleanup: Send MIDI Now only appears when a MIDI port exists (with
  setup guidance otherwise); dead MaskingHeatmap component removed.

## 3.4.0

- **Balance mode**: mark a REF stem (typically the kick) and Auto-Mix
  advice becomes relative to it - the anchor is never moved and every
  other stem's target shifts to preserve your gain staging while matching
  the genre's balance. The Auto-Mix panel shows which mode is active.
- **10 stem slots** (up from 8): covers kick, snare, drum bus, sub, bass,
  vocals, instruments, FX, noise bed, and a spare. Slots 9-10 are AURA
  Link only (sidechain buses remain 8 for host compatibility).
- **Feather-light senders**: "Send as" instances skip all analysis (no
  FFT, LUFS, or oversampled true peak) - Relay-class CPU footprint.
- **Single-master election**: exactly one master instance consumes the
  link streams; additional master instances show a notice instead of
  silently competing for the audio (the cause of missing stem spectrum
  curves and unstable levels while setting up many instances).
- Recommendation rows show ANCHOR on the REF stem; new automated tests
  cover the link registry, master election, and balance math.

## 3.3.0

The flagship release: per-stem analysis that actually works everywhere.

- **AURA Link** - stems connect with zero routing, in every DAW. Keep AURA
  on the master bus, add AURA to each stem track, set its mode to
  "Send as: Kick" (Bass, Vocals, ...). The stem appears in the master
  instance instantly - named, typed, live-metered. Most DAWs expose only a
  single sidechain input to plug-ins, which made the old 8-input sidechain
  design nearly impossible to use; Link replaces it with direct
  instance-to-instance streaming inside the session (classic sidechains
  still work where supported and show an SC badge).
- Stem mapping overhaul: choosing a TYPE auto-fills the label (editable),
  rows show live fast-peak levels and LINK/SC badges, and the 30 Hz UI
  refresh no longer reverts dropdown selections mid-click.
- Recommendation transparency: every Auto-Mix row shows the measured level
  and the genre target it was compared against ("peak -6.2 dB > target
  -14 dB"); averaged passes show "Listening..." until a stem has played
  enough for a reliable read; estimates are labeled with a link hint.
- Spectrum comparison: new GENRE TARGET dashed guide (the per-band level
  the mix would sit at when matching the genre curve) and REF DELTA view
  (mix minus reference around a zero line, +/-18 dB).
- Windows text rendering fixed permanently (MSVC /utf-8): the garbled
  em dashes/bullets/ellipses that appeared throughout the interface since
  v1 are gone; remaining UI literals normalized.

## 3.2.0

Deep-audit release: the full codebase was reviewed against the documentation
and brought in line with it.

- Integrated LUFS now uses BS.1770-4 gating (400 ms blocks, -70 LUFS absolute
  and -10 LU relative gates); True Peak is genuinely 4x-oversampled per
  BS.1770-4 Annex 2, catching inter-sample peaks.
- Spectrum peak-hold works (it was pinned at 0 dBFS); analysis and display
  are correct at 48/88.2/96 kHz sessions.
- MIDI fader recommendations transmit correctly from Send MIDI Now and the
  Auto-Mix one-shot; the virtual port name is MixSense AI Control again, and
  the MIDI channel parameter is honored.
- Sidechain buses map to stems by bus (a disconnected sidechain no longer
  shifts later stems onto the wrong channels); mono sidechains supported.
- Deep Field Mode: Escape closes the overlay, the genre radar always includes
  the selected genre (top matches shown alongside), waterfall fills
  left-to-right without rescaling.
- Spectrum band tints add the documented cyan on-target state; the mix-score
  dynamics component uses the genre's crest-factor target.
- Drag and drop a bounced audio file anywhere on the window for a Full Track
  Report; offline results are no longer overwritten by live data.
- Vocal and FX stem types can now be auto-detected; genre switching, stem
  mapping, and AI settings are thread-safe.
- Fixed memory leaks in the Auto-Mix and report panels, several
  crash-on-close paths, and the Settings Apply wiping AI configuration.
- Genre data: seven genres' True Peak ceilings corrected to match the
  published table (hardstyle, ambient, lo-fi, jazz, classical, acoustic,
  film score).

## 3.1.0

- AURA is now published by ACAS Tools.
- Renamed the plug-in binary from MixSenseAI to AURA. Installers remove the
  old MixSenseAI copy so DAWs do not scan both.
- Replaced the automatic startup version check with a manual **Check Updates**
  button in Settings. AURA now contacts nothing until you click.
- Added a Windows setup installer (previously a batch-script install).

## 3.0.0 (beta)

- Live Mix Score — continuous 0-100 score from the stereo mix FFT.
- 40-genre profile library with LUFS targets, spectral curves, and
  gain-staging rules.
- Deep Field Mode — spectrogram waterfall + genre radar overlay.
- Pinned diagnostics cards.
- Full Track Report — offline file analysis without playback.
