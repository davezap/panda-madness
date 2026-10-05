# Panda Madness — Behaviours

Each behaviour is a chain of **beats**. Drawings run at 8–12 FPS; motion is rendered at 60 FPS by the engine. Timings are starting points to tune by feel. Source: [2026-10-02 scoping conversation](conversations/2026-10-02-scoping.md).

## Summary

| # | Behaviour | Trigger | Needs from OS | Drawings | v1? |
|---|---|---|---|---|---|
| 0 | Idle / walk / hop | Default | — | idle 2–4, walk 6–8, hop 4–6 | ✅ |
| 1 | Superman crash 🐼➜💥 | Random | Screen bounds | 6–10 | ✅ |
| 2 | Curious climber 🐼🪟 | Random, near a window | Window rectangles | ~12–16 | ✅ windows / ⏳ icons |
| 3 | Eating bamboo 🐼🎋 | Random, occasional | — | 5–7 | ✅ |
| 4 | Sleeping 💤🐼 | Random, rare, long | — | 3–5 | ✅ |
| 5 | Startled by dialog 😳🐼 | New top-level window / alert | Window-appeared events | ~4–6 | ✅ Windows |
| 6 | Mouse hunt 🐼🖱️ | Cursor moving nearby | Cursor position | ~6–8 | ✅ |
| 7 | Drag & throw 🖱️💨🐼 | User click-and-drag | Mouse input on panda | ~5 | ✅ |

Several drawings are shared (e.g. heap/splat, recover, sit), so the real total is lower than the column sum.

## 0. Base locomotion

Idle (2–4), look around, walk (6–8), hop (4–6). Everything else starts and ends here.

## 1. Superman crash

| Step | Beat | Visual | Notes |
|---|---|---|---|
| 1 | **FLY** | 🐼➜ | Superman pose. Smooth 60 FPS screen motion; 2–4 frames. |
| 2 | **BONK** | 🐼💥│ | One exaggerated squash frame against the edge, held 100–150 ms. |
| 3 | **STUNNED** | 🐼│ | Hold 250–400 ms. The pause sells the joke. |
| 4 | **SLIDE** | ↘🐼│ | Slow wall slide 0.8–1.5 s, slight wobble, same pose. |
| 5 | **FLOP** | 💫🐼 | Short drop, splat pose, crumpled ~1 s, then recover. |

Hold the bonk a hair too long.

## 2. Curious climber

| Step | Beat | Visual | Notes |
|---|---|---|---|
| 1 | **NOTICE** | 🐼 👀 🪟 | Spots a nearby window edge or climbable target. |
| 2 | **APPROACH** | 🐼➜🪟 | Walks over, looks up before committing. |
| 3 | **CLIMB** | 🐼↗️🪟 | 4–6 frame scramble loop. |
| 4 | **SLIP** | 😳🐼↘️ | Pause, loses grip, slides backward. |
| 5 | **ROLL** | ↶ 🐼 ↶ | Tumbles backward, 3–4 rolling poses. |
| 6 | **RECOVER** | 💫🐼 | Heap, blink, up as if nothing happened. |

Requires a **scene model**: find nearby objects, classify as ledge/obstacle, pick one. v1: screen edges + visible application windows. Later: desktop icons, taskbar/dock, window contents.

## 3. Eating bamboo

| Step | Beat | Visual | Notes |
|---|---|---|---|
| 1 | **SIT** | 🐼 | Drops into a seated pose. |
| 2 | **GRAB** | 🐼🎋 | Bamboo stalk in both paws. |
| 3 | **BITE** | 🐼🥢 | 2–3 alternating bite poses, looped. |
| 4 | **CHEW** | 🐼✨ | Contented chew/blink. |
| 5 | **DONE** | 🐼🍃 | Drops stalk, pauses, idles or wanders off. |

Event-driven, not constant: every so often, munch for 5–10 s.

## 4. Sleeping

| Step | Beat | Visual | Notes |
|---|---|---|---|
| 1 | **FLOP** | 🐼⬇️ | Suddenly gives up, face-plants. |
| 2 | **OUT COLD** | 💤🐼 | Face down, limbs loose. Held for ages. |
| 3 | **BREATHE** | 🐼〰️ | 2-frame rise/fall; occasional paw or ear twitch. |
| 4 | **WAKE** | 🐼❓ | Slow head lift, puzzled look, stands up. |

Rarer than other behaviours but lasts tens of seconds — the desktop should feel momentarily abandoned.

## 5. Startled by a dialog

| Step | Beat | Visual | Notes |
|---|---|---|---|
| 1 | **CALM** | 🐼 | Idling, eating, sleeping or wandering. |
| 2 | **POPUP** | ⚠️🪟 | New dialog/alert or sudden foreground window. |
| 3 | **STARTLE** | 😳🐼⬆️ | Instant jump, ears up, one-frame squash/stretch. |
| 4 | **RETREAT** | 🐼💨 | Scuttles away or ducks behind the nearest edge. |
| 5 | **PEEK** | 👀🐼 | Cautious look back, then resumes. |

Reactive, interrupts other behaviours. OS adapter emits "window appeared" / "foreground changed"; the engine decides if it's surprising. Windows v1: new top-level windows/dialogs. Later: tell boring app switches apart from error dialogs/notifications.

## 6. Mouse hunt

| Step | Beat | Visual | Notes |
|---|---|---|---|
| 1 | **NOTICE** | 🐼 👀 🖱️ | Spots the moving cursor, freezes. |
| 2 | **STALK** | 🐼…➜ | Crouches, creeps in tiny steps. |
| 3 | **POUNCE** | 🐼💨🖱️ | Full-body leap at the cursor. |
| 4 | **MISS** | 🐼💥 | Slides past, tumbles or lands flat. |
| 5 | **PRETEND** | 🐼😐 | Gets up, acts like nothing happened. |

Note: on Wayland the cursor is only visible over our own window, so the hunt may need to trigger only when the cursor is near/over the panda there.

## 7. Drag & throw

| Step | Beat | Visual | Notes |
|---|---|---|---|
| 1 | **PICK UP** | 🖱️🐼 | Click-hold; goes floppy, doesn't fight. |
| 2 | **DRAG** | 🐼〰️🖱️ | Follows mouse with lag, dangling limbs. |
| 3 | **RELEASE** | 🖱️💨🐼 | Recent mouse velocity → throw velocity, capped. |
| 4 | **FLY** | 🐼↗️↻ | Floaty arc, gentle rotation, limbs trailing. |
| 5 | **BOUNCE** | 🐼↘️〰️ | Soft squash, small bounce/roll — never violent. |
| 6 | **RECOVER** | 🐼😒 | Sits up, mildly offended, shakes off, resumes. |

Physics: position, velocity, gravity, rotation. Clamp launch speed, damp each bounce, always settle quickly. Playful, never mean — the panda always pops back up unfazed.

## Shared drawing list (first cut)

idle ×3 · walk ×6 · hop ×4 · superman ×2 · bonk-squash ×1 · wall-slide ×1 · heap/splat ×1 · climb ×4 · slip ×1 · roll ×3 · sit ×1 · bamboo-hold ×1 · bite ×2 · chew ×1 · sleep ×2 · wake ×1 · startle ×1 · scuttle ×3 · peek ×1 · crouch/stalk ×2 · pounce ×1 · floppy-held ×1 · airborne ×1 · tumble ×1 · offended-sit ×1 → **~47 drawings**.
