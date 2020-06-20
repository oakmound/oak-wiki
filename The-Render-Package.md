Oak's `render` subpackage handles all operations related to drawing elements to window. This includes primitive types that can be drawn, loading of files and sprite sheets, and providing handlers to organize draw order. `render` provides subpackages of its own for specific operations on drawable elements and specific subtypes of drawable elements.

### The Renderable Interface

```go
// A Renderable is anything which can
// be drawn at a given draw layer, undrawn,
// and set in a particular position.
//
// Basic Implementing struct: Sprite
type Renderable interface {
    Draw(buff draw.Image)
    DrawOffset(buff draw.Image, xOff, yOff float64)
    GetDims() (int, int)

    Positional
    Layered
    physics.Attachable
}
```

The `Renderable` interface defines all types that can be drawn in oak's draw stack. Oak provides several implementations of this interface that should satisfy all basic use cases, and when creating your own implementation of the interface you can almost always rely on composing one of the built in types to satisfy most of these functions. 

The lesser interfaces making up a `Renderable` provide the ability to set and manipulate 2D position and manipulate an element's draw layer. `physics.Attachable` allows different 2D elements to be attached to one another so that when one moves, so does the other.


### The Modifiable Interface

```go
// A Modifiable is a Renderable that has functions to change its
// underlying image.
type Modifiable interface {
    Renderable
    GetRGBA() *image.RGBA
    Modify(...mod.Mod) Modifiable
    Filter(...mod.Filter)
    Copy() Modifiable
}
```

Most Implementations of `Renderable` that Oak provides also implement `Modifiable`. This interface catches each sort of image manipulation function into the `Modify` and `Filter` functions, the former to change an image and create a new image, and the latter to change an image in place. 

### Renderable Implementations

* **Sprite**:
`Sprite` is the most basic implementation of a `Renderable`, combining a 2D position, a draw layer, and a color buffer. When this is drawn to the window it's color buffer will be set at it's 2D position in the world. As with all `Renderable`s, for draw stack items that respect layers a `Sprite` at a higher layer will be visible on top of a Sprite at a lower layer.
* **Composite**: A `Composite` is equivalent to a list for `Renderable`s. There are two types of `Composite`, the `Composite`, and `CompositeR`, where the former is made up of the `Modifiable` interface and the latter of the `Renderable` interface. Most implementations of `Renderable` also satisfy `Modifiable` but if you are working with `Text` types or using this as a draw stack element you will need to use `CompositeR`. 
* **Switch**: A `Switch` is a Renderable type that stores multiple `Modifiable`s but at one time will only draw one of them, keyed by strings. This is ideal for something like an animating character that changes animation based on input.
* **Sequence**: A `Sequence` is a series of `Modifiable`s that will be drawn in order. Sequences have a frames per second rate for how fast they change `Modifiable`s, and each keeps track of whether they are allowed to be interrupted by other animations. When a `Sequence` reaches its last frame it will trigger the `AnimationEnd` event on its associated CID if it was assigned one.
* **Reverting**: A `Reverting` type is a `Modifiable` extension that can revert sets of modifications and restore old renderable elements.
* **ScrollBox**: A `ScrollBox` type holds a set of `Renderable`s and a scroll rate in the X and Y axes. It's manipulable to define at what positions the drawn `Renderable`s should be looped back around. This can build things like news tickers or scrolling backgrounds. 
* **Polygon**: A `Polygon` is a extension of a `Sprite` which stores point locations and a bounding rectangle so it can be filled with color to create a shape.
* **Text**: `Text` is created through the `render.Font` type and stores any `fmt.Stringer`. This means that a basic `Text` type can store actively changing text like score or a timer. Because of this, `Text` is not a `Modifiable`, but can be converted to one through `ToSprite`. 

### Fonts

_in progress_

### The Draw Stack

_in progress_

### Stackable Implementations

_in progress_

### Draw Polygons

_in progress_

### Loading and Batch Loading

_in progress_

### Other Constructors

_in progress_

### Paint Functions

_in progress_

### Subpackages

_in progress_

