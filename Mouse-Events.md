This page details each mouse event:

```go
// Mouse events: MousePress, MouseRelease, MouseScrollDown, MouseScrollUp, MouseDrag
// Payload: (mouse.Event) details of the mouse event
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
All mouse events contain information about the location the mouse was when the event occurred in `X()` and `Y()`, and the button, if any, used to trigger the event, `Button`. If more than one event could have caused a mouse event in your code, you can determine which it was through the `Event` field.

`Press` is triggered when the mouse is clicked down, as it is clicked down. There needs not be a release event tied with this. If an entity binds to Press, or any of the non-'On' events, they will receive an event regardless of where the mouse was. For reacting to an entity being clicked ON, use `PressOn` instead of `Press`. This applies for all mouse events.

`Release` is triggered when the mouse button is released, parallel to Press. Press and Release are not necessarily always paired, as one or the other could occur off-screen.

`ScrollDown` and `ScrollUp` are triggered when the mouse's scroll wheel is spun down or up, respectively. For any one spin, more often than not expect multiple triggers of these events.

`Click` is tracked through the mouse package boolean `TrackMouseClicks`, which is true by default. `Click` will be triggered whenever the mouse is pressed and released on the same entity without any other presses happening in between. This requires that the entities in question have a collision space in the mouse collision tree. `ClickOn` will be triggered on the specific entity that was clicked on, while `Click` will be triggered globally.

`Drag` is triggered whenever the mouse moves. 