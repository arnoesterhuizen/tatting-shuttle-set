# Fusion 360 Parameter Reference — Tatting Shuttle Set

Every user parameter in the design, in one place, for populating **MODIFY → Change Parameters**. Keep all parameters in **mm** (Fusion's Parametric Model Maker integration mishandles unitless parameters).

**Favorite (star) four parameters** so MakerWorld exposes them: `shuttle_length`, `shuttle_width`, `layer_height`, `nozzle_width`. Leave everything else un-starred (internal). A favorited parameter's **Comment** becomes the customizer tooltip; an internal parameter's comment is a note for anyone who opens and edits the source file.

---

## User parameters (favorite these)

| Parameter | Value | Unit | Comment |
|---|---|---|---|
| `shuttle_length` | `70` | mm | Overall shuttle length. 57 ≈ Clover, 70 ≈ standard (Aerlit/Boye), 83 ≈ Starlit, 114 ≈ Tatsy. Pick what fits your hand. |
| `shuttle_width` | `21` | mm | Widest point of the shuttle — your grip. The bobbin stays a standard size; this only sizes the shell around it. |
| `layer_height` | `0.12` | mm | Set this to the layer height you'll slice at. Flat parts build to the nearest whole layer count to 1 mm. Use 0.12, 0.16, or 0.20 mm — these sit on clean Z-stepper increments. |
| `nozzle_width` | `0.4` | mm | Set this to your nozzle size. The bobbin shaft walls and the press-fit are built from it. 0.4 mm is the tested default; 0.4–0.6 mm supported. |

### Customizer slider settings (MakerWorld Customize screen → gear icon)

| Parameter | Default | Min | Max | Step |
|---|---|---|---|---|
| `shuttle_length` | 70 | 57 | 114 | 1 |
| `shuttle_width` | 21 | 19 | 25 | 0.5 |
| `layer_height` | 0.12 | 0.12 | 0.20 | 0.04 |
| `nozzle_width` | 0.4 | 0.4 | 0.6 | 0.2 |

---

## Internal parameters (do not favorite)

Comments here are architectural notes for editing the source file, not user tooltips.

### Derived from print settings

| Parameter | Expression | Unit | Comment |
|---|---|---|---|
| `flat_layers` | `floor(1 mm / layer_height)` | — | Helper. Whole number of layers nearest 1 mm without exceeding it (0.12→8, 0.16→6, 0.20→5). `1 mm / layer_height` is length/length, so it evaluates unitless and `floor` accepts it. |
| `flat_thickness` | `max(0.8 mm; flat_layers * layer_height)` | mm | Thickness of all flat (bed-parallel) parts: shuttle blades, bobbin flanges, picot gauges. Targets ~1 mm (0.96–1.00 across the supported layer heights) so the top finishes on a clean layer. The 0.8 mm floor only guards layer heights outside the supported range. **Note:** the `max` argument separator is a **semicolon** in this file's locale (comma-decimal). If your Fusion uses period-decimals, change it to a comma: `max(0.8 mm, flat_layers * layer_height)`. |
| `wall_thickness` | `nozzle_width * 2` | mm | Bobbin shaft wall = exactly 2 perimeter lines (no gap-fill). Also the step size for the press-fit construction below. |

### Bobbin (fixed standard part — independent of the shuttle sliders)

| Parameter | Value / expression | Unit | Comment |
|---|---|---|---|
| `bobbin_flange_dia` | `22` | mm | Bobbin end-disc diameter. Fixed, NOT tied to `shuttle_width`, so the printed bobbin stays interchangeable with a standard commercial bobbin at every shuttle size. Sits proud of the shell below 22 mm width — intended, since the shuttle is open-sided and the proud flange is the grip for swapping the bobbin. |
| `bobbin_height` | `floor(11.2 mm / layer_height) * layer_height` | mm | Flange-to-flange height, targeting the Clover/Aero tatting bobbin (~11.2 mm), layer-snapped so the flanges finish on a clean layer. Keep internal: it drives the end-wedge angle, which also sets the effective shuttle length, so it is not an independent knob. |
| `bore_dia` | `6.1` | mm | Inner hole through the bobbin shaft; the thread winds around the shaft. Matches the standard sewing/tatting bobbin bore (Class 15 ≈ 6.1 mm). The driver for the press-fit and the shuttle lugs. |
| `thread_hole_dia` | `2.0` | mm | Hole in each bobbin flange to pass the thread tail through when starting a wind (as on a sewing bobbin). **Reused** for the ring hole in each picot gauge's bow — editing this changes both. |

### Shuttle shell

| Parameter | Value / expression | Unit | Comment |
|---|---|---|---|
| `lug_dia` | `bore_dia` (= 6.1 mm) | mm | Centre locating lugs, derived to equal `bore_dia`. Chamfered, so a slightly oversize lug still centres the bobbin. The lugs only CENTRE the bobbin; holding friction comes from the blades resisting being bent, so this fit is low-stakes. |
| `lug_height` | `floor(1.5 mm / layer_height) * layer_height` | mm | Lug height (~1.5 mm target), layer-snapped to a clean layer. |
| `wedge_length` | `shuttle_length / 10` | mm | Length of the end-wedge ramp, measured in from each pointed tip along the blade's long axis (same at both ends). Sets the **gluing surface** between the two blades — it does NOT set the curve (that's the wedge angle, driven by `bobbin_height`). **Proportional** (a tenth of the length at each end) so the wedge-to-blade ratio stays constant across the whole 57–114 mm range — this is what fixed the earlier failure at the long end, where a fixed wedge was too small a fraction of a 114 mm blade. Values: 5.7 mm at 57 → 7.0 mm at 70 → 11.4 mm at 114. Clear tip-to-tip span between the wedges is always `shuttle_length × 0.8` (≈46 mm even at the shortest shuttle), comfortably more than the 11.2 mm bobbin. Trade-off unchanged: longer wedge = more glue area but less free blade to flex the bobbin in. |

