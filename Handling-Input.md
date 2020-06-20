Our starting point for this page is the template from [A First Oak Program](https://github.com/oakmound/oak/wiki/Your-First-Oak-Program).

### Event Bindables

Both mouse and keyboard events, which we'll cover in this page, are reacted to through the `event` package. Reacting to events occurs by creating a binding on a `CID` or Caller ID from that package, through `CID.Bind`, for example. In this page, however, we are not reacting in the context of a particular entity or caller, so we'll use `event.GlobalBind`, a function that reacts to all events triggered of the appropriate event name. 

both `CID.Bind` and `event.GlobalBind` take in two arguments: an `event.Bindable` and an event string. When the appropriate event, named by the event string, occurs, the bindable will be called.  

Bindables have the following function signature:

```go
type Bindable func(int, interface{}) int
```

The integer argument to a bindable is the CID of the entity triggered on, or in the case of `event.GlobalBind`, 0. The `interface{}` argument is whatever data was included with the triggering event, documented by each event itself. The return value of a bindable is a success or error code, ala C. Returning 0 indicates to the event handler to do nothing. Other return results are dealt with in other articles.

### Mouse Events

The set of mouse events is documented by the `mouse` package, in strings.go:

```go
// Mouse events: MousePress, MouseRelease, MouseScrollDown, MouseScrollUp, MouseDrag
// Payload: (mouse.Event) details on the mouse event
const (
    Press      = "MousePress"
    Release    = "MouseRelease"
    ScrollDown = "MouseScrollDown"
    ScrollUp   = "MouseScrollUp"
    Click      = "MouseClick"
    Drag       = "MouseDrag"
    //
    PressOn      = Press + "On"
    ReleaseOn    = Release + "On"
    ScrollDownOn = ScrollDown + "On"
    ScrollUpOn   = ScrollUp + "On"
    ClickOn      = Click + "On"
    DragOn       = Drag + "On"
)
```

We bind to these events in our scene initialization function:

```go
//...
func (string, interface{}) {
    event.GlobalBind(func(no int, mevent interface{}) int {
        return 0
    }, mouse.Press)
},
//...
```
For now, lets just print out the position the mouse was pressed at:

```go
//...
func (string, interface{}) {
    event.GlobalBind(func(no int, mevent interface{}) int {
        me := mevent.(mouse.Event)
        fmt.Println("Mouse position:", me.X(), me.Y())
        return 0
    }, mouse.Press)
},
//...
```

Now if we `go run core.go` and click on the screen, we'll see print statements in the console informing us where we clicked. For more info about when each mouse event is triggered, see the [Mouse Events](https://github.com/oakmound/oak/wiki/Mouse-Events) page.

The package variable `mouse.LastEvent` is tracked for cases where you want to respond to what the mouse is doing, but not as it is doing it. `mouse.LastEvent` will always store the last received mouse event. For thread safety, copy the value of `mouse.LastEvent` into your code if you are going to be checking it multiple times.

### Keyboard Events

Oak only tracks two keyboard events: `key.Down` and `key.Up`, however for each of these events you can also bind to it occurring for an individual key. The `key` package tracks all keys that can be bound to. When a key event is triggered, the payload data it will send is the string value of the key triggered, but when responding to a specific key no payload will be sent:

```go
//...
func (string, interface{}) {
    event.GlobalBind(func(no int, st interface{}) int {
        k := st.(string)
        fmt.Println("Key pressed:", k)
        return 0
    }, key.Down)
    event.GlobalBind(func(no int, st interface{}) int {
        // st is nil!
        fmt.Println("Spacebar pressed")
        return 0
    }, key.Down + key.Spacebar)
},
//...
```

`key.Down` is triggered when a key is first pressed down. Unlike in, say, a text editor, if the OS sends this key again because it continues to be held down, this event will not be triggered again for each resent event.
`key.Up` is triggered when a key is released. 

To check how long a key has been held for, use `oak.IsHeld(key string)`, which will return both whether the key is currently held and for how long it has been held. Similarly, whether a key is currently held down can be checked with `oak.IsDown(key string)`.