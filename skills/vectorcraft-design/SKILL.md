---
name: vectorcraft-design
description: Design principles for producing professional, polished vector illustrations in VectorCraft — composition, grids and the golden ratio, spacing rhythm, visual hierarchy, color theory and harmonious palettes, contrast/accessibility, typography pairing and scale, depth via shadows and glows, consistency, and a final design-critique checklist. Use when the artwork must look good, not merely be geometrically correct. Companion to vectorcraft and vectorcraft-recipes.
license: MIT OR Apache-2.0
metadata:
  version: "1.0.0"
---

# VectorCraft design principles

VectorCraft docs explain *how* to draw; they do not explain *what looks good*. Load this skill whenever aesthetics matter — logos, icons, posters, landing-page art, infographics. The goal: output that a designer would approve, not a pile of correctly-formed primitives.

## 0. Start with a brief, then a plan

Before touching the canvas, answer in one paragraph:
- **Purpose & audience** (app icon? hero graphic? print poster?). This sets style and detail level.
- **Canvas & output** (square for icons, 16:9 for web hero, 3:4 for poster, 1:1 for avatar). Pick explicit dimensions with `file.new`.
- **One focal point.** Exactly one element should win the eye. Everything else supports it.
- **Palette: 3–5 colors max.** One dominant, one secondary, one accent, plus neutrals.
- **Style direction**: flat / flat+shadow / subtle 3D / line-art / duotone / gradient. Keep it consistent across the piece.

Write this plan down (in your reasoning) and stick to it. Random additions are what make agent art look "off".

## 1. Composition

- **Use a grid.** Divide the artboard into thirds (rule of thirds) or a 4/8-column grid. Place the focal point on a third line or intersection, not dead-center (center is acceptable for icons/badges where symmetry is the point).
- **Golden ratio** for natural proportions: a rectangle whose sides are ~1:1.618, or spiral-based placement for organic subjects. Use it for the main shape's bounding box or panel proportions.
- **Balance**: if the focal point sits left, counterweight it with smaller elements or type on the right. Avoid lopsided empty space.
- **Hierarchy of scale**: make the main element clearly larger than secondary elements (e.g. 3:1 size ratio). Similar-sized competing shapes look noisy.
- **Margin / safe area**: keep ~5–10% padding inside the artboard. Nothing important should touch the edge unless cropping is intentional.
- **Alignment**: align related objects to a common edge or centerline. Use Smart Guides mentally — snap x/y coordinates to shared values. Misalignment by a few points reads as sloppy.
- **Odd numbers** (3, 5, 7) compose more naturally than even groups; use for repeated motifs.

## 2. Spacing & rhythm

- **Consistent spacing scale**: pick a base unit (e.g. 8 pt) and use multiples (8, 16, 24, 32, 48, 64). Uniform gaps look intentional.
- **Gestalt proximity**: elements close together read as a group. Use spacing, not boxes, to group.
- **White space is a design element.** Do not fill every gap. If it feels empty, consider enlarging the focal element rather than adding decoration.
- **Rhythm**: repeat a motif or spacing interval to create a visual beat; break the rhythm once at the focal point for emphasis.

## 3. Color

- **60–30–10 rule**: ~60% dominant/background, ~30% secondary, ~10% accent. The accent is what draws the eye — use it sparingly on the focal element or CTA.
- **Harmonious palettes** (pick one and stay in it):
  - *Analogous*: 3 neighboring hues (e.g. blue → teal → green). Calm, cohesive.
  - *Complementary*: two opposite hues (blue/orange). Punchy — desaturate one to avoid vibration.
  - *Split-complementary*: base + the two hues adjacent to its complement. Balanced and vivid.
  - *Monochrome + one accent*: shades of one hue with a single bright accent. Elegant for icons/UI.
- **Think in OKLCH / perceptual terms**: choose colors by **lightness** for hierarchy and keep **chroma** consistent within a family. A palette with matched lightness/chroma steps looks designed; random hexes look arbitrary.
- **Contrast & accessibility**: aim for WCAG contrast ≥ 4.5:1 for normal text, ≥ 3:1 for large text and meaningful graphics. Test dark text on light bg and vice versa. Never rely on color alone to convey meaning — pair with shape, label, or icon.
- **Temperature & mood**: warm (reds/oranges) = energy, urgency; cool (blues/greens) = calm, trust. Match the brief.
- **Gradient tips (VectorCraft-specific)**:
  - Keep gradients subtle (2 stops, hues close together) unless the style calls for bold. Aim the light→dark direction consistently across all objects (light from upper-left is conventional).
  - `angle:-90` puts stop 0 at the top; `angle:90` is bottom-to-top; `0` is left-to-right. Use one gradient across multiple shapes with `span:"selection"` for cohesion.
  - Avoid muddy midpoints: interpolate in a perceptually smooth way (stops between neighboring hues, not across the color wheel).
