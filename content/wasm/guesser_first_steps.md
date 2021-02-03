+++
weight = 1
sort_by = "weight"
title = "Guesser: First Steps"
+++
Hello,

This is the first post in a series that will take you through the creation of a
guessing game using embedded `WASM`.

The initial idea is really simple, we build a native application that will run
`WASM` modules as players in a game where the objective is to guess a number.
The main objective of this series is to serve as a gentle introduction to
embedding `WASM` in an application.

All the code will be written in the `Rust` programing language, and we will use
the [wasmer](https://wasmer.io/) library for running our modules.

# TODO
