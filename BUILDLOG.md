# CREED build log

Running log of decisions made while implementing the spec end to end.

## Architecture
- Single `index.html`, Three.js r128 from cdnjs, no build step. Developed as
  concatenated sections (scratchpad) and emitted as one file; the repo holds
  only the finished artifact.
- Two seeded sfc32 streams as specified: `generation` (planet, peoples,
  founding faiths, per-settlement bubble dressing) and `history` (drift,
  events, prophets). Noise permutations get their own streams derived from the
  same seed so the map never depends on history draws.
- World substrate: 2600 cells on a Fibonacci sphere with 6-nearest adjacency;
  8 to 14 tectonic plates by jittered flood fill; convergence raises ridges.
  Sea level at the 30th elevation percentile (spec: roughly 30 percent ocean).
- Settlements are the belief-pool unit (spec's per-settlement `BeliefPool`).
  Roads are A* over a biome-and-slope cost field; top quartile by traffic are
  arteries; the road graph is the transmission network.

## Decisions (reversible defaults, logged as required)
- Zoom continuity: the globe-to-bubble handoff is a fast crossfade at a fixed
  altitude threshold rather than a true continuous LOD morph. A morph is not
  feasible in budget with one displaced icosphere; the crossfade is smooth,
  quick, and reversible later.
- Deep-time speeds: 0.08 / 0.6 / 30 / 200 years per real second. At the max
  speed the sim steps 4-year quanta with rate scaling.
- Per-community belief-vectors are approximated: each faith has one global
  vector plus per-settlement divergence scalars that accumulate under
  isolation and culture mismatch; crossing the schism threshold instantiates
  the daughter vector by a large drift step. Full per-community vectors would
  not change what the viewer can see and would triple the state.
- Figures are kept forever (ids are array indices; the record refers back to
  them). Trimming the array once broke ruler references; removed.
- Chronicle DOM keeps the last ~250 entries; full history kept in memory
  (capped at 24k events with oldest trimmed) for export.
- The creed-plate speckle uses Math.random: visual grain only, no sim effect.

## Tuning history
- First run: schism churn far too hot (71 faiths by year 323). Fixes:
  divergence gain 0.012 -> 0.005, schism requires parent age > 60y,
  adherents > 1500, and a 70-year per-faith cooldown; syncretism needs 90 to
  170 years of sustained contact; pool inertia added (communities lerp toward
  the computed distribution rather than jumping); cull threshold lowered so
  minorities persist; prophet base rate roughly doubled.

## Later decisions
- Day and night: the globe holds a standing golden hour that tracks the
  camera (a fixed terminator kept flying the viewer into unreadable darkness);
  the true day/night cycle lives in the bubble, where lamps, moonrise, and
  festival bonfires make night worth watching. Eclipses still dim the sun and
  a red moon still tints the sky at both scales.
- Presentation-layer chance (ambient rite choice, director timing, chronicle
  wording) draws from Math.random, never from the history stream, so watching
  the world cannot alter its history. Same seed, same acts, same rates give
  the same event sequence; the wording of entries may vary between runs.
- Divine manifestation (Literal dial) has two roads: a fulfilled apocalyptic
  prophecy, or accumulated devotion (share above 0.42 of all souls and high
  ritual density, purity, martyr fervor). It rewrites the era, doubles the
  faith's pools, disgraces rivals, and renders as a pillar of light over the
  holy city and a colossal figure in the bubble.
- Legitimacy is a divine-mandate meter: state faith at odds with the capital,
  famine, or a disgraced church erode it; below 0.18 the ruler falls and the
  coronation rite is rewritten by morning.
- The optional "Illuminate this entry" button appears only when
  window.claude.complete exists (artifact host) and falls back to a local
  scholarly line on any failure. The static build never shows it.
- HEARTH houses/factions are simplified to per-generation notables shown in
  the settlement inspector, plus named rulers with traits; lineages did not
  earn their complexity against the four pillars.
- Aurora: two slow additive rings over the poles; comets are sprites read as
  omens; the manifested god and festival bonfires are the only point lights.

## Verification
Acceptance results on the finished build (headless Chromium + SwiftShader):
- Phase 0 distinctiveness: five random belief-vectors produced five faiths
  with distinct symbols, temple archetypes (all nine forms observed across
  runs), postures, funerals, creeds, and plates.
- Determinism: same seed twice gives identical worldgen (names, settlements,
  founding faiths) and an identical history event sequence over the common
  prefix; wording of entries is presentation-layer and may vary.
- Five-minute test (speed 2, no input): PASS. Festivals and holy days seen,
  10 foundings, 13 schisms, 4 endings, 10 recorded rites, 6 director holds
  staged in the bubble, faith statuses visibly changing, zero page errors.
- Interaction sweep: all 15 acts, all 8 overlays, tree filters and node
  click-through, both exports, watch mode, hotkeys, and a full zoom round
  trip orbit -> bubble -> orbit, zero page errors.
- Manifestation (Literal dial): PASS; a vast, fervent faith called its god
  into the world, with the pillar-of-light beat covered by the director.
- Deep-time test (30 real minutes at deep-time speed, no input): PASS.
  12,025 years simulated; 33 living faiths at the end; 208 foundings, 250
  schisms, 303 syncretisms, 801 endings, 24 holy wars, 51 persecutions, 55
  martyrs, 66 philosophical foundings, 116 failed prophecies, 12 reforms, 15
  depositions; population stable; no value overflow; zero page errors.
- Stability run (deep-time-max, unattended): PASS. Plurality persisted,
  nothing overflowed, no errors.

All three acceptance tests of §22 pass, along with determinism, the Phase 0
distinctiveness gate, the literal-dial manifestation, and a full interaction
sweep. The build is done by the definition in §23.

- Headless Playwright + SwiftShader harness (`test.js` in scratchpad) routes
  the cdnjs three.js URL to a local copy, runs the sim at any speed, samples
  state, screenshots, and fails on any page error. SwiftShader fps is not
  meaningful for the 60 fps target (software GL); perf is judged by draw-call
  and geometry budget: one 20k-tri globe, one overlay shell, merged road
  lines, three Points clouds, two InstancedMeshes at globe scale; the bubble
  is instanced crowds and merged parametric temples. Draw calls stay well
  under 250 at every altitude.

## The museum-diorama pass
Requested after first screenshots read as bare primitives. Geometry now comes
from a small carver's bench of procedural helpers: lathed (revolved) profiles
for domes on drums, spires, columns with entasis and capitals, finials,
bells, fonts, idols, and lamp posts; beveled extrusions for all masonry;
gable roofs with eaves; noise-displaced blobs for canopies, rocks, and
standing stones; crenellated walls and stepped entries as merged ornament.
People are lathed robed figures with hoods and folded arms. All nine temple
archetypes were rebuilt on these forms. Lighting is a diorama rig: warm key
with PCF soft shadows, cool fill, hemisphere bounce; the globe gains a limb
glow atmosphere shader and the same key/fill/hemisphere trio; houses show lit
windows at night; a CSS vignette ties the frame to the vellum UI. Polygon
budget was never the constraint (the scene still sits well under 30 draw
calls); silhouette was. Verified: all archetypes build without error, full
interaction sweep clean, zoom round trip clean.

## The polygon-richness pass
Second graphics request: assets still read placeholder. Changes: all lathe,
dome, column, finial, and bell profiles roughly doubled in radial segments;
bevel and curve segments raised; canopies and stones to two subdivision
levels; the globe to icosahedron detail 7; bubble ground to 96x96. People
gained a sash and denser heads and hoods. Houses gained doors, ridge beams,
and a pushed-back ring so the square breathes. The plaza is now flagstone
paving drawn per settlement to a canvas (concentric stone courses with a
medallion) inside a carved kerb ring. Box-built sanctuaries (cathedral, hall,
monastery, shrinehouse) get façade dressing: cornice, pilaster strips, and an
arched portal with a dark recess; cathedral buttresses now shoulder the nave.
Banners are cloth (vertex-waved from the pole edge, with a gilded pole
finial), pyres and bonfires give smoke (recycled sprites), and birds circle
the sanctuary by day. Fixed in passing: bevelBox floated every beveled block
one full height above its base (roofs had been hiding it); animist lineages
no longer always build groves (spirit-houses when natureStance is low), so an
animist-dominated world is not a monoculture of clearings. Scene cost after
all of it: roughly 400k triangles and about 31 draw calls, still far inside
the perf floor on hardware GPUs.

## The figures-and-fire pass
Third graphics request: houses overlapped, people read as pawns, the bonfire
was a cone. Houses and trees now place by rejection sampling with keep-out
zones (the square, the sanctuary ground, the minority shrine, each other), so
nothing interpenetrates. People split into two instanced meshes sharing one
matrix stream: the robe (hood, sash, sleeves) tinted by faith, and a skin
mesh (face and folded hands) with per-person skin tone and build; idle
figures sway faintly. Fire is now a made thing: a log tepee in a ring of
stones, three nested additive flame layers that wobble and counter-rotate, a
flickering point light, rising embers among the smoke; bonfires and cremation
pyres share it, while burial faiths keep a mound and headstone instead.

## Archetype variation
Question raised: do all temples look the same? Nine belief-driven archetypes
were already distinct; within an archetype, proportions were identical. Each
faith now carries three variation genes derived from its id (vertical reach,
secondary masses, ornament count) applied across every archetype: tower and
tier heights, dome mass, minaret reach, wall heights, colonnade counts, grove
stone counts. Two cathedral faiths now build recognizably kin but different
sanctuaries. A contact-sheet test renders all nine forms plus kin variants.

## The legit-game pass (both tiers)
Diagnosis: the remaining demo-tells were silence, the hard cut between
worlds, split art direction, unacknowledged interaction, and no first minute.
- Sound, fully synthesized (no assets): altitude-scaled wind through a
  breathing lowpass, crowd murmur in the square, fire crackle, temple bells
  on director holds, an era drone, a quiet chronicle chime (rate-limited),
  descent whoosh, click ticks, ping notes. Wakes on first gesture; the note
  button in the HUD silences it.
- The globe-to-bubble seam is now a continuous descent: a cloud whiteout
  (fast in, slow clear) with camera motion unbroken through it; you fall out
  of the veil steep and high and ease to eye level, and climb out the same
  way. The old fade-to-black is gone.
- Color grade: terrain ramps desaturated and warmed at both scales, faith
  colors pulled toward jewel tones, canopies deepened, bubble sky warmed.
- Interaction acknowledgment: hover ring and pointer cursor over settlements,
  gilt selection ring on inspect, expanding ping rings (with a note) where
  priority events land and wherever an act of fate is placed.
- The first minute: a designed opening; the world resolves and turns beneath
  "The Creeds of <world>" with year and era, HUD hidden, skippable on any
  input.
- Tier two: chronicle grammar roughly doubled (new phrasings for foundings,
  endings, crossings, martyrs, persecutions, omens, holy wars, conversions,
  failed prophecies, blends, festivals, and all rite classes); territory
  fields composite through a single blur with inked border stippling where
  dominant faiths meet (a map, not a spray); the director letterboxes its
  human-time holds and holds them longer.
Verified: zero page errors across intro, transition, deep-speed run, and the
full interaction sweep.
