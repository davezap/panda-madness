# Panda Madness — Roadmap

Status: **v0.4** — design decisions settled; remaining questions are answered by the Phase 0 spike. Sources: [2026-10-02 scoping conversation](conversations/2026-10-02-scoping.md) and 2026-10-05 discussion. See [design.md](design.md) and [behaviours.md](behaviours.md).

A cute, slightly chaotic, thick-line-art panda that lives on your desktop. Windows first; Ubuntu and macOS later.

## Decisions log

- ✅ v1 = single animated panda, transparent, always on top, minimal settings, local-only. No backend, AI brain, stats, shops, accounts or sync.
- ✅ Windows first; Ubuntu desktop and macOS later. Design around Wayland's restrictions.
- ✅ Go + SDL3. No Tauri/Wails/web views. (Go reconfirmed 2026-10-05 after reviewing C, C++, Rust, C#, Zig/Odin.)
- ✅ Platform-agnostic engine + thin OS adapter + replaceable art pack folder.
- ✅ Art: thick clean line art, flat black and white, no textures or shading. AI-assisted.
- ✅ Limited animation: drawings at 8–12 FPS, motion at 60 FPS.
- ✅ v1 behaviours: idle/walk/hop, Superman crash, curious climber (windows), eating bamboo, sleeping, startled by dialog, mouse hunt, drag & throw.
- ✅ Desktop icons, taskbar/dock and window contents are post-v1.
- ✅ (2026-10-05) Engine works in desktop coordinates; windows are just viewports. See [design.md § Window strategy](design.md#window-strategy-).
- ✅ (2026-10-05) Default: small look-ahead window that covers where the panda is going; re-placed only near its edge, not every frame.
- ✅ (2026-10-05) Rare big/fast moves: temporary transparent full-screen burst overlay (one per monitor, pre-created hidden), closed when the sequence ends.
- ✅ (2026-10-05) Relay windows kept as a fallback only if the burst overlay misbehaves.
- ✅ (2026-10-05) Ubuntu Wayland: plan to run via XWayland; confirm in Phase 5.
- ✅ (2026-10-05) Panda size: 170 × 170 logical px by default (2560 × 1440 at 100 % scaling); art frames exported at 340 × 340. See [design.md § Panda size](design.md#panda-size-).
- ✅ (2026-10-05) v1 has a tray icon (show/hide, pause, size, start with computer, quit). See [design.md § Tray icon](design.md#tray-icon-).
- ✅ (2026-10-05) Start with computer: **on by default**, tray toggle to turn off.
- ✅ (2026-10-05) Licence: Apache 2.0 for code and art.
- ✅ (2026-10-05) SDL3 from Go via a cgo-free (purego) binding; which one is decided in the spike.

## Open questions

All answered by the Phase 0 spike — see [design.md § Open questions](design.md#open-questions): binding choice, click-through, burst overlay quirks.

## Phases

### Phase 0 — Foundations
- Choose Go SDL3 binding (purego: go-sdl3 or purego-sdl3).
- Throwaway spike on Windows:
  - small transparent, borderless, always-on-top window; move/resize it;
  - click-through on empty pixels, in the small window and in a full-monitor overlay, and its cost;
  - burst overlay: show/hide without flicker or focus stealing, no fullscreen-app detection, clean handoff to/from the small window;
  - SDL3 tray icon with a menu.
- Repo layout (`cmd/`, `engine/`, `platform/`, `art/`), Apache 2.0 `LICENSE`, `.gitignore`.
- Art pack manifest schema.
- CI build (added on GitHub directly — Metatrash won't push `.github/workflows/`).

### Phase 1 — Panda on screen (Windows)
- Transparent always-on-top window; placeholder art.
- Tray icon: show/hide, pause, quit.
- Viewport planner (look-ahead sizing and re-placement).
- Art pack loader + animation player (frames, fps, loop, anchor).
- Idle / walk / hop; scheduler that picks random idles.
- Drag & throw with soft physics (it's also the easiest way to test the panda).

### Phase 2 — Gags
- Burst overlay for big moves.
- Superman crash (screen edges, across monitors).
- Eating bamboo, sleeping.
- Mouse hunt.

### Phase 3 — Desktop awareness (Windows)
- Scene model: screen edges + visible window rectangles.
- Curious climber.
- Startled by dialog (new top-level window / foreground change).

### Phase 4 — Art and v1 release
- Final art pack (~47 drawings at 340 × 340, plus tray icon) replacing placeholders.
- Tray additions: size (50–200 %), start with computer (on by default).
- Windows build + release on GitHub. **v1.0.**

### Phase 5 — Ubuntu and macOS
- Ubuntu: X11, and Wayland via XWayland; behaviours needing window rectangles or the global cursor degrade gracefully where unavailable. Tray via AppIndicator.
- macOS: adapter, menu-bar icon, signing/notarisation.

### Later / ideas
- Desktop icons and taskbar/dock as climbable objects.
- Smarter startle (errors/notifications vs app switches).
- Alternative art packs.
- Relay windows (only if the burst overlay proves unreliable somewhere).
