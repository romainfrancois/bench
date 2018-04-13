
<!-- README.md is generated from README.Rmd. Please edit that file -->

# bench

[![Travis build
status](https://travis-ci.org/jimhester/bench.svg?branch=master)](https://travis-ci.org/jimhester/bench)

The goal of bench is to benchmark code.

## Installation

You can install the development version from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("jimhester/bench")
```

## Example

This is a basic example which shows you how to solve a common problem:

``` r
dat <- data.frame(x = runif(10000, 1, 1000), y=runif(10000, 1, 1000))

# Throws an error if the results are not equivalent, so you don't accidentally
# benchmark against the wrong answer
results <- bench::mark(
  dat[dat$x > 500, ],
  dat[which(dat$x > 499), ],
  subset(dat, x > 500))
#> Error: results[[1]]$result not equal to results[[2]]$result.
#> Attributes: < Component "row.names": Numeric: lengths (4907, 4916) differ >
#> Component "x": Numeric: lengths (4907, 4916) differ
#> Component "y": Numeric: lengths (4907, 4916) differ

results <- bench::mark(
  dat[dat$x > 500, ],
  dat[which(dat$x > 500), ],
  subset(dat, x > 500))

results
#> # A tibble: 3 x 12
#>   name              relative     n     mean     min  median    max `n/sec`
#>   <chr>                <dbl> <int>    <dbl>   <dbl>   <dbl>  <dbl>   <dbl>
#> 1 dat[which(dat$x …     1.70  1192 0.000417 2.49e-4 2.70e-4 0.0308   2399.
#> 2 dat[dat$x > 500,…     1.38   965 0.000494 3.20e-4 3.49e-4 0.0304   2024.
#> 3 subset(dat, x > …     1.00   700 0.000711 3.98e-4 4.40e-4 0.0300   1406.
#> # ... with 4 more variables: allocated_memory <chr>, memory <list>,
#> #   result <list>, timing <list>
```

``` r
library(tidyverse)
results %>%
  mutate(name = fct_reorder(name, relative)) %>%
  select(name, timing) %>%
  unnest() %>%
  ggplot(aes(name, timing, color = name)) +
    geom_jitter() +
    scale_y_log10() +
    coord_flip()
```

<img src="man/figures/README-pressure-1.png" width="100%" />