### Picot gauges

Each gauge has a house-key silhouette: a rounded **bow** (the grip, with a ring hole), a **shoulder** stepping down to the flat **blade**. The five gauges are **separate bodies** — the functional blade *width* is hard-coded per body (2 / 4 / 6 / 8 / 10 mm, giving ~1 / 2 / 3 / 4 / 5 mm picots), so it is NOT a parameter. These shared dimensions are the same across all five:

| Parameter | Value | Unit | Comment |
|---|---|---|---|
| `gauge_blade_length` | `40` | mm | Length of the flat blade (the working tongue the thread wraps over). |
| `gauge_bow_dia` | `20` | mm | Diameter of the rounded bow (the grip end). Its ring hole reuses `thread_hole_dia`. |
| `gauge_shoulder_length` | `15` | mm | Length of the shoulder — the step transitioning from the bow down to the blade. |

> Blade thickness is not listed here — it uses the shared `flat_thickness` rule (~1 mm). Blade *width* is per-body (the five fixed sizes), not a parameter.

### Plate layout

| Parameter | Value | Unit | Comment |
|---|---|---|---|
| `part_spacing` | `5.0` | mm | Gap between the nine parts on the build plate. **Layout only — does not touch any functional geometry.** Position each body relative to its neighbour's edge plus `part_spacing` (not at fixed coordinates), so the plate stays collision-free as the shuttle resizes across the 57–114 mm range. Only requirement: the part *bodies* must not touch at any size. Brims merging is fine — a shared brim just adds bed adhesion. |

---

## Design notes (for future me)

Why the design is built the way it is — the reasoning behind the parameters above, so I don't have to re-derive it when I come back to this.

**The core idea: print flat, curve afterward.** A tatting shuttle is a boat shape, and that curve prints badly — supports or layer lines land right on the thread path. The Hairy Lion model (print flat, then reshape with heat) reminded me that a part doesn't have to leave the bed in its final shape. So the shuttle prints as two flat blades and I glue them into the curve. Everything else follows from keeping the print flat.

**The curve comes from the end wedges, not a radius.** There's no curve parameter. Wedges at the blade ends set an angle, and when the two blades are glued the shuttle pulls into the boat shape. The wedge angle is driven by `bobbin_height`. Consequence to remember: a taller bobbin curves the blades more and *shortens* the effective length, and a much longer `shuttle_length` (e.g. 114 mm) curves proportionally less — check it still reads as a boat at that extreme.

**`shuttle_length` is the flat print length, not the assembled length.** I use `shuttle_length` directly as the flat blade length and let the assembled (curved) shuttle come out marginally shorter point-to-point. The difference is under ~1 mm at the default size (a 70 mm flat blade folding over ~half the bobbin height loses roughly 0.6 mm), which is negligible for a hand tool. I deliberately did *not* add an arc-length derivation to "correct" for it — it's below print tolerance and would add a fragile sketch that could break at the slider extremes. The reference sizes (57 ≈ Clover, 70 ≈ standard, 83 ≈ Starlit, 114 ≈ Tatsy) are flat lengths; treat flat ≈ assembled within ~1 mm.

**The bobbin is fixed, not scaled with the shuttle.** `bobbin_flange_dia`/`bobbin_height`/`bore_dia` are constants, independent of the width/length sliders. I tried two other ways and rejected both: tying width to flange would have killed the grip slider (the whole point of width), and letting the flange track width would have made every bobbin a different, non-standard size. Fixing the bobbin keeps it interchangeable with a real commercial bobbin at any shuttle size, and lets width just shape the grip.

**The proud flange is intentional.** Below 22 mm width the flange stands proud of the shell. That's fine because the shuttle is open-sided — the bobbin drops in from the side, and the proud flange is what you pinch to swap it. There is no "width must exceed flange" rule.

**The press-fit is geometric, with no clearance to tune.** The two shaft walls are drawn as sketch **offsets**, not as fixed-diameter parameters (which is why there's no `nominal_radius` / `outer_shaft_dia` in the table). Starting from the bore, offset outward by `wall_thickness` to get the inner shaft's outer wall; offset outward by `wall_thickness` again for the outer shaft. Both halves land on the same offset circle, so they meet there with no gap and no interference. Because every step is `wall_thickness` (= `nozzle_width * 2`), the fit holds at any nozzle size as long as I print at the nozzle I set. A hand-tuned clearance would be less robust, so there's deliberately no press-fit clearance parameter.

For reference, at a 0.4 mm nozzle (`wall_thickness` = 0.8 mm) the offsets work out to: bore ⌀6.1 → interface ⌀7.7 → outer ⌀9.3.

**The lugs only centre; the blades hold.** The centre lugs match the bore and are chamfered, so they locate the bobbin even if slightly oversize. They don't grip — the holding friction is the glued blades resisting being bent. That's why the lug fit is low-stakes and I didn't fuss over its tolerance.

**Everything vertical snaps to whole layers.** `flat_thickness`, `bobbin_height`, and `lug_height` all use `floor(target / layer_height) * layer_height`, so every bed-parallel face finishes on a clean layer at any supported layer height — no ragged partial top layer. `flat_thickness` targets ~1 mm (replacing an earlier fixed `layer_height * 8`, which went chunky at coarse layers).

**The blades glue face-to-face, no gap.** CA glue bonds the mating faces directly, so there's no glue-gap parameter.

**If I expose more later:** `blade_glue_gap` (~0.08 mm) is the only sensible extra user knob. Do not expose `bobbin_height` (it perturbs the effective length) or any press-fit clearance (the geometry handles it).
