# Panda Madness — Design

A cute, slightly chaotic panda that lives on your desktop and keeps you company — without becoming yet another full product. Distilled from the [2026-10-02 scoping conversation](conversations/2026-10-02-scoping.md).

## Scope

**v1 is:** a tiny desktop app with a single animated panda. Transparent, always on top, minimal settings, local-only logic. Windows first.

**Out of scope:** backend, AI brain, feeding/stats/needs, shops, accounts, cloud sync.

**Success:** a charming panda (not a stupid one) with enough personality to be worth leaving running.

## Licence ✅

**Apache License 2.0** for the whole repository — code and art pack.

## Platforms

- **Windows** — first target.
- **Ubuntu desktop** and **macOS** — designed for from day one, delivered later.
- **Wayland is the strict case we design around:** no global cursor snooping, no global window placement or window enumeration guaranteed. Anything that needs those goes behind an optional adapter capability.
- **Ubuntu on Wayland:** clients cannot position their own windows or force always-on-top, so neither a moving window nor an overlay works natively. Plan: run SDL through XWayland (X11 video driver) on Ubuntu, where positioning and always-on-top mostly work. To confirm in Phase 5.

## Technology ✅

- **Go + SDL3** (Simple DirectMedia Layer 3). SDL3 provides transparent, borderless, always-on-top windows, shaped/click-through windows and a tray API across the major OSes.
- Go chosen 2026-10-02 and confirmed 2026-10-05 after reviewing C, C++, Rust, C# and Zig/Odin with SDL3: easy cross-compiling, simple single exe, plain-Go engine.
- No Tauri/Wails/web views: transparency and click-through are awkward there.
- Small single executable; no JavaScript, no backend.
- **Go binding for SDL3:** prefer a **cgo-free (purego) binding**, which loads SDL3 as a shared library at runtime. That keeps Go's easy cross-compiling (build the Windows exe from any machine, no C toolchain). Candidates:
  - `github.com/Zyko0/go-sdl3` — idiomatic Go wrapper; can embed SDL3.dll in the exe and unpack it at runtime (true single file).
  - `github.com/JupiterRider/purego-sdl3` — thin, close to the C API.
  - Shortlist to confirm in the Phase 0 spike: coverage of the functions we need (transparent windows, window shape, tray, display info).
- Fallback if a needed function is missing: add it ourselves (purego makes that a few lines), or use cgo bindings.

## Panda size ✅

- **Default: 170 × 170 px** on Dave's 2560 × 1440 monitor at 100 % scaling — big enough to have presence, small enough not to be in the way.
- Size is defined in **logical (scale-independent) pixels**, so at 100 % scaling it is exactly 170 × 170 real pixels, and it keeps the same physical size on high-DPI screens (e.g. 340 × 340 real pixels on a 200 %-scaled 4K display).
- The 170 × 170 box is the **frame canvas** for every pose: standing, flat-out sleeping and horizontal Superman all fit inside it, with a shared foot/anchor point.
- User scaling: a simple size setting (e.g. 50–200 %); default stays 170.

## Tray icon ✅

- v1 has a **system tray icon** (Windows notification area; macOS menu bar; Ubuntu AppIndicator). SDL3 includes a tray API, so this should not need per-OS code — confirm in the spike.
- Menu (first cut): **Show/Hide panda**, **Pause** (panda sits still, no gags), **Size** (small/medium/large), **Start with computer** (toggle, **on by default**), **Quit**.
- Icon: a tiny flat black-and-white panda face, from the same art pack.
- Without a tray icon there is no obvious way to quit a borderless, click-through panda — so this is required, not optional.

## Start with computer ✅

- **On by default** — a desktop companion that doesn't come back after a reboot isn't much of a companion.
- Registered on first run; the tray toggle turns it off. Windows: per-user Run registry key (no admin rights). Ubuntu: `~/.config/autostart` entry. macOS: login item.

## Window strategy ✅

The panda is **mostly stationary**; big screen-crossing moves are rare and short. So use the cheapest window for each situation.

- The engine works purely in **desktop coordinates**. Windows are just viewports onto the desktop; each one draws the panda at `pandaPos − windowPos`, so anything outside a window is naturally clipped. The engine doesn't care which window mode is active.
- The viewport planner returns a **list** of window rectangles; the platform layer shows exactly those.

### 1. Resting and local movement — small look-ahead window (default)

- Size the window to cover where the panda is *going*, not just where it is: panda bounds (170 × 170) + margin, stretched in the direction of travel (e.g. ~2× the panda width ahead of a walk).
- The panda moves smoothly inside it at 60 FPS; the window is **re-placed only when the panda nears its edge** or changes direction.
- At rest (idle, eating, sleeping) the window shrinks to panda + small margin and stays still.

### 2. Big, fast moves — temporary full-screen burst overlay (primary)

