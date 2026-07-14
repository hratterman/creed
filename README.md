# CREED

**A living history of faith in a single file.**

CREED is a procedurally generated documentary about the gods a world invented,
told by the far-future philologist who dug them back up. Open `index.html` and
a planet is forged from a seed: peoples settle it, roads connect them, and the
first small faiths of valley and hearth begin to drift, spread, splinter,
blend, and die across deep time. Everything that happens is written into a
scrolling illuminated record, drawn onto a braiding tree of faiths, and framed
by a cinematic auto-director that flies the camera to the story.

## Installing and playing (macOS and Windows)

There is nothing to install. Pick whichever of these suits you:

1. **Play in the browser.** Once GitHub Pages is enabled for this repository
   (Settings → Pages → Source: GitHub Actions, a one-time click; the deploy
   workflow is already included), the game is live at
   `https://hratterman.github.io/creed/` in any modern browser on any OS.
2. **Download one file and double-click it.** Grab
   [`CREED-offline.html`](CREED-offline.html) (use the Download raw file
   button), then open it by double-clicking. It contains everything,
   including the 3D engine, and runs entirely offline. Works identically on
   macOS (Safari, Chrome, Firefox) and Windows (Edge, Chrome, Firefox).
3. **Clone and open.** `git clone` the repository and open `index.html`.
   This variant loads Three.js r128 from cdnjs, so it wants an internet
   connection on first load.

No build step, no server, no dependencies to install in every case. The only
difference between the two HTML files is that the offline one inlines the
Three.js engine; `index.html` is the canonical CDN build.

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
