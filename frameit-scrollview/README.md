# FrameIT ScrollView

The ScrollView of [UFrameIT](https://gl.kwarc.info/FrameIT/UFrameIT), ported from TypeScript to Scala
(Scala.js + [Laminar](https://laminar.dev)/Airstream). It runs in the same browser (Unity's WebView)
as UPL's FrameIT backend and talks to it directly in Scala, i.e. without any JS glue.

Unity is only needed to drag facts into the scroll; everything else (loading levels, holding the
logical world, applying scrolls) happens in UPL. UFrameIT only needs the generated JavaScript file.

## Build

```
sbt frameitScrollView/fastLinkJS     # or fullLinkJS for an optimized build
```

Output: `frameit-scrollview/target/scala-2.13/frameit-scrollview-fastopt/main.js`
(`...-opt/main.js` for `fullLinkJS`).
This single file contains UPL (including the global `FrameIT` object) and the ScrollView.
UPL's own build (`sbt fastLinkJS`, used by the VSCode extension) is unaffected.

## Test without Unity

Serve the folder `frameit-scrollview` with any local web server (not `file://`), e.g.

```
cd frameit-scrollview
python3 -m http.server
```

and open `index.html`. It loads a test level (intercept theorem) and shows its scroll.
- "helper()" fills all slots, "Scroll anwenden" applies the scroll: the result is `CD = 210`.
- "Messungen anmelden" declares the measurements as Unity would; they can then be dragged onto slots.
  Right click clears a slot. Facts of the wrong type are rejected.
- "Level aus .pp-Datei laden" loads the same level from `levels/intercept/intercept.pp`.

## JavaScript API

| function | purpose |
|---|---|
| `loadLevelFromProjectFile(url)` | load a level from a `.pp` file and the files it lists (returns a Promise) |
| `loadLevelFromStrings(pp, files)` | same, with the `.pp` content and an object `path -> file content` |
| `loadLevel(background, schemata, stageInit?)` | load a level from strings |
| `RenderScroll(json?)` | show a scroll (from the argument or from `#Unity-Data-Interface[data-scroll-dynamic]`) |
| `applyScroll()` | apply the current scroll; returns the names of the new facts |
| `declareMeasuredFact(json)` | declare a measurement from Unity in UPL |
| `refreshScrollView()` | recompute the view after the logical world was changed via `FrameIT` |
| `loadDemoLevel()`, `helper()` | testing |

A level can also be loaded on startup with `<body data-level="path/to/level.pp">`.
If the page defines `returnSchemaApplicationResult(json)`, the results of a scroll application are passed to it.

## Files (`src/main/scala/info/kwarc/p/frameit`)

- `Backend.scala`: access to UPL (`FrameIT_Backend`, `FrameITProject`); signals changes of the logical world
- `Types.scala`: facts as references to UPL declarations, MathML rendering, registry of declared facts
- `ApiParser.scala`: parses scrolls and facts from the JSON format of the old backend
- `Scroll.scala`: scroll, slots and results; rendering with Laminar/Airstream; application via UPL
- `DropFacts.scala`: drag and drop, right click, declaring measurements
- `SetScrollContent.scala`: exported functions, level loading (`.pp`/`.p`), start

## Known limitations

- Directories in `.pp` files are not supported (the browser cannot list them).
- In `test/FrameIt/Gameplay_Example`, the level loads and accepts declarations, but lookups fail
  with "declaration clash" for `A` (from `theory Triangle` / `triA` in `background.p`).
  This is in UPL, not in the ScrollView; the intercept level here does not use these declarations.
- `FrameIT.lookupNum` throws "Stage0 is not a name" as long as the logical world of a level
  without `stageInit` is empty (the ScrollView catches this).