- **Neutrals**: use a slightly tinted neutral (e.g. `#0f172a` slate, not pure `#000000`) — it reads more refined.

## 4. Typography

- **One or two typefaces, max.** Pair a distinctive display/heading face with a quiet, legible body face. (e.g. a geometric sans for headings + a humanist sans for body).
- **Establish a type scale**: e.g. 12 / 16 / 24 / 36 / 54 pt with a consistent ~1.5 ratio. Reuse these sizes rather than eyeballing each.
- **Hierarchy**: size, weight, and color differentiate levels — not just size. A bold small label can outrank a large light one.
- **Alignment**: left-align body text (easier to read); center only short headings or labels. Keep a single text axis per block.
- **Line length**: 45–75 characters for body copy; use `add_text` area type with `width`/`height` and `fit:"shrinkText"` — then verify it didn't clip.
- **Letter-spacing / tracking**: headings can be slightly tighter (or wider for all-caps labels); body text default. Avoid all-caps for long copy.
- **Fonts may be missing.** If a font silently falls back, the design changes — check export warnings, or choose common fonts (Inter, Source Sans 3, system sans).
- In VectorCraft, `add_text` `x` is the **left edge**, `y` is the **baseline** — convert your layout coordinates accordingly.

## 5. Depth & dimension (use with restraint)

- **Layering**: build background → midground → foreground. Overlap shapes slightly to imply depth.
- **Shadow as depth**: `stylize.dropShadow` with small offset, moderate blur, low-opacity color (`#00000030`). One consistent light source (usually upper-left) across all shadows. Multiple conflicting light directions look wrong.
- **Soft glow / emphasis**: `stylize.outerGlow` (color `#hex + alpha`, e.g. `#38bdf866`) for highlights, neon, or focus. Use on the accent element only.
- **Atmosphere**: a subtle radial-gradient halo behind the focal point, or a faint large circle, adds polish without clutter.
- **Flat design**: skip shadows; differentiate layers with value (lightness) steps and slight hue shifts instead.
- **3D** (`threeD.extrude`) is a strong effect — use on a single hero element, not everywhere.

## 6. Icons & logo specifics

- **Grid & optical centering**: draw on a 24×24 or 32×32 grid with 2px padding. Optically center — circles and squares often need a slight upward/leftward nudge to *look* centered.
- **Consistent stroke weight** across an icon set (e.g. 2 px at 24 px). Use rounded caps/joins for friendly, sharp for technical.
- **Consistent corner radius** family (e.g. 4 / 8 / 12). Mixing random radii looks amateur.
- **Silhouette test**: view at small size (`render_preview max_side:64` or export 32px). A good icon/logo reads as a clear shape, not fine detail.
- **One strong idea.** A logo should be memorable and reducible. Avoid gradients+shadows+bevels on the same mark.
- Build marks with geometric booleans: `pathfinder {operation:"minusFront"}` on overlapping ellipses creates clean crescent/cutout shapes; group for live edits.

## 7. Data visualization / infographics

- Use a sequential or categorical palette with **distinguishable** hues; label directly rather than relying on a distant legend.
- Sort bars by value unless there is a meaningful order. Start axes at zero.
- Emphasize one data point with the accent color; mute the rest.
- Recolor `create_graph` output: it draws black/grey defaults — find series groups with `document.find` and apply `paint.setFill` per group.
- Keep gridlines faint (`#e5e7eb`), labels legible, and chart junk minimal.

## 8. Final design-critique checklist (run before export)

Render the full artboard (`render_preview`) and honestly answer:

- [ ] One clear focal point? Does the eye land where intended?
- [ ] Balanced composition, sensible margins, no accidental collisions/overlaps?
- [ ] Everything aligned to a grid/common edges? Spacing consistent (8-pt multiples)?
- [ ] Palette ≤5 colors, 60–30–10 respected, accents used sparingly?
- [ ] Text legible, correct size scale, no overflow/clipping, fonts present?
- [ ] Contrast accessible (≥4.5:1 text, ≥3:1 graphics)? Meaning not color-only?
- [ ] Depth consistent (one light source, restrained shadows/glows)?
- [ ] Style consistent throughout (stroke weights, corner radii, gradient directions)?
- [ ] Reads well at target size / small-size silhouette test?
- [ ] No stray objects, hidden duplicates, or unlocked background that `select.all` would grab?

If any box fails, fix it **before** exporting. Re-render after each fix.

## 9. Iteration mindset

Agents often stop at "technically done." Push one quality notch further:
1. Render → critique with the checklist.
2. Fix the single biggest weakness (usually spacing or hierarchy).
3. Re-render → repeat 2–3 times.
4. Only then save + export.

Small, deliberate refinements beat one giant regenerating pass.

## References

- `vectorcraft/SKILL.md` — core loop
- `vectorcraft-mcp/SKILL.md` — exact call syntax for effects/paint/type
- `vectorcraft-recipes/SKILL.md` — build patterns
