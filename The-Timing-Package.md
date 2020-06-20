One of the sub-packages that Oak provides is `timing`, which provides miscellaneous utilities related to delays, converting time values, and a `time.Ticker` equivalent which can have it's tick rate changed actively.

### Delaying Functions

In normal Go code to delay a function call you might call `time.Sleep` and then your function, or in a `select` block with `time.After` and some alternatives. In Oak, this is a problem similar to the problem faced in network routing where scenes can change at any moment, or in other words, delayed calls might need to be cancelled and not executed. Failing to take this into account can result in code attempting to use a CID from an old scene which now maps to a new entity, or no entity, causing a crash.

The Utility that oak provides to resolve this is `timing.DoAfter`. All this does is wrap your given function in a select with a `time.After` call, but whenever a scene ends all waiting `timing.DoAfter` calls will be terminated without their function arguments being called.

### FPS Operations

For internal use, but exported for anyone else to use, Oak provides functions which convert between frames-per-second or FPS values and Nanoseconds, `timing.FPSToDuration` and `timing.FPSToNano`, and a function to calculate FPS given two timestamps, `timing.FPS`.

### DynamicTicker

A `timing.DynamicTicker` is a type which is used by Oak's draw loop and event loop to process draw events and new frame events at specific FPS rates. It is exported, like the FPS operations, for anyone else to use despite being designed for internal use. 

One sets a DynamicTicker up by calling `NewDynamicTicker()` and `DynamicTicker.SetTick(time.Duration)`. `DynamicTicker.C` will have a tick sent every time that that input duration passes. If `SetTick()` is never called, no signal will ever be sent along that channel. Signals can be forced to be sent through `DynamicTicker.Step()`, and `Stop()` will close off the ticker from being used. 