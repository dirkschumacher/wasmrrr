
<!-- README.md is generated from README.Rmd. Please edit that file -->

# WebAssembly in R using wasmer

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![Travis build
status](https://travis-ci.org/dirkschumacher/wasmrrr.svg?branch=master)](https://travis-ci.org/dirkschumacher/wasmrrr)
<!-- badges: end -->

The goal of `wasmr` is to run
[WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly/Concepts)
code from R using the [wasmer](https://wasmer.io/) runtime.

This is a super early version with probably bugs, missing features and
not the best code quality. But it already works and I am happy to hear
feedback.

## Installation

``` r
remotes::install_github("dirkschumacher/wasmr")
```

Also make sure to have a very recent rust compiler. The best way to
install the rust toolchain is to use [rustup](https://rustup.rs/).

## Example

``` r
library(wasmr)
```

``` r
f <- system.file("examples/sum.wasm", package = "wasmr")
instance <- instantiate(f)
instance$exports$sum(10, 20)
#> [1] 30
```

``` r
f <- system.file("examples/hello.wasm", package = "wasmr")
instance <- instantiate(f)
memory_pointer <- instance$exports$hello()
(hi <- instance$memory$get_memory_view(memory_pointer))
#>  [1] 48 65 6c 6c 6f 20 77 6f 72 6c 64
rawToChar(hi)
#> [1] "Hello world"
```

``` r
f <- system.file("examples/fib.wasm", package = "wasmr")
instance <- instantiate(f)
instance$exports$fib(20)
#> [1] 6765

fib <- function(n) {
  if (n < 2) return(n)
  fib(n - 1) + fib(n - 2)
}

microbenchmark::microbenchmark(
  instance$exports$fib(20),
  fib(20)
)
#> Unit: microseconds
#>                      expr      min       lq       mean     median
#>  instance$exports$fib(20)   70.267   75.910   173.3486   134.6565
#>                   fib(20) 7834.426 8183.016 11326.3046 10112.8035
#>         uq       max neval
#>    146.849  3401.142   100
#>  12676.181 46626.247   100
```

## Memory

``` r
f <- system.file("examples/fib.wasm", package = "wasmr")
instance <- instantiate(f)
instance$memory$get_memory_length()
#> [1] 2

# grow the number of pages
instance$memory$grow(1)
instance$memory$get_memory_length()
#> [1] 3
```

### Imports

``` r
set.seed(42)
imports <- list(
  env = list(
    add = typed_function(
      function(a, b) {
        # use R's RNG to add a random number to the result
        a + b * as.integer(runif(1) * 100)
      },
      param_types = c("I32", "I32"),
      return_type = "I32"
    )
  )
)
f <- system.file("examples/sum_import.wasm", package = "wasmr")
instance <- instantiate(f, imports)
# sum: add(a, b) + 42
instance$exports$sum(1, 5)
#> [1] 498
```

## Caveats and limitations

  - It is not feature complete and does not support everything that
    `wasm` supports
  - Imported functions can only have integer parameters
  - No globals
  - There is hardly any documentation except for the examples
  - `I32/I64` are mapped to `IntegerVector` and `F32/F64` to
    `NumericVector`. Currently no way differentiate.
  - WIP

## Inspiration and References

  - [wasmer Python](https://github.com/wasmerio/python-ext-wasm) -
    haven’t tried it but looked at the API/examples
  - [wasmer](https://github.com/wasmerio/wasmer) - especially the C api
    and the tests give some good examples.
  - @jeroen ’s [rust template](https://github.com/r-rust/hellorust)
  - @romainfrancois ’s
    [altrepisode](https://github.com/romainfrancois/altrepisode) and
    @jimhester ’s [vroom](https://github.com/r-lib/vroom) on how to use
    ALTREP and C++.

## Contribute

While this is work in progress, the best way to contribute is to test
the package and write/comment on issues. Before sending a PR it would
great to post an issue first to discuss.

## License

MIT just like wasmer.
