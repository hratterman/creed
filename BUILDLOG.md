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

## The round-globe and many-cities pass
Two critiques: the globe's limb wobbled, and every settlement was the same
town. The limb is now a true circle: vertical exaggeration halved (relief
lives in shading, not silhouette) and the ocean sphere rebuilt at high
segment count. Settlements gained three axes of real difference:
- Worldgen now settles the hard country: each culture pushes outposts into
  its deserts, steppes, tundra, and high cold, which the old
  top-habitability placement never touched.
- Building style follows local materials with the culture supplying the
  palette family: adobe and flat roofs in the drylands, dark timber under
  steep slate in the cold, and temperate cultures split into plaster,
  fieldstone, or timber traditions. Dry settlements get rocks and scrub
  instead of canopy trees; tundra gets sparse dark scrub.
- Town plans differ: rings, platted grids with crossing streets, or ribbons
  strung along a curving road, with worn street strips drawn into the
  ground; houses face their street.
- Furniture of rank: capitals fly paired faith banners and hold five market
  stalls, towns get stalls and a roofed well, villages get a well and
  haystacks in the fields.
Verified: deep run, interaction sweep, and determinism all clean after the
worldgen change.

## The carpentry pass
Critique: a house was still a triangle on a rectangle. Houses are now built
things, assembled by a house factory: three archetypes per settlement, each
merged into five material slots (plinth stone, wall, timber frame, dark
leaves and panes, roof) and instanced per slot. The temperate and cold
archetypes are a cottage, a jettied town house whose upper story oversails
the street on visible joist ends, and a longhouse with a lean-to annex and a
woodpile; all carry corner posts, wall plates, diagonal braces, framed
windows with shutters, plank doors under lintels, stone chimneys with caps,
and shingle-course strips with ridge caps and fascia boards on the roofs.
The dryland archetypes are pueblo work: stacked flat-roofed adobe with
parapets, protruding vigas, and ladders to the upper floors. Temples share
the roof dressing (courses, ridge caps, fascia) on every gabled sanctuary.
Capitals favor town houses; villages favor cottages and longhouses.
Verified: zero errors across interaction sweep, deep run, and rite
lifecycles with the new geometry.

## Distribution
The game ships three ways from the repository, all zero-install:
- GitHub Pages via an included Actions workflow (site holds index.html and
  the offline file); enabling Pages with Source: GitHub Actions is the only
  one-time click required in repo settings.
- CREED-offline.html: the same build with Three.js r128 inlined (844 KB),
  committed to the repo so it can be downloaded and double-clicked on macOS
  or Windows with no network at all. Verified headlessly: boots and runs
  from file:// with the browser context fully offline.
- Clone and open index.html (canonical CDN build).
The offline file is generated by build.sh alongside index.html so the two
never drift.

## The full improvement slate
All three tiers of the improvement list, taken in one pass.
- Rite signature moments: sermons raise a plinth and lift the preacher while
  listeners arrive in staggered files; processions carry a gold-roofed
  palanquin across the square; weddings raise a garland arch over the couple
  before an officiant; funerals bear a shrouded bier trailed by mourners;
  idle believers now form pilgrim files that walk to the temple forecourt.
  All props live in a per-rite group and are disposed when the rite ends.
- Faith biography: the inspector tells a faith's life as labeled, clickable
  chapters (founding, schisms, wars, persecutions, endings); each row flies
  the camera to the event or opens the relevant faith.
- The tree grew up: wheel zoom about the cursor, drag pan, era bands behind
  deep time, hover cards with symbol, span, believers, and creed line, and a
  Minor branches chip that hides small dead sects when the canopy passes a
  hundred and twenty faiths.
- Shareable moments: the address is seed plus year (#s=SEED&y=YEAR); opening
  such a link relives the years deterministically with a progress line, then
  rebuilds the record. A Copy moment link button writes the current address.
- Timeline strip: a thin band under the world striped by era, ticked by
  loud events in the color of their faith, with the present as a bright
  bar; hovering names the year and age, clicking flies to the nearest tick.
- Postcards: P or a button renders the frame to a vellum-bordered PNG
  captioned with world, year, and era.
- The regional view speaks doctrine: settlement temple markers now take one
  of five silhouettes (spire, dome, ziggurat, gabled hall, grove) chosen by
  the holding faith's architecture, tinted its color, and scaled by its
  share of the world; tilled field patches gather around every settlement.
- Visible growth in the bubble: the main temple scales with the faith's
  world share, and a capital past six thousand souls raises a towered
  circuit wall with two open gates.
- Reach: pinch zoom on touch devices, touch-action none on the canvas, page
  meta and a gilt favicon, a help overlay on ? or the corner button, and
  shadow and sharpness toggles in the World tab.
Verified: interaction sweep clean; moment link fast-forward to year 400
lands correctly; five marker families assign; timeline draws and answers
the pointer; tree pans, zooms, and filters; the walled capital builds; a
sixty second max-speed soak reached year 11,821 with nine living faiths,
twenty two draw calls, and zero errors.

## The first live-play notes
Four complaints from the first session on the deployed site, four fixes.
- Descent you can aim. Zooming in now pulls the view toward whatever the
  cursor rests on. Below three landing altitudes a pulsing gold ring marks
  the town you will enter, with its name announced; double-clicking any
  town locks it as the target and dives straight down into it, and the
  lock releases if you drag away. Entry no longer randomizes the compass:
  you keep the bearing you arrived with. The cloud veil closes a third
  slower.
- A rounder, truer globe. Land color is now a distance-weighted blend of
  nearby cells, so biomes wash into one another instead of showing hard
  per-vertex triangle edges. The faith overlay was rebuilt from airbrushed
  blobs to a true region map: a one-time pixel-to-cell index renders every
  territory as a crisp shape with real coasts and real borders, packed as
  one 32-bit write per pixel and refreshed every 1.4 seconds.
- Towns stopped rhyming. The square is sized by rank and sometimes bare
  earth in villages; the kerb is not universal; the temple takes its own
  bearing on the plaza rim and faces the center, the minority shrine
  answers from the far side; radial towns can throw one or two hamlet
  lobes off to a side so no two plans read alike.
- Rites became occasions. A visitor sees ordinary life first (first rite
  16 to 34 seconds in), then rites arrive every 50 to 130 seconds instead
  of every 6 to 16, the same kind almost never twice in a row, and the
  chronicle notes fewer of them.
Verified headlessly: locked descent lands on the chosen town, overlay
refresh dropped from 43ms toward single digits with packed writes, four
towns entered without error, twenty seconds at speed 2 clean.
