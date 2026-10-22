# Experimental DD buffer-release correction

This patch targets DroidDeck's Android Wayland compositor, rather than a Mesa or
PanVK driver. The source is pinned to
`Droid-Deck/DroidDeck@9ae771c50d575fa94de93c577e5d8bc653011a68`.
The compositor code is unchanged from the tested `c5b25920` application build.

A buffer shared across surfaces could acquire two queued release events. An
immediate release could also leave an earlier queued release behind. The patch
keeps one queued release per buffer and consumes that queue entry before an
immediate release. Existing displayed-buffer and unknown GPU completion guards
remain in effect.

Buffer trace records now include the Wayland client and reset at attach for every
buffer type. This corrects false duplicate warnings for single-pixel and SHM
buffers. A duplicate warning alone in earlier logs does not prove corruption.

The new queue and trace regression cases each fail twice against the original
production functions and pass with this patch. The focused suites contain 17
tests; three related suites add 11 tests. These tests do not demonstrate that the
phone's black screen is fixed. Device comparison is still required.

The `DD buffer release test module` workflow applies the patch, runs these tests,
and builds the full `droiddeckwayland` target (including the frame-generation
engines) with NDK 27.3.13750724. The artifact includes SHA256 checksums and source
provenance. It is a rooted-device experimental native module, not an installable
APK or a PanVK driver. Back up the existing module and retain app data before
device testing. Reinstalling/updating the app replaces a temporary module change.
