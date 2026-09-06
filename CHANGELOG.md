# AURA release notes

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
