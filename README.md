# Defer-Friendly Program Exit

[![GoDoc](https://pkg.go.dev/badge/github.com/bassosimone/deferexit)](https://pkg.go.dev/github.com/bassosimone/deferexit) [![Build Status](https://github.com/bassosimone/deferexit/actions/workflows/go.yml/badge.svg)](https://github.com/bassosimone/deferexit/actions) [![codecov](https://codecov.io/gh/bassosimone/deferexit/branch/main/graph/badge.svg)](https://codecov.io/gh/bassosimone/deferexit)

The `deferexit` Go package implements panic-based program exit so that
deferred cleanup (lockfile release, marker cleanup, temp file removal,
etc.) actually runs before the process terminates.

Calling `os.Exit` directly from `main` skips all `defer`s registered
up the stack. This package replaces that pattern with a typed panic
that the outermost `main` recovers from and converts into a real
`os.Exit`, after every deferred function has run.

```Go
import (
	"os"

	"github.com/bassosimone/deferexit"
)

func main() {
	defer deferexit.Recover(os.Exit)
	realMain()
}

func realMain() {
	defer cleanup() // actually runs, even on the failure path

	if somethingWrong() {
		deferexit.Panic(2) // unwinds the stack instead of os.Exit(2)
	}
}
```

`deferexit.Run` is a test-only helper that runs a `main`-like function
and returns the exit code carried by any `Panic` raised inside it (or
zero if it returned normally). Production `main` should NOT be written
as `os.Exit(Run(realMain))`: that pattern calls `os.Exit`
unconditionally even on the success path, which makes `main`
untestable. See the package documentation for details.

## Installation

To add this package as a dependency to your module:

```sh
go get github.com/bassosimone/deferexit
```

## Development

To run the tests:

```sh
go test -v .
```

To measure test coverage:

```sh
go test -v -cover .
```

## License

```
SPDX-License-Identifier: GPL-3.0-or-later
```

## History

This package graduated from
[bassosimone/npte](https://github.com/bassosimone/npte),
where it was originally introduced. It was promoted to its own module
to be reusable across unrelated projects that share the same need
for defer-friendly program exit.
