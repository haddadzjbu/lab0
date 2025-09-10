MA 415/615 – Lab 0: Workflow example
================
Zainab Alhaddad
Week of 2025-09-08

# Introduction

This is a simple notebook documenting the basic analyses in Lecture 2 of
MA 415 / MA 615.

## Setting up R

After starting RStudio, we load packages in `tidyverse`:

``` r
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 4.4.3

    ## Warning: package 'ggplot2' was built under R version 4.4.3

    ## Warning: package 'tibble' was built under R version 4.4.3

    ## Warning: package 'tidyr' was built under R version 4.4.3

    ## Warning: package 'readr' was built under R version 4.4.3

    ## Warning: package 'purrr' was built under R version 4.4.3

    ## Warning: package 'dplyr' was built under R version 4.4.3

    ## Warning: package 'stringr' was built under R version 4.4.2

    ## Warning: package 'forcats' was built under R version 4.4.3

    ## Warning: package 'lubridate' was built under R version 4.4.3

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.1.0     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

If the package is not installed, we can do it with:

``` r
install.packages("tidyverse") # it may take a while!
```

# Exploring the data

We will be using the `mpg` dataset on fuel economy data:

``` r
mpg
```

    ## # A tibble: 234 × 11
    ##    manufacturer model      displ  year   cyl trans drv     cty   hwy fl    class
    ##    <chr>        <chr>      <dbl> <int> <int> <chr> <chr> <int> <int> <chr> <chr>
    ##  1 audi         a4           1.8  1999     4 auto… f        18    29 p     comp…
    ##  2 audi         a4           1.8  1999     4 manu… f        21    29 p     comp…
    ##  3 audi         a4           2    2008     4 manu… f        20    31 p     comp…
    ##  4 audi         a4           2    2008     4 auto… f        21    30 p     comp…
    ##  5 audi         a4           2.8  1999     6 auto… f        16    26 p     comp…
    ##  6 audi         a4           2.8  1999     6 manu… f        18    26 p     comp…
    ##  7 audi         a4           3.1  2008     6 auto… f        18    27 p     comp…
    ##  8 audi         a4 quattro   1.8  1999     4 manu… 4        18    26 p     comp…
    ##  9 audi         a4 quattro   1.8  1999     4 auto… 4        16    25 p     comp…
    ## 10 audi         a4 quattro   2    2008     4 manu… 4        20    28 p     comp…
    ## # ℹ 224 more rows

We get a description of the dataset by issuing `help(mpg)` or just
`?mpg`.

Our main *question of interest* is to find a relationship between
`displ` (engine displacement in litres) and `hwy` (highway mileage).
Initial visualization with a scatterplot:

``` r
ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) + geom_point()
```

<img src="figures/displhwy1-1.png" width="70%" style="display: block; margin: auto;" />

Mileage seems to *decrease* with engine size; what is the effect of
`cyl` (number of cylinders)?

``` r
ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) +
  geom_point(aes(color = cyl))
```

<img src="figures/displhwy2-1.png" width="70%" style="display: block; margin: auto;" />

We notice that `cyl` has only a few values so it’s best seen as a
*factor* (categorical variable):

``` r
ggplot(data = mpg |> mutate(cyl = factor(cyl)),
       aes(x = displ, y = hwy)) + geom_point(aes(color = cyl))
```

<img src="figures/displhwy3-1.png" width="70%" style="display: block; margin: auto;" />

## A bit of modeling

There seems to be a functional relationship between `displ` and `hwy`:

``` r
ggplot(mpg |> mutate(cyl = factor(cyl)), aes(displ, hwy)) +
  geom_point(aes(color = cyl)) +
  geom_smooth()
```

<img src="figures/displhwy4-1.png" width="70%" style="display: block; margin: auto;" />

We can attempt to find such a relationship using a linear model. We
assume that if $H$ is `hwy` and $D$ is `displ` then, at least
approximately on average, we have

$$H \propto D^{\beta}$$ After taking the log of both sides, this yields
the following *linear model*:

$$\log(H) = \alpha + \beta \cdot \log(D)$$ Fitting the model yields

``` r
fit <- lm(log(hwy) ~ log(displ), data = mpg) 
coef(fit)
```

    ## (Intercept)  log(displ) 
    ##   3.7640700  -0.5471563

and thus, $\beta \approx -0.5$, which yields

$$H \propto 1 / \sqrt{D}.$$

A visual check seems to confirm that `hwy` has an approximately linear
relationship with the inverse square root of `displ`:

``` r
ggplot(mpg |> mutate(cyl = factor(cyl)),
       aes(1 / sqrt(displ), hwy)) +
  geom_point(aes(color = cyl)) +
  geom_smooth()
```

<img src="figures/displhwy5-1.png" width="70%" style="display: block; margin: auto;" />
