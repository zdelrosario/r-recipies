Using the ids package
================

The [`ids` package](https://reside-ic.github.io/ids/) provides tools to
create unique identifiers.

``` r
library(ids)
```

Adjective-animal mode is my favorite\! :D

This is affected by the random seed, and one can set max word lengths
for both the adjective and animal:

``` r
set.seed(101)
adjective_animal(10, max_len = c(6, 8))
```

    ##  [1] "maxixe_toucan"   "zombie_monkfish" "beamy_harpseal"  "stunty_hawk"    
    ##  [5] "viral_mammal"    "peewee_wyvern"   "cussed_elephant" "tanned_rhino"   
    ##  [9] "mousy_godwit"    "smelly_bluefish"
