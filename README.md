<div align="center">
  <h1><code>wasmtime-go</code></h1>

  <p>
    <strong>Go embedding of
    <a href="https://github.com/bytecodealliance/wasmtime">Wasmtime</a></strong>
  </p>

  <strong>A <a href="https://bytecodealliance.org/">Bytecode Alliance</a> project</strong>

  <p>
    <a href="https://github.com/bytecodealliance/wasmtime-go/actions?query=workflow%3ACI">
      <img src="https://github.com/bytecodealliance/wasmtime-go/workflows/CI/badge.svg" alt="CI status"/>
    </a>
    <a href="https://pkg.go.dev/github.com/bytecodealliance/wasmtime-go">
      <img src="https://godoc.org/github.com/bytecodealliance/wasmtime-go?status.svg" alt="Documentation"/>
    </a>
    <a href="https://bytecodealliance.github.io/wasmtime-go/coverage.html">
      <img src="https://img.shields.io/badge/coverage-master-green" alt="Code Coverage"/>
    </a>
  </p>

</div>

## Installation

```sh
go get -u github.com/bytecodealliance/wasmtime-go
```

Be sure to check out the [API documentation][api]!

This Go library uses CGO to consume the C API of the [Wasmtime
project][wasmtime] which is written in Rust. Precompiled binaries of Wasmtime
are checked into this repository on tagged releases so you won't have to install
Wasmtime locally, but it means that this project only works on Linux x86\_64 and
macOS x86\_64 currently.

[api]: https://pkg.go.dev/github.com/bytecodealliance/wasmtime-go
[wasmtime]: https://github.com/bytecodealliance/wasmtime

## Usage

A "Hello, world!" example of using this package looks like:

```go
package main

import (
    "fmt"
    "github.com/bytecodealliance/wasmtime-go"
)

func main() {
    // Almost all operations in wasmtime require a contextual `store`
    // argument to share, so create that first
    store := wasmtime.NewStore(wasmtime.NewEngine())

    // Compiling modules requires WebAssembly binary input, but the wasmtime
    // package also supports converting the WebAssembly text format to the
    // binary format.
    wasm, err := wasmtime.Wat2Wasm(`
      (module
        (import "" "hello" (func $hello))
        (func (export "run")
          (call $hello))
      )
    `)
    check(err)

    // Once we have our binary `wasm` we can compile that into a `*Module`
    // which represents compiled JIT code.
    module, err := wasmtime.NewModule(store, wasm)
    check(err)

    // Our `hello.wat` file imports one item, so we create that function
    // here.
    item := wasmtime.WrapFunc(store, func() {
        fmt.Println("Hello from Go!")
    })

    // Next up we instantiate a module which is where we link in all our
    // imports. We've got one import so we pass that in here.
    instance, err := wasmtime.NewInstance(module, []*wasmtime.Extern{item.AsExtern()})
    check(err)

    // After we've instantiated we can lookup our `run` function and call
    // it.
    run := instance.GetExport("run").Func()
    _, err = run.Call()
    check(err)
}

func check(e error) {
    if e != nil {
        panic(e)
    }
}
```

## Contributing

So far this extension has been written by folks who are primarily Rust
programmers, so it's highly likely that there's some faux pas in terms of Go
idioms. Feel free to send a PR to help make things more idiomatic if you see
something!

To work on this extension locally you'll first want to clone the project:

```sh
$ git clone https://github.com/bytecodealliance/wasmtime-go
```

Next up you'll want to have a [local Wasmtime build
available](https://bytecodealliance.github.io/wasmtime/contributing-building.html).
Once you've got that you can set up the environment of this library with:

```sh
$ ./ci/local.sh /path/to/wasmtime
```

This will create a `build` directory which has the compiled libraries and header
files. Next up you can run normal commands such as:

```sh
$ go test
```

And after that you should be good to go!