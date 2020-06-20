All of the code snippets used in this tutorial have complete versions available in [oak/examples/top-down-shooter-tutorial](https://github.com/oakmound/oak/tree/release/2.0.0/examples/top-down-shooter-tutorial).

### Starting Point

In this tutorial we'll start with already handling input and character movement. See "A Platformer" for more detailed comments on reaching that point. Our starting code looks like this:

```go
package main

import (
    "image/color"

    "github.com/oakmound/oak"
    "github.com/oakmound/oak/collision"
    "github.com/oakmound/oak/entities"
    "github.com/oakmound/oak/event"
    "github.com/oakmound/oak/key"
    "github.com/oakmound/oak/physics"
    "github.com/oakmound/oak/render"
    "github.com/oakmound/oak/scene"
)

const (
    Enemy  collision.Label = 1
    Player collision.Label = 2
)

var (
    playerAlive = true
)

func main() {
    oak.Add("tds", func(string, interface{}) {
        playerAlive = true
        char := entities.NewMoving(100, 100, 32, 32,
            render.NewColorBox(32, 32, color.RGBA{0, 255, 0, 255}),
            nil, 0, 0)

        char.Speed = physics.NewVector(5, 5)
        render.Draw(char.R)

        char.Bind(func(id int, _ interface{}) int {
            char := event.GetEntity(id).(*entities.Moving)
            char.Delta.Zero()
            if oak.IsDown(key.W) {
                char.Delta.ShiftY(-char.Speed.Y())
            }
            if oak.IsDown(key.A) {
                char.Delta.ShiftX(-char.Speed.X())
            }
            if oak.IsDown(key.S) {
                char.Delta.ShiftY(char.Speed.Y())
            }
            if oak.IsDown(key.D) {
                char.Delta.ShiftX(char.Speed.X())
            }
            char.ShiftPos(char.Delta.X(), char.Delta.Y())
            hit := char.HitLabel(Enemy)
            if hit != nil {
                playerAlive = false
            }

            return 0
        }, event.Enter)

    }, func() bool {
        return playerAlive
    }, func() (string, *scene.Result) {
        return "tds", nil
    })
    oak.Init("tds")
}
```

In addition to the scene setup and movement we also have collision detection triggering the `playerAlive` boolean, and thus the game restarting. So, although none exist yet, when the character hits an enemy the game  will reset.


### Shooting

We'll have the player character shoot at things by clicking on the screen where the shot should go. For now, we'll have the shot just be a line we draw indicating where we are firing. To do this, we want to add a binding to the character on `mouse.Press`:

```go
char.Bind(func(id int, me interface{}) int {
    char := event.GetEntity(id).(*entities.Moving)
    mevent := me.(mouse.Event)
    render.DrawForTime(
        render.NewLine(char.X()+char.W/2, char.Y()+char.H/2, mevent.X(), mevent.Y(), color.RGBA{0, 128, 0, 128}),
        time.Millisecond*50,
        1)
    return 0
}, mouse.Press)
```
The new code here calls `render.DrawForTime`, which acts just like `Draw`, followed by `time.Sleep`, followed by `Undraw`. Here we're drawing these lines for 50 milliseconds, so they'll look to be a gunshot flash. We draw the line from the center of the player by adding it's width and height over two to the origin of the line. 

### Enemies

To add in enemies we'll need two things: a global binding that regularly spawns a new enemy, and the ability for our shots to destroy those enemies. We'll do the latter first, inside our mouse press binding:

```go
x := char.X() + char.W/2
y := char.Y() + char.H/2
ray.DefaultCaster.CastDistance = floatgeom.Point2{x, y}.Sub(floatgeom.Point2{mevent.X(), mevent.Y()}).Magnitude()
hits := ray.CastTo(floatgeom.Point2{x, y}, floatgeom.Point2{mevent.X(), mevent.Y()})
for _, hit := range hits {
    hit.Zone.CID.Trigger("Destroy", nil)
}
```

This involves the use of the `collision/ray` package. We first tell the ray package's default caster (because we aren't using our own custom caster) to only project a ray as long as the distance between the origin and the mouse position. Then we ask it to Cast between those two points. `CastTo` returns a slice of `collision.Point` structs, each indicating at what point the collision between the ray and the hit space occurred while also containing a pointer to the collision space that was hit. 

Collision spaces can contain CIDs to refer to the entity that possesses them, so here we trigger on the CID of all hit spaces that they should react to a "Destroy" event. This relies on the enemies having a binding to "Destroy" to act in response.

