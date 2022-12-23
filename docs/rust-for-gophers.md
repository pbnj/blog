---
Date: 2020-02-12
tags: ["rust", "go"]
draft: true
---

# Rust for Gophers

## Table of Contents

- [Rust for Gophers](#rust-for-gophers)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Packages](#packages)
  - [Imports](#imports)
  - [Exports](#exports)
  - [Functions](#functions)
  - [Variables](#variables)
  - [Structs](#structs)
  - [Enums](#enums)
  - [Error Handling](#error-handling)

## Introduction

I have been a Gopher (a [Golang](https://golang.org) developer) for about 5
years now. I love the language for its simplicity and pragmatism, but
[Rust](https://www.rust-lang.org) has been on my radar for a few years. I was
intrigued by its memory safety guarantees without a Garbage Collector (GC), so I
decided to explore the language.

This blog post serves as a summary of Rust language features from the
perspective of a Gopher.

## Packages

In Go, you create a namespace for your functions and variables using the
`package` keyword:

```go
package greet

import "fmt"

func hello(arg string) {
    fmt.Printf("Hello, %s", arg)
}
```

In Rust, you can do the same with `mod`:

```rust
mod foo {
  fn hello(arg String) {
    println!("Hello, {}", arg)
  }
}
```

However, idiomatic Rust encourages you to use the filesystem instead:

```rust

```

## Imports

Go:

```go

```

Rust:

```rust

```

## Exports

Go:

```go

```

Rust:

```rust

```

## Functions

Go:

```go

```

Rust:

```rust

```

## Variables

Go:

```go

```

Rust:

```rust

```

## Structs

Go:

```go

```

Rust:

```rust

```

## Enums

Go:

```go

```

Rust:

```rust

```

## Error Handling

Go:

```go

```

Rust:

```rust

```
