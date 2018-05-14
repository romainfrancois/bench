
<!-- README.md is generated from README.Rmd. Please edit that file -->

# bench

[![Travis build
status](https://travis-ci.org/r-lib/bench.svg?branch=master)](https://travis-ci.org/r-lib/bench)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/github/jimhester/bench?branch=master&svg=true)](https://ci.appveyor.com/project/jimhester/bench)
[![Coverage
status](https://codecov.io/gh/r-lib/bench/branch/master/graph/badge.svg)](https://codecov.io/github/r-lib/bench?branch=master)

The goal of bench is to benchmark code, tracking execution time, memory
allocations and garbage collections.

## Installation

You can install the development version from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("r-lib/bench")
```

## Features

`bench::mark()` is used to benchmark one or a series of expressions, we
feel it has a number of advantages over [alternatives](#alternatives).

  - Always uses the highest precision APIs available for each operating
    system (often nanoseconds).
  - Tracks memory allocations for each expression.
  - Tracks the number and type of R garbage collections per expression
    iteration.
  - Verifies equality of expression results by default, to avoid
    accidentally benchmarking inequivalent code.
  - Has `bench::press()`, which allows you to easily perform and combine
    benchmarks across a large grid of values.
  - Uses adaptive stopping by default, running each expression for a set
    amount of time rather than for a specific number of iterations.
  - Expressions are run in batches and summary statistics are calculated
    after filtering out iterations with garbage collections. This allows
    you to isolate the performance and effects of garbage collection on
    running time (for more details see [Neal
    2014](https://radfordneal.wordpress.com/2014/02/02/inaccurate-results-from-microbenchmark/)).

The times and memory usage are returned as custom objects which have
human readable formatting for display (e.g. `104ns`) and comparisons
(e.g. `x$mem_alloc > "10MB"`).

There is also full support for plotting with
[ggplot2](http://ggplot2.tidyverse.org/) including custom scales and
formatting.

## Usage

Benchmarks can be run with `bench::mark()`, which takes one or more
expressions to benchmark against each other.

``` r
library(bench)
set.seed(42)
dat <- data.frame(x = runif(10000, 1, 1000), y=runif(10000, 1, 1000))
```

`bench::mark()` will throw an error if the results are not equivalent,
so you don’t accidentally benchmark inequivalent code.

``` r
bench::mark(
  dat[dat$x > 500, ],
  dat[which(dat$x > 499), ],
  subset(dat, x > 500))
#> Error: All results must equal the first result:
#>   `dat[dat$x > 500, ]` does not equal `dat[which(dat$x > 499), ]`
```

Results are easy to interpret, with human readable units.

``` r
bench::mark(
  dat[dat$x > 500, ],
  dat[which(dat$x > 500), ],
  subset(dat, x > 500))
#> # A tibble: 3 x 10
#>   expression                     min     mean   median      max `itr/sec` mem_alloc  n_gc n_itr total_time
#>   <chr>                     <bch:tm> <bch:tm> <bch:tm> <bch:tm>     <dbl> <bch:byt> <dbl> <int>   <bch:tm>
#> 1 dat[dat$x > 500, ]           306µs    390µs    341µs   1.27ms     2561.     416KB    44   848      331ms
#> 2 dat[which(dat$x > 500), ]    231µs    280µs    255µs    1.2ms     3572.     357KB    60  1283      359ms
#> 3 subset(dat, x > 500)         372µs    411µs    395µs   1.38ms     2431.     548KB    63   789      325ms
```

`bench::press()` is used to run benchmarks against a grid of parameters.
Provide setup and benchmarking code as a single unnamed argument then
define sets of values as named arguments. The full combination of values
will be expanded and the benchmarks are then *pressed* together in the
result. This allows you to benchmark a set of expressions across a wide
variety of input sizes, perform replications and other useful tasks.

``` r
set.seed(42)

create_df <- function(rows, cols) {
  as.data.frame(setNames(
    replicate(cols, runif(rows, 1, 1000), simplify = FALSE),
    rep_len(c("x", letters), cols)))
}

results <- bench::press(
  rows = c(10000, 100000),
  cols = c(10, 100),
  {
    dat <- create_df(rows, cols)
    bench::mark(
      min_iterations = 100,
      bracket = dat[dat$x > 500, ],
      which = dat[which(dat$x > 500), ],
      subset = subset(dat, x > 500)
    )
  }
)
```

## Plotting

`ggplot2::autoplot()` can be used to generate an informative default
plot. This plot is colored by gc level (0, 1, or 2) and faceted by
parameters (if any). By default it generates a
[beeswarm](https://github.com/eclarke/ggbeeswarm#geom_quasirandom) plot,
however you can also use specify other plot types (`jitter`, `ridge`,
`boxplot`, `violin`). See `?autoplot.bench_mark` for full details.

``` r
ggplot2::autoplot(results)
```

<img src="man/figures/README-autoplot-1.png" width="100%" />

You can also produce fully custom plots by un-nesting the results and
working with the data directly.

``` r
library(tidyverse)
results %>%
  unnest() %>%
  filter(gc == "none") %>%
  ggplot(aes(x = mem_alloc, y = time, color = expression)) +
    geom_point() +
    scale_color_brewer(type = "qual", palette = 3)
```

<img src="man/figures/README-custom-plot-1.png" width="100%" />

## `system_time()`

**bench** also includes `system_time()`, a higher precision alternative
to
[system.time()](https://www.rdocumentation.org/packages/base/versions/3.5.0/topics/system.time).

``` r
bench::system_time({ i <- 1; while(i < 1e7) i <- i + 1 })
#> process    real 
#>   333ms   333ms
bench::system_time(Sys.sleep(.5))
#> process    real 
#>    86µs   502ms
```

## Alternatives

  - [rbenchmark](https://cran.r-project.org/package=rbenchmark)
  - [microbenchmark](https://cran.r-project.org/package=microbenchmark)
  - [tictoc](https://cran.r-project.org/package=tictoc)
  - [system.time()](https://www.rdocumentation.org/packages/base/versions/3.5.0/topics/system.time)
