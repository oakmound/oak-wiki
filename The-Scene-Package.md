To manage scenes and operations between scenes, Oak 2.0 has the `scene` subpackage. It defines function types for building scenes, a map type for managing a set of scenes, and transition operations to manipulate the screen from one scene to the next. 

### Scene Types

A `scene.Scene` is a struct composed of a `scene.Start`, `scene.Loop`, and `scene.End`, where operations on a scene expect the start to function to be called, then the loop function until it returns to stop calling it, and finally the end function. 

```go
// Start is a function taking in a previous scene and a payload
// of data from the previous scene's end.
type Start func(prevScene string, data interface{})

// Loop is a function that returns whether or not the current scene
// should continue to loop.
type Loop func() bool

// End is a function returning the next scene and a SceneResult of 
// input settings for the next scene.
type End func() (string, *Result)
```

The `data interface{}` passed into a `Start` is the same as the `NextSceneInput` supplied in the previous `End`'s `Result:

```go
// A Result is a set of options for what should be passed into the next
// scene and how the next scene should be transitioned to.
type Result struct {
    NextSceneInput interface{}
    Transition
}
```

### Scene Transitions

A `scene.Transition` is a function which manipulates the last visible frame of a scene to transition into the next scene:

```go
type Transition func(*image.RGBA, int) bool
```

At each frame following a scene end, the transition will be passed in the image data of the screen at the end of the last scene and how many frames have passed since the last scene ended. If the transition manipulates this image data, that same manipulated image data will be passed to the next frame of transitioning. Transitions will not stop until they return false, so the next scene will not begin until they return false. While the Transition is ongoing, the next scene's `Start` will run. 

Oak offers a couple of built in Transitions: `Fade` and `Zoom`, both of which are rather simplistic and will operate for a defined number of frames. 


### Scene Maps

Internally when `oak.Add` is called, this forwards to `oak.SceneMap.Add`. So too with `oak.AddScene`. Both of these functions are defined internally on the `scene.Map` type and an alternative to using `Add` and `AddScene` is to create your own `scene.Map` and directly replace `oak.SceneMap`. 

In addition to `Add` and `AddScene`, maps support `Get` and `GetCurrent`, both of which are locking calls around an internal `map[string]Scene`. As Scenes are not pointers, modifying these results will not modify the scene map. As it stands right now attempting to `Add` a scene to a scene map which is already defined by name will fail, but this support will be enabled eventually. 
