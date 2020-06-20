All of the code snippets used in this tutorial have complete versions available in [oak/examples/platformer-tutorial](https://github.com/oakmound/oak/tree/release/2.0.0/examples/platfomer-tutorial).

### Adding Entities

At a low level, entities are any type that get registered to be an Entity with the `oak/event` package, but for the purposes of this tutorial we'll be using the `oak/entities` helper package to construct the elements of our game world.

We'll start with a player character, which for this is just going to be a box that moves around and can jump. To initialize a moving character using the `entities` package, we want to call `entities.NewMoving`:

```go
package main

import (
	"image/color"

	"github.com/oakmound/oak"
	"github.com/oakmound/oak/entities"
	"github.com/oakmound/oak/render"
	"github.com/oakmound/oak/scene"
)

func main() {
    oak.Add("platformer", func(string, interface{}) {

        char := entities.NewMoving(100, 100, 16, 32,
            render.NewColorBox(16, 32, color.RGBA{255, 0, 0, 255}),
            nil, 0, 0)

        render.Draw(char.R)

    }, func() bool {
        return true
    }, func() (string, *scene.Result) {
        return "platformer", nil
    })

    oak.Init("platformer")
}
```

`NewMoving` takes in a whole bunch of arguments, like most entities constructors. Here, in order, we have `X` position, `Y` position, `Width`, `Height`, `render.Renderable`, `collision.Tree`, `event.CID`, and `Friction`. 

`X`, `Y`, `Width`, and `Height` define where and how big our entity is, and the `Renderable` element describes what should be drawn at the entity's position. Here we make a red box with the same dimensions as the entity. We don't use friction, so we set it to zero, and both `collision.Tree` and `event.CID` are arguments that will default to sensible values if we pass in zero values for them.

Also, we draw the entity by accessing it's `R render.Renderable`, which is just the value we passed in to the constructor. `render.Draw` can do more complex things than we've asked it to do, but we aren't worrying about layering our drawn components yet, so we only give it the renderable itself. 

Running this file will get us a screen like this:

![](https://i.imgur.com/hLJLDlL.png)

### Moving the Character

We'll now get the character to move left and right when we press A or D. To do this, one approach would be to respond to keyboard events, track when A and D are pressed down, then move the character so long as they aren't released. We aren't going to do that because what we'll end up doing is a lot simpler, and that approach can present concurrency problems down the line. 

We're going to bind a function that will get called at the start of every logical frame, binding it to `event.Enter`. In this function we'll check if A or D is pressed, and if they are, we'll move the character appropriately:

```go
// ... after render.Draw(char.R)
char.Speed = physics.NewVector(3, 3)

char.Bind(func(id int, nothing interface{}) int {
    char := event.GetEntity(id).(*entities.Moving)
    // Move left and right with A and D
    if oak.IsDown(key.A) {
        char.ShiftX(-char.Speed.X())
    }
    if oak.IsDown(key.D) {
        char.ShiftX(char.Speed.X())
    }
    return 0
}, event.Enter)
``` 

Our bind call, which we can call on the character because it is composed of an `event.CID`, first retrieves the character based on the ID that it's given through `event.GetEntity`. If it was possible that this ID could not be our character, because this binding wasn't cleared between scenes perhaps, then we could check that that entity was a character before proceeding with assuming that it is with an optional `ok` second return value from that type conversion to `*entities.Moving`.

Technically we don't need to get a character from the `id` in this program, because there is only one character and we already have that character in scope from our initialization function, but for larger programs processing the passed in `id` is a must.

Now if we run the program and press A and D, we'll get something like this:

![](https://i.imgur.com/48CUhup.gif)

### Adding Gravity

To add gravity we just need to keep track of a delta over time for what the character's current falling speed is. If there isn't ground below the character, we increase it, and if there is, then we set it back to zero. 

In order to actually check whether there is ground, we'll create a `entities.Solid` and set it's `collision.Space`'s `Label` to be `Ground`, which we'll declare in our file. Then each frame, we'll have the character check if it's touching ground or not through `collision.HitLabel`. 

```go
//... at the top of the file

const Ground collision.Label = 1

//... In the event.Enter binding

fallspeed := .1
hit := char.HitLabel(Ground)
if hit == nil {
    // Fall if there's no ground
    char.Delta.ShiftY(fallSpeed)
} else {
    char.Delta.SetY(0)
}
// Actually apply gravity
char.ShiftY(char.Delta.Y())

//... In the scene start function

ground := entities.NewSolid(0, 400, 500, 20,
    render.NewColorBox(500, 20, color.RGBA{0, 0, 255, 255}),
    nil, 0)
ground.UpdateLabel(Ground)

render.Draw(ground.R)

```

The end result of this will have the same character, now falling onto some blue ground. You can walk the character off the side of the ground to fall off the screen.


### Jumping

The last thing we need for a basic platformer is adding in jumping. This is as easy as, in our gravity check, if we are touching ground, then if spacebar is down we should add a lot to our Y delta in the upwards direction. Because we already have gravity, this will get us jumping in arcs. 

```go
// ... after the else, when hit != nil
// Jump with Space
if oak.IsDown(key.Spacebar) {
    char.Delta.ShiftY(-char.Speed.Y())
}
```

If we run this, we can jump but we can notice some immediate issues. One: we can also jump if we're stuck inside the ground, if we walk off the right and then back to the left. Two: We aren't flush with the ground when we land. We can apply a couple tweaks to our physics to correct this:

```go
oldY := char.Y()
char.ShiftY(char.Delta.Y())
hit := collision.HitLabel(char.Space, Ground)
// If we've moved in y value this frame and in the last frame,
// we were below what we're trying to hit, we are still falling
if hit != nil && !(oldY != char.Y() && oldY+char.H > hit.Y()) {
    // Correct our y if we started falling into the ground
    char.SetY(hit.Y() - char.H)
    // Stop falling
    char.Delta.SetY(0)
    // Jump with Space when on the ground
    if oak.IsDown(key.Spacebar) {
        char.Delta.ShiftY(-char.Speed.Y())
    }
    aboveGround = true
} else {
    // Fall if there's no ground
    char.Delta.ShiftY(fallSpeed)
}		
```

### Complete Version

Feel free to read through the code and comments on [the complete code](https://github.com/oakmound/oak/blob/release/2.0.0/examples/platfomer-tutorial/6-complete/complete.go), which has additional, relatively simple logic to correct cases like bumping into platforms from below or walking into the side of a platform. It isn't perfect, and I recommend playing around with it to see if you can find ways to improve it.

![](https://i.imgur.com/kElEyM1.gif)


One particular challenge I'd present is to stop the player from shaking back and forth when you are jumping and you walk into a wall. Doing this will require changing what most of the `Set` and `Shift` functions operate on. 

### Next Steps

***

You might want to walk through the [Top Down Shooter](https://github.com/oakmound/oak/wiki/A-Top-Down-Shooter) tutorial, to start working with the viewport, raycasting, and sprites.