To spawn enemies, we'll rely on the payload of the `event.Enter` event:

```go
EnemyRefresh := 30
event.GlobalBind(func(_ int, frames interface{}) int {
    f := frames.(int)
    if f%EnemyRefresh == 0 {
        NewEnemy()
    }
    return 0
}, event.Enter)
```

`event.Enter` has a payload of the number of frames that have passed since this scene started. Here we say that every time that number is divisible by `EnemyRefresh`, we should create a new enemy. An alternative would be to start a goroutine that spawned a new enemy after some duration, but the problem with that would arise if the performance of the engine significantly dropped: the game would start playing slower, but enemies would still be spawning at that constant rate. With the sort of implementation we use here, the rate of enemies stays consistent with the player's ability to react.

This is the `NewEnemy` function we call above:

```go
const EnemySpeed = 2
func NewEnemy() {
    x, y := enemyPos()

    enemy := entities.NewSolid(x, y, 16, 16,
        render.NewColorBox(16, 16, color.RGBA{200, 0, 0, 200}),
        nil, 0)

    render.Draw(enemy.R)

    enemy.UpdateLabel(Enemy)

    enemy.Bind(func(id int, _ interface{}) int {
        enemy := event.GetEntity(id).(*entities.Solid)
        // move towards the player
        x, y := enemy.GetPos()
        pt := floatgeom.Point2{x, y}
        pt2 := floatgeom.Point2{playerPos.X(), playerPos.Y()}
        delta := pt2.Sub(pt).Normalize().MulConst(EnemySpeed)
        enemy.ShiftPos(delta.X(), delta.Y())
        return 0
    }, event.Enter)

    enemy.Bind(func(id int, _ interface{}) int {
        enemy := event.GetEntity(id).(*entities.Solid)
        enemy.Destroy()
        return 0
    }, "Destroy")
}
```

`enemyPos` just returns a random point along the border of the screen. We then create a new solid entity, draw it, give it the `Enemy` label, and give it a binding to slowly follow the player. As a part of this slowly following the player we've made the `Vector` underlying the player a global var element in this file as `playerPos`. Finally we add a reaction to the `Destroy` event, which calls `Solid.Destroy()` to undraw the entity, unbind it's bindings, and remove it from its collision tree.

Now we'll have this when we run the game:

