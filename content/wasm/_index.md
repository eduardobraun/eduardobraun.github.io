+++
weight = 3
sort_by = "weight"
title = "WebAssembly"
+++
Here you will find posts related to `WASM`.

`WASM` was originally created to be a portable binary-code format for the web.
But it turns out that running binary-code in a sandboxed environment is great
for many other applications.  One example is network proxys, like
[Envoy](https://www.envoyproxy.io/), that usually provide a way to do advanced
filtering of requests.  In the case of `Envoy` originally these filters were
programed in `LUA`, but recently support for `WASM` filters was added.