For rare screen-crossing sequences (Superman flight, a hard throw, a long fall):

- When the sequence starts, show a **fully transparent, borderless, always-on-top window covering each monitor the path touches** (one per monitor, so each matches its monitor's scaling). The panda animates freely inside.
- When the sequence ends, hide the overlay(s) and drop back to the small window at the panda's resting spot.
- Overlay windows are **pre-created hidden at startup** (one per monitor) so there's no creation delay; recreate on display changes.
- Because it lasts only a second or two, the downsides of a permanent overlay (constant full-screen compositing, the risk of blocking clicks) shrink to a brief window. Click-through outside the panda is still required.
- During drag & throw the mouse is captured by the panda anyway, so blocked clicks don't matter until release.

Watch-outs for the spike:
- **Fullscreen detection:** a borderless topmost window exactly the monitor's size can be treated by Windows as a fullscreen app (taskbar hides, notifications go quiet, display mode quirks). Common workaround: make it 1 px smaller than the monitor, or don't mark it fullscreen.
- **Focus:** showing the overlay must never take keyboard focus from the user's app.
- **Flicker** on show/hide and on the handoff between overlay and small window (draw the first overlay frame before hiding the small window, and vice versa).

### 3. Fallback — relay windows

If the burst overlay misbehaves on a platform: lay a short chain of still windows from a hidden pool (3–4) along the planned path, one segment per monitor, and hand the panda from one to the next. Where windows meet, both draw their part of the panda. Kept as a documented fallback, not built unless needed.

### Click-through and monitors

- The empty parts of every window must let clicks through. The click-through mask follows the panda's pixels (updated with the drawing, 8–12 FPS, and with position). The spike checks the cost per platform (SDL3 shaped windows vs native per-pixel-alpha hit testing), including on a full-monitor overlay.
- No window straddles monitors with different scaling.

## Architecture

```
+---------------------------------------------------+
|  Panda engine (pure Go, platform-agnostic)         |
|  - behaviour state machine (sequences of beats)    |
|  - scheduler: random idles vs reactive events      |
|  - soft 2D physics (gravity, throw, bounce)        |
|  - scene model: screen edges, window rectangles    |
|  - viewport planner: list of window rectangles     |
+-----------------------+---------------------------+
                        | narrow interface
+-----------------------v---------------------------+
|  Platform layer                                    |
|  - SDL3: small window, per-monitor overlays,       |
|    tray icon, transparency, rendering, input       |
|  - per-OS shims for what SDL can't do:            |
|      window rectangles, "new dialog appeared",     |
|      global cursor, click-through hit regions,     |
|      start-with-computer                           |
+---------------------------------------------------+
|  Art pack (replaceable folder of PNGs + manifest)  |
+---------------------------------------------------+
```

- **Engine** never imports OS code. It receives events ("cursor moved", "window appeared", "drag started") and a scene snapshot, and emits a sprite + desktop position each frame, plus the set of viewport rectangles it wants shown.
- **OS adapter** reports capabilities; behaviours that need a missing capability (e.g. window rectangles on Wayland) are simply not scheduled.
- **Art pack** is data, not code: swap the folder to restyle the panda without touching Go.

## Art direction

- **Thick, clean line art. Flat black and white.** No textures, no shading.
- Bold, readable poses; comedy comes from timing and holds, not frame count.
- **Limited animation:** drawings at 8–12 FPS ("on twos/threes"), while the app renders motion at 60 FPS.
- Art created with AI assistance (Dave is not an artist) — consistency of the character across poses is the main risk.
- Frame budgets per behaviour are in [behaviours.md](behaviours.md); total for v1 roughly 40–60 drawings.

## Art pack format (proposal)

```
art/default/
  manifest.json      # animations: name -> frames, fps, loop, anchor point, hitbox
  tray.png           # tray icon (panda face)
  idle_01.png ...
  walk_01.png ...
```

- PNG with alpha, **340 × 340 px per frame** (2× the 170 logical size): crisp on 200 % displays, scaled down cleanly at 100 %. Thick flat line art survives downscaling well.
- Master artwork kept larger (or as vector/SVG if the AI workflow allows) so the 340 px exports can be regenerated.
- Line weight chosen to still read at 170 px — roughly 6–8 px lines at 340 px.
- One consistent canvas size and a foot anchor point per frame so poses line up. _Exact manifest schema to be designed._

## Open questions

1. Which purego SDL3 binding (go-sdl3 vs purego-sdl3) — coverage of window shape, tray and display APIs (Phase 0 spike).
2. Click-through: panda pixels clickable, the rest passes through — cost and support of SDL3 shaped windows on Windows, for both the small window and a full-monitor overlay (Phase 0 spike).
3. Burst overlay: fullscreen detection, focus stealing and flicker on show/hide (Phase 0 spike).
