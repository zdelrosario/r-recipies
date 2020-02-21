TidyEval: Brace Syntax
================
Zach
2020-02-21

The syntax for tidy evaluation changed (Circa Feb. 2020) since I first
learned it in the DCL. The syntax now uses braces and dots rather than
explicit quotation and dereferencing. The page [Programming with
dplyr](https://dplyr.tidyverse.org/dev/articles/programming.html) has
full up-to-date details on this syntax, but I’m recording the quotation
stuff here for my own reference.

``` r
library(tidyverse)
```

When writing a function that takes **tibble variable names** (as
strings) and uses dplyr operations, you can use the `.data` pronoun with
square braces `[[ ]]`.

``` r
for (var in names(mtcars)[1:2]) {
  mtcars %>%
    count(.data[[var]]) %>% print()
}
```

    ## # A tibble: 25 x 2
    ##      mpg     n
    ##    <dbl> <int>
    ##  1  10.4     2
    ##  2  13.3     1
    ##  3  14.3     1
    ##  4  14.7     1
    ##  5  15       1
    ##  6  15.2     2
    ##  7  15.5     1
    ##  8  15.8     1
    ##  9  16.4     1
    ## 10  17.3     1
    ## # … with 15 more rows
    ## # A tibble: 3 x 2
    ##     cyl     n
    ##   <dbl> <int>
    ## 1     4    11
    ## 2     6     7
    ## 3     8    14

When writing a function that takes **tibble variables** and uses dplyr
operations, you need to use curly braces `{{ }}` to signal this
*indirection*.

``` r
var_summary <- function(data, var) {
  data %>%
    summarize(
      n = n(),
      min = min({{ var }}),
      max = max({{ var }})
    )
}

mtcars %>%
  group_by(cyl) %>%
  var_summary(mpg)
```

    ## # A tibble: 3 x 4
    ##     cyl     n   min   max
    ##   <dbl> <int> <dbl> <dbl>
    ## 1     4    11  21.4  33.9
    ## 2     6     7  17.8  21.4
    ## 3     8    14  10.4  19.2

To capture **multiple tibble variables**, use the `...` pattern. When
using this feature, the [Tidyverse style
guide](https://design.tidyverse.org/dots-prefix.html) recommends
dot-prefixing all other argument names. This is to help prevent
unintentional capture in the `...` set.

``` r
my_summarize <- function(.data, ...) {
  .data %>%
    group_by(...) %>%
    summarize(
      mass = mean(mass, na.rm = TRUE),
      height = mean(height, na.rm = TRUE)
    )
}

starwars %>% my_summarize(species, gender)
```

    ## # A tibble: 43 x 4
    ## # Groups:   species [38]
    ##    species   gender  mass height
    ##    <chr>     <chr>  <dbl>  <dbl>
    ##  1 Aleena    male    15       79
    ##  2 Besalisk  male   102      198
    ##  3 Cerean    male    82      198
    ##  4 Chagrian  male   NaN      196
    ##  5 Clawdite  female  55      168
    ##  6 Droid     none   140      200
    ##  7 Droid     <NA>    46.3    120
    ##  8 Dug       male    40      112
    ##  9 Ewok      male    20       88
    ## 10 Geonosian male    80      183
    ## # … with 33 more rows
