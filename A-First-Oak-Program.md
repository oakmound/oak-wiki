### Installing Go

If you haven't installed Go on your machine, see Go's [downloads](https://golang.org/dl/) page for instructions on how to do that.

### Starting Template

Once you've got go running on your machine, we'll write our first file which will run oak. In this tutorial we'll call this file `core.go`. 

To start, we'll need a main package with a main function:

```go
package main

func main() {
}
```

If you `go run core.go`, the program will start and immediately exit, because we've not told it to do anything yet.

Oak programs are controlled through scenes, and we'll need an initial scene to run our program. This is done through the `oak.Add` function (or the `oak.AddScene` function). We'll call this scene "game", and `oak.Add` takes the name of the scene as it's first argument:

```go
func main() {
    oak.Add("game", ...)
}
```

Scenes are built of three functions: a start or initialization function, a loop, and an end function. These functions have the following required signatures:

```go
type Start func(prevScene string, data interface{})
type Loop func() bool
type End func() (string, *scene.Result)
```

`oak.Add` takes these three functions in order, so we'll populate it with a basic template:

```go 
func main() {
    oak.Add("game",
        func(prevScene string, inData interface{}) {
        },
        func() bool {
            return true
        },
        func() (string, *scene.Result) {
            return "game", nil
        },
    )
}
```

In the start function, we take in the previous scene and whatever data was passed to us from the previous scene and create our entities we want to exist in this scene. In this case, we do nothing. 

In the loop function, we indicate by returning true that this scene will never exit, but will just keep looping. If we return false here, the end function will get called.

In the end function (although it isn't called) we indicate that the next scene that should occur if this scene ends is "game", this same scene, and we give it no information about how this scene ended. `*scene.Result` can be populated with information about how scenes should transition and given data to pass in to the `inData interface{}` field of the next start function, but we don't do either for this template.

Just doing this, and calling `go run core.go` will still do nothing, as while oak now has a "game" scene, it hasn't been told to run it. 

To run this scene, we add an `oak.Init(scene string)` call to our template:

```go
func main() {
    oak.Add("game",
        func(prevScene string, inData interface{}) {
        },
        func() bool {
            return true
        },
        func() (string, *scene.Result) {
            return "game", nil
        },
    )
    oak.Init("game")
}
```

And now if we run `go run core.go`, we should get this:

![](https://i.imgur.com/XjA8NNk.png)

Because we aren't creating anything to be drawn, the window is appropriately blank.

`oak.Init` is a function that won't return until the engine has stopped (and the window is closed), so nothing after `oak.Init` will get called in our main function, unless we call `Init` in a new goroutine.

### Next Steps

***

You might want to walk through the [Platformer](https://github.com/oakmound/oak/wiki/A-Platformer) tutorial, to get into entity and collision handling, or [Handling Input](https://github.com/oakmound/oak/wiki/Handling-Input) for a more broad overview of how to respond to user input.