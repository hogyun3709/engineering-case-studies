# 10. Read the crash before touching the code

**System:** three touch kiosks in a public exhibition hall, running unattended
**Stack:** Flutter (Windows)

## Problem

About a year after launch, the kiosks started failing on site at boot. From the outside it looked like one problem: "the app does not come up".

## Investigation

Before changing any Dart code, I read the app logs and Windows Event Viewer. They showed two unrelated failures:

1. A failed host lookup at boot. The app started before the network was ready.
2. An access violation inside the Visual C++ runtime installed on the machine.

## Fixes

- Replaced a fixed 5 s startup wait with a bounded DNS readiness check.
- Bundled matching runtime libraries with the app instead of relying on what the machine had installed.
- Made temporary video files atomic, so an interrupted write cannot leave a broken file behind.

## Takeaway

Two causes with one symptom. Fixing either one alone would have looked like a partial, flaky fix.
