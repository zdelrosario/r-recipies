Setup: Use Python in RStudio
================

It is possible to run Python code from RStudio, but there are known
issues with
[reticulate](https://github.com/rstudio/reticulate/issues/831) that
complicate the setup.

One-time step: Install the development version of `reticulate`.

``` r
# remotes::install_github("rstudio/reticulate")
```

Code setup: Use an `r` chunk to target your local python binary:

``` r
reticulate::use_python("/Users/zach/opt/anaconda3/bin/python") # On my macbook
```

You can now create and run Python chunks\!

``` python
import numpy as np
A = np.arange(9).reshape((-1, 3))
A
```

    ## array([[0, 1, 2],
    ##        [3, 4, 5],
    ##        [6, 7, 8]])
