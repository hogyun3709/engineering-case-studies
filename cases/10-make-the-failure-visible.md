# 10. Make the failure visible before fixing it

**System:** touch kiosks at a public memorial site in Korea, starting automatically at boot and running unattended
**Stack:** Flutter (Windows)

## Problem

About a year after launch, the kiosks started failing at boot on site: blank screens after loading, missing images, a background video that never played, and on one PC a crash. From the outside it looked like one problem: "the app does not come up".

## Investigation

A kiosk has no console, so I first put the startup log on screen with a copy button. A first round of fixes did not solve it. The on-screen log and Windows Event Viewer then showed separate causes:

1. A failed host lookup at boot. Image downloads started before DNS was ready.
2. An access violation inside the Visual C++ runtime installed on one PC.

A third issue sat behind the video: an interrupted first boot could leave a half-written temporary video file that broke video start on every boot after that.

## Fixes

- Turned a fixed 5 s startup wait into a bounded DNS readiness check: wait at least 5 s, probe the image host every second, and continue after 30 s at most so an offline kiosk does not hang.
- Bundled the runtime libraries with the app instead of relying on what the machine had installed.
- Wrote temporary video files to a staging file and renamed them into place, and retried once with a fresh copy if video start failed.

## Takeaway

Several causes, one symptom. Fixing any one alone would have looked like a partial, flaky fix. Getting the logs off the device is what separated them.
