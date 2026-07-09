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

## Verification
- Headless Playwright + SwiftShader harness (`test.js` in scratchpad) routes
  the cdnjs three.js URL to a local copy, runs the sim at any speed, samples
  state, screenshots, and fails on any page error. SwiftShader fps is not
  meaningful for the 60 fps target (software GL); perf is judged by draw-call
  and geometry budget: one 20k-tri globe, one overlay shell, merged road
  lines, three Points clouds, two InstancedMeshes at globe scale; the bubble
  is instanced crowds and merged parametric temples. Draw calls stay well
  under 250 at every altitude.
