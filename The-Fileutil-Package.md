Oak provides a `fileutil` package which allows for the embedding of binary data such as images and audio in an oak binary with the help of tools like go-bindata.

Through supplying `fileutil.BindataFn`, which matches the signature of the `Asset` function that go-bindata supplies, and `fileutil.BindataDir`, parallel to `AssetDir`, Oak can be instructed to search within stored binary data before looking at real local files when attempting to open asset files.

Oak provides the utility function `oak.SetBinaryPayload` to set both of these values at once. 

If you yourself are accessing assets, and not using oak's loaders, you can also check the binary functions first by using `fileutil.Open` instead of `os.Open`, and so on for `fileutil.ReadDir` and `fileutil.ReadFile`.

