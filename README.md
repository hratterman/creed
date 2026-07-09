# CREED

**A living history of faith in a single file.**

CREED is a procedurally generated documentary about the gods a world invented,
told by the far-future philologist who dug them back up. Open `index.html` and
a planet is forged from a seed: peoples settle it, roads connect them, and the
first small faiths of valley and hearth begin to drift, spread, splinter,
blend, and die across deep time. Everything that happens is written into a
scrolling illuminated record, drawn onto a braiding tree of faiths, and framed
by a cinematic auto-director that flies the camera to the story.

## Running it

Open `index.html` in a browser. That is all: no build step, no server, no
assets. The only dependency is Three.js r128, loaded from cdnjs. The same file
runs as a static page (Netlify or any host) or as a Claude artifact.

- The URL hash (`#s=1234567`) is the seed and the only persistence. Copy the
  share link to hand someone your world.
- "Export the record" and "Export the tree" download the run's spiritual
  history and faith genealogy as text; these are the artifacts of a run.

## Watching it

With zero input it behaves like a documentary; press **C** for Watch mode.

- **Scroll** to descend from orbit, through the continental and regional
  bands, into a temple square where rites are staged at human speed.
- **Click** anything: a faith territory, a settlement, a temple, a believer, a
  pilgrim stream, a branch of the tree. The inspector shows its creed-plate.
- **T** opens the tree of faiths, a dendrogram that splits at schisms and
  braids at syncretisms. **Space** pauses; **1 to 5** set the two clocks from
  human time to deep-time maximum; **H** hides the panels.
- The left drawer holds the rate sliders (drift, zeal, tolerance, schism
  threshold, prophet frequency, prosperity, upheaval, literacy, state power),
  the literalism dial (Sociological / Ambiguous / Literal), the fifteen acts
  of fate (raise a prophet, sow a schism, loose a persecution, reveal a
  sign...), and the map overlays.

## The treatment principle

Every faith in CREED is invented, assembled from a parameter space (the
belief-vector) and rendered with the care of an illuminator. The engine models
the sociology of belief evenhandedly and takes no position on whether any
faith is true; nothing here maps onto or caricatures any real religion.

## Development

The repository ships the single finished artifact (`index.html`) plus
`BUILDLOG.md`, the running log of design decisions and tuning history made
while implementing the spec end to end.
