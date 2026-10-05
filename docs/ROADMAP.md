# Panda Madness — Roadmap

Status: **draft v0.1** — for discussion. Nothing here is decided until marked ✅.

A desktop panda widget for Windows, Apple platforms and Ubuntu: a small animated panda that lives on the desktop, reacts to the user, and is fun without being annoying.

## Open questions (answer these first)

1. **"iOS" — iPhone/iPad, or macOS?** A free-roaming desktop widget is a macOS thing; on iOS the equivalent is a WidgetKit home-screen widget (static snapshots, limited animation) or an app. This changes the tech choice a lot.
2. **What does the panda do?** Options: idle/sleep/eat animations; walks along the taskbar/window edges; reacts to mouse (pet, poke, drag-and-drop); reacts to system events (time of day, CPU load, notifications, idle time); speech bubbles; mini-games.
3. **Character / art:** who draws it — hand-drawn sprites, vector, 3D? Style reference? Resolution and frame count budget?
4. **Tech stack preference:** one cross-platform codebase vs native per platform? Any language preference (C/C++, Rust, C#, Python, JS)?
5. **Distribution:** GitHub releases only, or app stores (Microsoft Store, Mac App Store, Snap/Flathub)? Signing/notarisation budget?
6. **Licence and openness:** open source? Licence for code vs art?

## Candidate tech stacks

| Option | Pros | Cons |
| --- | --- | --- |
| **Tauri (Rust + web canvas)** | Small binaries, transparent windows, Win/macOS/Linux, iOS support maturing | Transparent click-through windows need per-OS tweaks; Linux/Wayland quirks |
| **Godot 4** | Great for sprite animation; exports Win/macOS/Linux/iOS; transparent borderless windows | Bigger runtime; desktop-integration features (tray, click-through) are limited |
| **Qt 6 (C++/QML)** | Mature, native feel, good transparency/tray support on all desktops | Licensing considerations; heavier toolchain |
| **Native per platform** | Best integration (WidgetKit on iOS, layered windows on Windows) | Three codebases |

Initial leaning (to discuss): a shared core (animation state machine + assets) with a thin per-platform shell.

## Phases

### Phase 0 — Foundations
- Answer open questions; pick stack.
- Repo layout, licence, `.gitignore`, contribution notes.
- CI build skeleton for each target (note: Metatrash refuses pushes to `.github/workflows/`, so CI files are added on GitHub directly).

### Phase 1 — Panda on screen (MVP)
- Transparent, borderless, always-on-top window with one idle animation.
- Drag to move; position remembered between runs.
- Tray/menu-bar icon: show/hide, quit.
- Runs on Windows first, then Ubuntu, then Apple.

### Phase 2 — Personality
- Animation state machine: idle, sleep, eat bamboo, wave, fall/bounce.
- Mouse interactions: pet, poke, pick up and drop.
- Time-of-day behaviour (sleepy at night).
- Click-through when not hovered.

### Phase 3 — Desktop awareness
- Walk along the taskbar/dock and window edges.
- React to idle time, system load, battery, notifications (opt-in).
- Multi-monitor and DPI scaling.

### Phase 4 — Polish and release
- Settings window (size, behaviours, quiet hours, autostart).
- Installers/packages per platform; signing/notarisation.
- iOS/home-screen variant if in scope.
- v1.0 release.

### Later / ideas
- Multiple pandas, skins/costumes, speech bubbles, mini-games, plugin hooks.

## Platform notes
- **Windows:** layered windows for per-pixel transparency; autostart via registry or Startup folder.
- **Ubuntu:** X11 vs Wayland differ for always-on-top, positioning and click-through; Wayland restricts global window placement. Decide minimum Ubuntu version.
- **macOS:** borderless NSWindow at floating level; notarisation needed for distribution.
- **iOS:** no free-floating desktop overlays; WidgetKit or an in-app experience only.

## Decisions log
_(empty — add ✅ entries as we agree things)_
