---
name: verifying-roku-on-device
description: Use when writing, reviewing, or shipping a Roku channel in BrightScript or SceneGraph — especially when the code compiles clean, unit tests pass, and it has not yet run on a physical device. Also use when a channel exits to the Roku home screen with no error, a screen hangs with no log line, posters paint black, a row shows only its first tile, or a feature silently does nothing on hardware while working in tests.
---

# Verifying Roku on Device

**`bsc` and Rooibos prove the pure layer only. The node layer is unexercised until the channel launches on a TV.**

Roku's failure modes are disproportionately *silent*. A wrong field name is dropped without error. A cross-thread object construction returns `invalid`. An oversized texture paints black. None of it reaches a compiler, a unit test, or a careful reading — and every one of them looks, from the outside, like a mild UI quirk.

## Why a careful review isn't enough

Measured, not assumed. Two independent agents reviewed a SceneGraph screen carrying eight planted device-only defects, told only that it compiled clean and had passed two reviews. **Each found the same two and missed the same six.**

The two they found — node comparison with `=`, and `type()` returning a different name by origin — are the two that are well documented publicly. The six they missed are the ones below.

Worse than missing: **both noticed the channel-killing line and downgraded it.** A render-thread `CreateObject("roUrlTransfer")` was reported as a garbage-collection concern, severity Medium, "analytics may be undercounted." It actually returns `invalid` and the next dot-access kills the whole channel.

So the failure isn't inattention. It's that these constructs *look* minor. Correcting severity matters as much as naming the trap.

## The catalog

Each row: what you see in the code → what the device actually does → what a person watching the TV sees.

| Construct | On device | External symptom | Find it |
|---|---|---|---|
| `CreateObject("roUrlTransfer")` in a render-thread function (`init`, an observer callback, a key handler) | MAIN\|TASK-only. Returns `invalid`; next dot-access kills the **channel** | App exits to Roku home mid-use. No dialog, no log line. Looks like a random crash | `grep -rn 'roUrlTransfer' --include=*.bs` then check the caller's thread |
| Setting a field the node doesn't have — e.g. `itemSpacings`, which does not exist on `RowList` | Write is **silently dropped**. No error, no compile signal | Layout is subtly wrong forever. Reads as a design choice | Console only: `Tried to set nonexistent field '...'` |
| `RowList.itemSize` never set (distinct from `rowItemSize`) | Row viewport collapses | **Every row shows only its first tile.** Reads as "the feed only returned one item" | `grep -n 'itemSize\|rowItemSize'` — you need both |
| Full-resolution image URL into a small slot | Exceeds the ~1920×1080 texture budget; **paints black** rather than scaling | Black tiles where posters belong. Reads as missing artwork / a CDN problem | Console: `Loaded texture (W x H) larger than the UI resolution` |
| `a <> invalid and a.field = x` | BrightScript `and`/`or` **do not short-circuit**. `a.field` is evaluated regardless | Crash on precisely the input the guard exists to handle | `grep -nE '(and\|or).*\.' --include=*.bs` — nest the ifs |
| `callFunc` to a name not in the component's `<interface>` | Returns `invalid` **silently** | Screen hangs forever. Identical to a dropped network request | Cross-check every `callFunc("x")` against a `<function name="x"/>` |
| `=` between two `roSGNode`s | Runtime crash | Channel exits | Use `.isSameNode()` |
| `type(n) = "roInt"` | `ParseJSON` yields `"Integer"`; an AA literal yields `"roInteger"`. `"roInt"` matches **neither** | Feature never renders. Nothing logs | Build fixtures with `ParseJSON` on a real payload string, never hand-written literals |

**`RowList` field names are the usual source of the two silent rows above.** `itemSize` is the row's viewport, `rowItemSize` is each tile, and the per-item gap is **`rowItemSpacing`** — a `[x,y]` vector2d, not a bare number. `itemSpacings` is not a field at all. Check any size/spacing name against the SDK rather than inferring it; the cost of a wrong guess is a dropped write and a screen that looks merely badly designed.

**Fixture shape is the deeper lesson in the last row.** Specs built from AA literals assert against a type the code will never receive in production. A suite can be green against data whose shape crosses no wire.

## Two structural traps

**A `.brs` file is copied through verbatim.** No compiler plugin can transform it — AST edits are made and discarded on write. This voids plugin behavior silently and looks exactly like "the plugin does nothing." Rename to `.bs`.

**A `.bs` file must be in a real scope** — `pkg:/source` (global) or a component `<script>` tag — before its diagnostics mean anything. A file no component references is validated in a scope where the builtins aren't visible, so a green gate on an unreferenced file proves very little. **Nothing counts as linted until it is reachable.**

## The device loop

```
make clean            # ALWAYS. Staging is not cleared between builds:
                      # a deleted file gets repackaged and still runs
nc <device-ip> 8085   # START THIS FIRST — boot errors only appear here
curl -d '' "http://<device-ip>:8060/launch/dev"
```

ECP on `:8060` needs no auth for `query`/`launch`/`keypress`. Sideload and screenshot (`/plugin_install`, `/plugin_inspect`) use HTTP digest auth, realm `rokudev` — the username is fixed, only the password varies per device.

**Read the console, not the screen.** Most of the catalog above announces itself in exactly one place: a console warning nobody was watching for.

## Common mistakes

- **A crashed app leaves a suspended Micro Debugger.** The log looks frozen and your fix looks like it failed. Relaunch and re-capture before concluding anything.
- **Read the `Total:` count, not just `RESULT: Success`.** Under `failFast`, one failure halts the run and every later suite silently never executes. A partial run looks like a pass.
- **Verify a redeploy actually landed** — unzip the package and grep for your change. A deploy that didn't take is indistinguishable from a fix that didn't work.
- **Derive screenshot crop coordinates from layout constants, never by eye.** A guessed crop invents defects that aren't there.
- **The screenshot endpoint only captures your own sideloaded channel.** After deep-linking out, `active-app` proves *which app* is foreground and nothing about what it is showing. Confirming a landing requires a human looking at the TV.
- **Check sibling repos first for any asset or layout problem.** Web/other-TV builds of the same product often already solved it; a port that skipped the fix is more likely than a novel bug.
