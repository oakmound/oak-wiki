Oak provides `oak/alg` as a package to handle relatively miscellaneous math and selection algorithms. This package is a candidate for renaming in the future, or moving all of its contents to subpackages due to its miscellaneous nature. Right now `alg` itself contains several utilities and selection algorithms, while its two subpackages, `intgeom` and `floatgeom` respectively handle integer and floating point geometry data types. 

### Selection Algorithms

`ChooseX` and `UniqueChooseX` both operate on slices of `float64` weights, returning a slice of indices of chosen elements from that weight set. `ChooseX` can choose the same index more than once, while `UniqueChooseX` cannot. In other words, `ChooseX` is _with replacement_ and `UniqueChooseX` is _without replacement_.

There exists two smaller utilities, `WeightedChooseOne` and `WeightedMapChoice`, both of which operate on lists of cumulative weights. We call _cumulative weights_ here a set of floats converted from their initial values so that each index represents the total sum of all indices before or after it in the original set, including that index. There are two potential orderings of cumulative weights, one with the total weight at index 0, returned by `CumulativeWeights`, and one with the total weight at the final index, returned by `RemainingWeights`. Both `Weighted` functions accept weights stylized as `RemainingWeights` returns, with the latter operating on a map, if you happen to be using a map instead of a slice.


### Math Helpers

`F64eq` and `F64eqEps` both equate floating points within input epsilon values, the former using a package local value. A user might find this useful if they're doing a lot of geometry with floating points.

`RoundF64` rounds a float to an integer.

In addition, `DegToRad` and `RadToDeg` hold constant values that can be used to convert radians and degrees between one another. 

### Floatgeom and Intgeom

Both the `alg/floatgeom` and `alg/intgeom` packages provide `Point` and `Rect` types for working with floating point and integer geometry. Utilities on these types range from arithmetic to angle manipulation, to dot and cross products, and is continually expanding. 