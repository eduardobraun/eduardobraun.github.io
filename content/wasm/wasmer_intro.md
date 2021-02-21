+++
weight = 1
sort_by = "weight"
title = "Wasmer Introduction"
+++
[Wasmer](https://wasmer.io/) is a `WASM` runtime coded in `Rust`, so it is
really easy to embed the runtime inside you own `Rust` programs.
The library is great, but its website and [docs](https://docs.wasmer.io/) seems
to be targeting its use as a CLI tool and not as a library.

If you dig a little bit you will find a link to the [wasmer-runtime
docs](https://docs.rs/wasmer-runtime/0.17.1/wasmer_runtime/), which is good to
look at documentation of specific components of the library, but it is far from
an introduction to its features.

To really see what the `Wasmer` is about we must dive into the
[examples](https://github.com/wasmerio/wasmer/tree/master/examples).

Trying to make sense of various examples may be a little time consuming, and
that is where this post series comes in, giving an easy to follow introduction
to using the `Wasmer` library and to `WASM` in general.

# Wasmer Basics
We can summarize most of the examples in 5 steps:
```rust
// 1. Get the WASM code
let wasm_bytes = get_wasm_bytes();

// 2. Setup the runtime
let compiler_config = Cranelift::default();
let engine = JIT::new(compiler_config).engine();
let store = Store::new(&engine);

// 3. Compile the WASM code into a Module
let module = Module::new(&store, wasm_bytes)?;

// 4. Create an instance of the Module
let import_object = imports! {};
let instance = Instance::new(&module, &import_object)?;

// 5. Get the exported function reference and call it
let sum = instance.exports.get_function("sum")?;
let results = sum.call(&[Value::I32(1), Value::I32(2)])?;
```

## Step 1: The WASM Code
First we need to get our `WASM` code. The code is just a sequence of bytes, in
the `Wasmer` example it is inlined into the code, but you can read it at
runtime from anywhere.

## Step 2: WASM Runtime
To run this code we first need to compile it, we will use the just-in-time
(JIT) engine from to build our code at runtime, there are other options but I
will not explore them here.

## Step 3: The Module
The chosen engine is used to build the `WASM` code into a `Module`.

## Step 4: Instantiation
This `Module` can be instantiated.

## Step 5: Use The Instance
Finally, we can get references to the `WASM` module exported data, this allows
us to call functions that will be run inside the `Wasmer` sandboxed
environment.
