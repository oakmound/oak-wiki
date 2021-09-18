This document details all changes between oak v2.5.0 and oak v3.0.0.

## Overall

- Throughout the project, tests have been expanded, and use of `github.com/stretchr/testify` has been removed.
- Throughout the project, errors returned by packages are more regularly errors from the `oakerr` package.
- Reduced verbosity of all packages. Unnecessary logs have been removed throughout the codebase
- Several functions which previously logged or ignored errors now return them.
- When a type, function, or variable was named `Def$` before, it is now named `Default$`.
- Utilities or types that were unused or trivial to implement have been removed.
- Global state variables have been adjusted so that they are used less often, and specifically less often without specifically invoking them or invoking package level functions instead of one's own created variables. They still exist, as in small examples or prototypes they are convenient, and some closer to the OS level are still always used and unexposed for user manipulation. 
- The Gopkg.lock and Gopkg.toml files, used for [dep](https://github.com/golang/dep), have been removed. Dep has been deprecated and archived.

## New Subpackages

Several of these new packages are former dependencies that were brought into oak. This was motivated by the number of additional steps it took to manage propagating bug fixes from within these dependencies back to oak. By moving them here, we remove needing to cut new releases of the dependencies and needing to update oak's go module files for the new version. 

### window

- New package, handling the interface between `scene` and `oak.Controller`.

### debugtools

- New package, contains common debug helper functions like rendering collision trees and printing mouse position.

### shiny

- New package, a port of from `github.com/oakmound/shiny`.
- Oak programs shouldn't need to directly interact with `shiny` other than to decide which driver to use, if not using the default.
- Darwin, Linux, and Windows default drivers now at approximate feature parity.

### entities/x/shake

- New package split from `oak/shake.go`, can generically shake screens and any entity that can be positioned.

### alg/range

- New package, a port of `github.com/200sc/go-dist`, representing ranges of colors, integers, and floats.

### audio/*

- `github.com/200sc/klangsynthese` has been ported here. 

## Major Changes to Subpackages

### oak

#### Additions

- Added `Config.IdleDrawFrameRate`, controlling how often the screen will redraw when it does not have focus.
- Added the `Window` type, containing old package globals and enabling running multiple oak windows simultaneously.
- Most public functions in the package are now methods on `Window`s, in addition to default `oak.<Method>` helpers, which will redirect to a default window.
- We now propagate the `event.FocusLoss` and `event.FocusGain` events on appropriate window changes.
- Mouse events now trigger both an absolute and `Relative` event, the latter taking into account the viewport position.
- Added `HideCursor`, `ShowNotification`, and `SetTrayIcon` window methods.
- Added `UpdateViewSize`, to enable resizing the rectangle rendered within the client window.
- Added `SkipRNGSeed` to `Config`, a setting which will skip calling `math/rand.Seed` if set. 
- Added `Window.DoBetweenDraws`, which will confine a function to be called in between screen draws. 

#### Adjustments

- `Screen.Scale` is now a `float64` instead of an `int`.
- `SetBinaryPayload(func(...), func(...))` is now `SetFS(fs.FS)`.
- `Config.EventRefreshRate` can now be properly unmarshaled from JSON.
- Config creation and loading has been overhauled, using `NewConfig` and functional option loaders, `FileConfig` and `ReaderConfig`
- The draw loop has been substantially simplified.
- Backgrounds can now be any type that have `GetRGBA() *image.RGBA`, enabling complex or animating backgrounds.
- Init now takes in a variadic set of `ConfigOption`s, so patterns that used to `setup config, then oak.Init(scene)` now will look like `oak.Init(scene, func() {setup config})`.
- Key events now have a `key.Event` payload instead of a `string` payload.
- `Joystick` is now `InputTypeJoystick`.
- `KeyboardMouse` is now `InputTypeKeyboardMouse`.
- `trackInputChanges` now propagates `InputChange` events.
- The engine will more reliably exit with a clean error if it cannot proceed during initialization.

#### Removals

- `Assets` no longer contains an `AssetPath` or a `FontPath`.
- The `Font` config type has been removed. Non-default fonts must be manually loaded outside of a base oak config, with a full asset path. This change emphasizes the built in nature of the default font and the reality of font configuration that the color, dpi, and hinting of a font is not often customizable from an end user's perspective.  
- `Setup...` constants have been removed and folded into the `Config` struct.
- Shiny gestures are no longer supported at the input loop layer, as they were fundamentally unused. This may be revisited.
- Keybindings have been removed as they did not cooperate well with other key refactoring. This will be revisited.
- Language constants have moved to `oakerr`.
- SeedRNG is no longer public, and no longer logs excessively.
- Removed `Add`, preferring `AddScene`.
- Debug console functionality has been moved to `debugstream`.
- `Shake` functionality has moved to `entities/x/shake`.

### audio

#### Adjustments

- The `Cache` type has been introduced to handle file loading and caching. Several loading functions have changed to work with this new type and the `DefaultCache` global variable.
- `DefFont` has been renamed to `DefaultFont`.
- `DefPlay` has been renamed to `DefaultPlay`.
- Fixed a bug in blank audio loading where the provided "blank" audio was invalid PCM.

#### Removals
- The `FontManager` type has been removed. It was not frequently used and can be trivially introduced as a standalone package.
- Channels, signals, and channel managers have been removed. They were tricky to use correctly and designed toward a specific game, not used by any games since.
- `GetSounds` has been removed.

### collision

#### Additions

- Added `PhaseCollisionWithBus`.
- Added `Space.SetZLayer(float64)`.

#### Adjustments

- `DefTree` has been renamed `DefaultTree`.
- `Attach` now takes a `*Tree` argument, and no longer binds to priority -1.
- `PhaseCollision` now takes a single `*Tree` which can be `nil` instead of a variadic set.
- `NewTree` now takes no arguments and cannot error. The old `NewTree` with custom node sizes has been renamed to `NewCustomTree`.
- Fixed a bug where `With` would return `nil` spaces.
- Moved `CallOnHits` to be a method on a `Tree`.
- `ReactiveSpace` can now take in a `Tree`, and its methods are concurrent safe.
- `CID` is now `IDTypeCID`.
- `PID` is now `IDTypePID`.

#### Removals

- `NewEmptyReactiveSpace` has been removed.
- `Space.String()` has been removed.

### dlog

#### Additions

- Log strings used by oak are now exposed as constants in this package.
- `SetOutput` has been added, replacing `FileWrite` and `CreateLogFile`.
- Exposed `DefaultLogger`, replacing `SetLogger`.

#### Adjustments

- `SetDebugFilter` has been renamed `SetFilter`.
- Log filters are now generic functions: `func(string) bool`, instead of picking between regex matching and `strings.Contains`.
- The `Logger` and `FullLogger` interfaces are now combined.
- `SetDebugLevel` has been renamed `SetLogLevel`.

#### Removals

- `CreateLogFile` has been removed.
- `Warn` has been removed. 
- `SetLogger` has been removed.
- `Logger.GetLogLevel` has been removed. Its hard to imagine a scenario where you would want to query the current log level other than to update it to be more or less verbose-- just set the desired verbosity.
- `FileWrite` has been removed.

### event

#### Additions

- Added `TriggerCIDBack`.
- Added the `Empty` helper, for when a `Bindable` does not care about its arguments or return value.

#### Adjustments

- `Enter` events now take an `EnterPayload`, containing total frames, duration since last frame, and proportionally how that duration compares to the expected average tick duration.
- event Buses now take a `CallerMap` reference.
- `Bus.ResolvePending` has been renamed to `Bus.ResolveChanges`.
- The global caller list is now represented by the `CallerMap` type.
- Many methods that used to be optional in a `Handler` are now required.
- `GlobalBind` and `Bind` now take `string, Bindable` as their argument order, formerly `Bindable, string`. We felt `Bind(EventName, func(){})` read better, especially with larger functions.
- `Bindable`s now accept `CID, interface{}` instead of `int, interface{}`.

#### Removals

- Event bindings in the built in bus no longer support priority levels, as they were virtually unused.

### render

#### Additions

- The `Cache` type has been introduced to handle file loading and caching. Several loading functions have changed to work with this new type and the `DefaultCache` global variable.
- `Font`s can now be provided fallback fonts, which they will try to use to render characters that would be rendered as an undefined character
- Added `ToSprite` to `CompositeR`.

#### Adjustments

- `Font`s now more accurately determine which elements of a font are undefined runes, e.g. a kanji character in a roman alphabet font.
- `Font` and `FontGenerator`s have had their interface significantly adjusted to simplify creation and reduce accidental persistence of one font's qualities or pointer to secondary fonts.
- `Font`s are now copied several times as they are passed around, to enforce that the same font is not used to draw two pieces of text at once, which can panic.
- `Font.Generate` can now return an error.
- `Text`s are now drawn from the top left, like all other `Renderable`s.
- Batch loading has been rewritten. Alias json files are no longer supported. Individual file names can now hint to the batch loading system that they are a sheet with a particular sheet size, and this will supercede a hint coming from their containing directory. Directories and files now must end in explicitly %dx%d e.g. `window64x64.png` or `/tiles32x32/` to be caught and automatically cached as sheets.
- The `Polygon` type now uses `floatgeom.Polygon2`.
- `OverlaySprites` takes a slice of pointers instead of structs.
- `ShinyOverwrite` has been renamed to `OverwriteImage`.
- `ShinyDraw` has been renamed to `DrawImage`.
- `Stackable`s have been changed to an interface that can be externally used, by removing private methods.
- `Stackable`s now have a `Clear` method.
- FPS types are now `Renderable`s instead of `Stackable`s.
- `DefFont` is now `DefaultFont`.
- DrawToScreen now accepts a *intgeom.Point2 for the viewport position.
- `NewStrText` is now `NewText`, the old `NewText` now `NewStringerText`.

#### Removals

- `Noisebox` has been removed.
- `Scrollbox` has been removed.
- `DrawOffset` on `Renderable`s has been removed and `Draw` has taken on its arguments (x and y offsets).
- `Font.DrawString` has been privatized.
- `ParseSubSprite` has been removed.
- `LoadSprites` has been removed.
- `SetFontDefaults` has been removed.
- `LoadSpriteAndDraw` has been removed.
- `ShinySet` has been removed.
- `DrawForTime` has been removed (moved to a scene context method).
- Draw polygons have been removed.
- `FontManager` has been removed.
- Deprecated methods have been removed.

## Minor Changes to Subpackages

### alg

- `RemainingWeights` has been renamed to `CumulativeWeights`, and the old `CumulativeWeights` function has been removed.

### alg/floatgeom

- Added the `Polygon2` type.

### entities

- Removed `String` methods from all entities.

### entities/x/btn

- Added `Destroy` to `Btn`.
- `DisallowRevert` is now flipped to `AllowRevert`.
- Added the `Label(collision.Label)` option.

### entities/x/force

- Now accepts `*scene.Context`s.

### entities/x/mods

- Adds the `Lighter` utility to return a lighter version of a color.

### examples

- Many new examples have been added and most existing examples have been adjusted.
- Removed the `cliffracers` example.

### fileutil

- This package has been stripped down to just be an exposed `FS fs.FS`, a couple configuration options, and the `Open`, `ReadFile`, and `ReadDir` functions.

### joystick

- Stick changes can be isolated to only propagate if they exceed a deadzone threshold.

### key

- Added the `State` type, encompassing key down/up/held states.

### mouse

- Added the `Binding` helper, converting a weakly typed `interface{}` payload into a `*Event` type.
- `DefTree` has been renamed to `DefaultTree`.
- Buttons are now strongly typed.
- Mouse bindings now accept a `*Event` instead of a `Event`.
- `Event` now has a `StopPropagation` boolean which, when set, will stop mouse collision spaces beneath the one triggered on from receiving the event. 
- Mouse gestures, unused, have been removed, for now.

### oakerr

- Errors now can be formatted to multiple languages.
- Several redundant error types have been removed.

### physics

- All `Vector` methods now modify `Vector`s in place. (#115)

### render/mod

- `gift.Resampling` is now aliases as the local `Resampling` type.

### render/particle

- Added the `Allocator` type to contain particle allocation requests.
- Allocators can now be stopped.

### scene

- Scene start functions now take in `(*scene.Context)` as their only argument, a new struct containing everything a scene should need to render, bind, modify the window, etc.
- Added `DoAfter`, `DoAfterContext`, and `DrawForTime` to `Context`.

### timing

- `FPSToDuration` has been renamed to `FPSToFrameDelay`.
- Added `FrameDelayToFPS`.
- Removed `DynamicTicker`, the standard library `*time.Ticker` type supports our use case for these since Go 1.15.
- Removed `DoAfter` and `DoAfterContext`, moved to scene package.
