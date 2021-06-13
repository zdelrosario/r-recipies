Filtering ‘across’
================
Zachary del Rosario
2021-06-13

`filter()` uses special syntax for `across`; namely, there are
replacement
    helpers.

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✔ ggplot2 3.3.3     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.2     ✔ dplyr   1.0.6
    ## ✔ tidyr   1.1.3     ✔ stringr 1.4.0
    ## ✔ readr   1.4.0     ✔ forcats 0.5.0

    ## ── Conflicts ───────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

# Filter `across()` replacements

Remembering this syntax is tricky; see the
[vignette](https://dplyr.tidyverse.org/articles/colwise.html#filter) for
details. But in short there are two replacements for `across()` when
using filter:

``` r
df_tofilter <- 
  tribble(
    ~idx, ~A, ~B, ~C,
     "N",  0,  0,  0,
     "X",  1,  0,  0,
     "Y",  0,  1,  0,
     "Z",  0,  0,  1,
     "A",  1,  1,  1
  )
df_tofilter
```

    ## # A tibble: 5 x 4
    ##   idx       A     B     C
    ##   <chr> <dbl> <dbl> <dbl>
    ## 1 N         0     0     0
    ## 2 X         1     0     0
    ## 3 Y         0     1     0
    ## 4 Z         0     0     1
    ## 5 A         1     1     1

## `if_any()`

``` r
df_tofilter %>% 
  filter(if_any(c(A, B, C), ~ . > 0))
```

    ## # A tibble: 4 x 4
    ##   idx       A     B     C
    ##   <chr> <dbl> <dbl> <dbl>
    ## 1 X         1     0     0
    ## 2 Y         0     1     0
    ## 3 Z         0     0     1
    ## 4 A         1     1     1

## `if_all()`

``` r
df_tofilter %>% 
  filter(if_all(c(A, B, C), ~ . > 0))
```

    ## # A tibble: 1 x 4
    ##   idx       A     B     C
    ##   <chr> <dbl> <dbl> <dbl>
    ## 1 A         1     1     1
