Oak internally has all of it's debug and logging calls go through the `dlog` package. This package exports the function `SetLogger` which users can use to override default logging behavior. If the user does nothing, a default logger will be used.

### The Default Logger

The default logger in `dlog` will simultaneously write to file and `fmt.Print` incoming log statements. It will attempt to create a log file with a name matching the current time when oak is run, in a `logs` directory relative to where oak is run from. If this directory does not exist, no logs will be written to file.

The default logger supports four levels of logging: `Verb`, `Info`, `Warn`, and `Error`, and will only display (but will still write to file) messages that contain a customizable string filter. The error level and string filter are set through `oak.SetupConfig`. 

### Replacing the Default Logger

Users are able to replace the default logger with anything that satsfies one of two interfaces-- it can either satisfy `dlog.Logger`, which supports four-level logging, or it can satisfy `dlog.FullLogger` which has additional utilities along with file creation, logging just to file, and setting debug filter and level. If the input to `SetLogger` does not satisfy `dlog.FullLogger`, these additional operations will become NOPs. 