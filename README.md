<!-- README.md is generated from README.Rmd. Please edit that file -->

[![Build
Status](https://travis-ci.org/hypertidy/geodist.svg)](https://travis-ci.org/hypertidy/geodist)
[![Project Status: Concept - Minimal or no implementation has been done
yet.](http://www.repostatus.org/badges/0.1.0/concept.svg)](http://www.repostatus.org/#concept)
[![codecov](https://codecov.io/gh/hypertidy/geodist/branch/master/graph/badge.svg)](https://codecov.io/gh/hypertidy/geodist)
[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/geodist)](http://cran.r-project.org/web/packages/geodist)
![downloads](http://cranlogs.r-pkg.org/badges/grand-total/geodist)

# geodist

An ultra-lightweight package for calculation of geodesic distances. Only
package dependency is `Rcpp`.

## Installation

You can install geodist from github with:

``` r
# install.packages("devtools")
devtools::install_github("hypertidy/geodist")
```

## Usage

`geodist` contains only one eponymous function which accepts only one or
two arguments, each of which must be some kind of rectangular object
with unambiguously labelled longitude and latitude columns (that is,
some variant of `lon`/`lat`, or `x`/`y`).

    #> Loading geodist

``` r
library(geodist)
```

``` r
# current verison
packageVersion("geodist")
#> [1] '0.0.0.9000'
```

``` r
n <- 1e1
x <- tibble::tibble (x = -180 + 360 * runif (n),
                     y = -90 + 180 * runif (n))
dim (geodist (x))
#> [1] 10 10
y <- tibble::tibble (x = -180 + 360 * runif (2 * n),
                     y = -90 + 180 * runif (2 * n))
dim (geodist (x, y))
#> [1] 10 20
```

### Comparison

Geodesic distance calculation is available in the [`sf`
package](https://cran.r-project.org/package=sf), requiring only
conversion of sets of numeric lon-lat points to `sf` form with the
following code:

``` r
require (magrittr)
#> Loading required package: magrittr
x_to_sf <- function (x)
{
    sapply (seq (nrow (x)), function (i)
            sf::st_point (x [i, ]) %>%
                sf::st_sfc ()) %>%
    sf::st_sfc ()
}
```

Note, however, that the relevant function, `sf::st_distance()`
calculates the Vicenty distance via `liblwgeom`, and that these
calculations are considerably more complicated than Haversine distances.

Haversine distances are also very straightforward to calculate directly
in matrix form as follows:

``` r
havdist <- function (x)
{
    xmat <- array (x [, 1], dim = rep (nrow (x), 2))
    ymat <- array (x [, 2], dim = rep (nrow (x), 2))
    xd <- (xmat - t (xmat)) * pi / 180
    yd <- (ymat - t (ymat)) * pi / 180
    sxd <- sin (xd / 2)
    syd <- sin (yd / 2)
    earth <- 6378.137 # radius of Earth
    d <- syd * syd +
        cos (ymat * pi / 180) * cos (t (ymat) * pi / 180) * sxd * sxd
    2 * earth * asin (sqrt (d));
}
```

The following code demonstrates relative speeds of the three ways of
calculating distances:

``` r
n <- 1e3
x <- cbind (-180 + 360 * runif (n), -90 + 180 * runif (n))
colnames (x) <- c ("x", "y")
xsf <- x_to_sf (x)
sf_dist <- function (x) sf::st_distance (x, x)
rbenchmark::benchmark (replications = 10, order = "test",
                      havdist (x),
                      sf_dist (xsf),
                      geodist (x)) [, 1:4]
#>           test replications elapsed relative
#> 3   geodist(x)           10   0.416    1.000
#> 1   havdist(x)           10   1.888    4.538
#> 2 sf_dist(xsf)           10   7.114   17.101
```

### Test Results

``` r
require (devtools)
require (testthat)
```

``` r
date()
#> [1] "Wed Jun 13 15:00:30 2018"
devtools::test("tests/")
#> Loading geodist
#> Testing geodist
#> ✔ | OK F W S | Context
✔ | 14       | geodist [0.1 s]
#> 
#> ══ Results ════════════════════════════════════════════════════════════════
#> Duration: 0.2 s
#> 
#> OK:       14
#> Failed:   0
#> Warnings: 0
#> Skipped:  0
```
