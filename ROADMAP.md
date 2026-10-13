# Performance Roadmap

## Goal

Keep Jomtien Ride responsive on a wide range of laptops, phones, and browsers. Preserve the pixel art and game feel while reducing stutter, excess CPU/GPU work, memory use, and startup time. Use measured results to guide each change.

## How we will measure progress

Before changing rendering behavior, record a baseline on representative hardware: a recent laptop, an older or integrated-graphics laptop, and a mid-range phone. Try Firefox and Chromium based browsers, with audio both on and off. Record:

- Average and 95th-percentile frame time during a typical ride and a busy scene.
- Long frames and visible stutter during scrolling, collisions, and police/audio activity.
- Time from opening the page to the Start button becoming usable.
- CPU use, memory use, and battery impact during a five-minute ride.

The in-game FPS and debug display are useful for diagnosis, but final comparisons should use browser performance tools as well.

## Milestones

### 1. Establish a performance baseline

- Create a repeatable five-minute ride scenario and note device, browser, resolution, and power mode.
- Add an opt-in performance summary that reports frame-time percentiles and dropped-frame counts without updating the DOM every frame.
- Profile the full frame: simulation, drawing, canvas scaling, and audio mixing.
- Set initial targets: hold 60 FPS on capable devices; remain at or above 30 FPS on the low-end baseline, with no sustained stalls over 100 ms after startup.

**Done when:** results can be compared before and after a change on the same scenarios and devices.

### 2. Remove avoidable work from every frame

- Stop writing debug values to canvas dataset fields every animation frame. Update diagnostics a few times per second or only when the debug panel is open.
- Separate game updates, rendering, diagnostics, and audio work so each can be profiled and throttled independently.
- Avoid recalculating static geometry and repeated values inside draw functions; retain and reuse precomputed scenery where possible.
- Pause rendering and audio analysis when the tab is hidden, then resume cleanly.

**Done when:** profiling shows less main-thread time per frame with no visible or gameplay change.

### 3. Scale rendering to the device budget

- Keep a clear internal render resolution and cap the canvas pixel ratio to avoid excessive high-DPI work.
- Add a small set of quality levels that adjust canvas resolution and optional scenery effects, with an automatic mode based on sustained frame-time measurements.
- Make quality changes gradual and avoid switching levels repeatedly around a threshold.
- Respect reduced-motion and battery-saving settings where browsers expose them.

**Done when:** low-end devices maintain playable frame pacing, while capable devices keep the current visual quality.

### 4. Reduce startup and asset costs

- Measure which embedded images and audio contribute most to download size, parse time, decode time, and memory.
- Optimize oversized assets and decode images before gameplay begins; report asset failures clearly instead of leaving Start disabled indefinitely.
- Keep a simple offline-play option. If assets move out of the HTML, provide a generated single-file build or a local server option so the game remains easy to run.
- Load nonessential audio or effects after the first playable frame when practical.

**Done when:** startup becomes faster and memory use drops without missing art, audio, or offline play.

### 5. Make audio and simulation resilient under load

- Profile traffic audio mixing and limit active voices based on distance and importance.
- Reuse audio nodes where safe; stop and clean them up predictably when traffic leaves range or the game pauses.
- Use a fixed or bounded simulation step so a slow frame does not cause a large jump in movement or collision checks.
- Keep audio work from delaying the next visual frame.

**Done when:** crowded scenes and audio-heavy moments have frame times comparable to quiet scenes and gameplay remains stable after a pause or stall.

### 6. Verify across systems and ship

- Repeat the baseline scenarios on supported browsers and devices after each performance milestone.
- Check keyboard, mouse, and touch controls at each quality level, including resizing and switching tabs.
- Keep a short performance table in release notes with device/browser, frame-time results, and known limits.
- Recheck the single-file download and GitHub hosted version after packaging changes.

**Done when:** targets from milestone 1 are met or any remaining device-specific limits are documented with a useful fallback.

## Priority order

1. Measure first, including lower-end hardware.
2. Remove per-frame diagnostics and other avoidable main-thread work.
3. Add safe adaptive resolution and quality controls.
4. Optimize asset loading and audio based on profiles.
5. Validate and publish measured results.

## Scope guardrails

- Keep the game playable offline and retain the current single-file option.
- Preserve the existing controls and visual style while optimizing.
- Avoid adding a framework or complex build chain unless measurements show it solves a real performance or maintenance problem.