![](https://i.imgur.com/7Af2WvY.gif)  

### Replacing Colored Boxes with Sprites

Now most games don't draw their characters and backgrounds as a bunch of solid colors, so we're going to replace the enemies, player, and background with some sprites. To start off doing these we'll be using oak's batch loading feature, where if we indicate to oak to batch load our assets, when oak starts all of our assets will be loaded in before the game starts. 

```go
// This indicates to oak to automatically open and load image and audio
// files local to the project before starting any scene.
oak.SetupConfig.BatchLoad = true

oak.Init("tds")
```

This only loads assets within specific paths (also defined in oak.SetupConfig), and we're using the default paths for this game.

Next when we initialize our scene we'll get our assets that were loaded in and replace the player with their new sprite:

```go
var err error
sheet = render.GetSheet(filepath.Join("16x16", "sheet.png"))

// Player setup
eggplant, err := render.GetSprite(filepath.Join("character", "eggplant-fish.png"))
playerR := render.NewSwitch("left", map[string]render.Modifiable{
    "left": eggplant,
    // We must copy the sprite before we modify it, or "left"
    // will also be flipped.
    "right": eggplant.Copy().Modify(mod.FlipX),
})
if err != nil {
    dlog.Error(err)
}
```

This demonstrates two different ways of loading in basic image files: as sprite sheets or as one single image. Sprite sheets require that the files underneath them are divisible in width and height by their expected tile width and height, and the batch loader will split up and store the sheet split in those tiles, in this case, 16x16 in size each. 

We'll have a trivial animation that just has the player facing left or right depending on where they were going last, so we use a `render.Switch` to store both the left facing and right facing sprites for the player. We'll do this for both enemies and the player, so in each of their enter bindings we'll have something like this:

```go
// update animation
swtch := enemy.R.(*render.Switch)
if delta.X() > 0 {
    if swtch.Get() == "left" {
        swtch.Set("right")
    }
} else if delta.X() < 0 {
    if swtch.Get() == "right" {
       swtch.Set("left")
    }
}
```

To actually get the enemy's sprite, it is stored in the top left slot of the sheet we loaded in at the start of the scene:

```go
enemyFrame := sheet[0][0].Copy()
enemyR := render.NewSwitch("left", map[string]render.Modifiable{
    "left":  enemyFrame,
    "right": enemyFrame.Copy().Modify(mod.FlipX),
})
enemy := entities.NewSolid(x, y, 16, 16,
    enemyR,
    nil, 0)
```

And finally we draw the background, which is just a random scattering of the remaining sprites in the sheet.

```go
// Draw the background
for x := 0; x < oak.ScreenWidth; x += 16 {
    for y := 0; y < oak.ScreenHeight; y += 16 {
        i := rand.Intn(3) + 1
        // Get a random tile to draw in this position
        sp := sheet[i/2][i%2].Copy()
        sp.SetPos(float64(x), float64(y))
        render.Draw(sp, 1)
    }
}
```

### Manipulating the Viewport

We don't always want to be restricted to the initial screen size of the game, so the game world should really be larger than the screen size. This requires that we have a way of telling oak which part of the game world should be on screen, and that is done through `oak.ViewPos` and `oak.SetScreen`. 

To set the bounds of our game world, where the viewport is not allowed to pass, we use `SetViewportBounds`:

```go
const (
    fieldWidth  = 1000
    fieldHeight = 1000
)

func main() {

    oak.Add("tds", func(string, interface{}) {
        // Initialization
        playerAlive = true
        var err error
        sheet = render.GetSheet(filepath.Join("16x16", "sheet.png"))

        oak.SetViewportBounds(0, 0, fieldWidth, fieldHeight)
        ...
```

We'll also restrict the player's movement from leaving the game world, and have the viewport follow the player:

```go
// ... in the character's event.Enter binding
// Don't go out of bounds
if char.X() < 0 {
    char.SetX(0)
} else if char.X() > fieldWidth-char.W {
    char.SetX(fieldWidth - char.W)
}
if char.Y() < 0 {
    char.SetY(0)
} else if char.Y() > fieldHeight-char.H {
    char.SetY(fieldHeight - char.H)
}
oak.SetScreen(
    int(char.R.X())-oak.ScreenWidth/2,
    int(char.R.Y())-oak.ScreenHeight/2,
)
```

Here we use `oak.SetScreen` to change `oak.ViewPos` in a way that respects the bounds we set before. If the position we give it would show anything outside of the bounds we passed into `SetViewportBounds` then `ViewPos` would either change less significantly or not change in order to avoid showing that part of the game world. 

Aside from these changes for the viewport, wherever we used `oak.ScreenWidth` or `oak.ScreenHeight` we now use our private `fieldWidth` and `fieldHeight`. In addition our way of shooting needed to be changed to incorporate `oak.ViewPos`:

```go
mx := mevent.X() + float64(oak.ViewPos.X)
my := mevent.Y() + float64(oak.ViewPos.Y)
ray.DefaultCaster.CastDistance = floatgeom.Point2{x, y}.Sub(floatgeom.Point2{mx, my}).Magnitude()
hits := ray.CastTo(floatgeom.Point2{x, y}, floatgeom.Point2{mx, my})
// ...
```

### Performance

As an end note, the way that we're drawing all of our background tiles is not particularly performant. By default, Oak uses a heap to store everything that is drawn, and we're now drawing ~4000 tiles all at the same layer of that heap. Iterating through all of the heap elements is an `n log n` operation, whereas if we just had Oak draw all of the tiles at the same layer, and not sort them with the enemies and the player, it would be a linear operation. In the case of 4000 entiites that's the difference between 40k and 4k (as a vague representation of total work spent iterating). 

In the final changes to this tutorial we add this code to set up a way of drawing every background tile below the heap, also providing some fps measures for us:

```go
oak.SetupConfig.BatchLoad = true

render.SetDrawStack(
    render.NewCompositeR(),
    render.NewHeap(false),
    render.NewDrawFPS(),
    render.NewLogicFPS(),
)

oak.Init("tds")
```

This requires changing all of our previous `render.Draw` calls to also specify which part of the stack they should be drawn on. In our case, the background will now be drawn at `0, n` and all other elements at `1, n` where `n` was their previous layer they were drawn at.

And here is a sample of the finished tutorial game:

![](https://i.imgur.com/8ssBlYh.gif)
https://imgur.com/8ssBlYh