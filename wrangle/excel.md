Wrangling Excel Documents
================
Zach
2019-12-10

**Purpose**: Wrangling Excel files is often *nightmarish*. This notebook
details some tricks I’ve learned to handle issues found in the wild.

``` r
library(tidyverse)
library(readxl)
library(httr)

## Use my tidy-exercises copy of UNDOC data for stability
url_undoc <- "https://github.com/zdelrosario/tidy-exercises/blob/master/2019/2019-12-10-news-plots/GSH2013_Homicide_count_and_rate.xlsx?raw=true"
```

## Example: UNDOC Homicide Dataset

<!-- -------------------------------------------------- -->

The following code will use `httr` to access the Excel file at
`url_undoc`, then pass to
    `read_excel`.

``` r
GET(url_undoc, write_disk(tf <- tempfile(fileext = ".xlsx")))
```

    ## Response [https://raw.githubusercontent.com/zdelrosario/tidy-exercises/master/2019/2019-12-10-news-plots/GSH2013_Homicide_count_and_rate.xlsx]
    ##   Date: 2019-12-11 04:42
    ##   Status: 200
    ##   Content-Type: application/octet-stream
    ##   Size: 85.9 kB
    ## <ON DISK>  /var/folders/xv/k5cp232j1cn3y8kym53_nnkc0000gn/T//RtmplrW0oC/file1796d6859fc51.xlsx

``` r
df_raw <- read_excel(
  tf,
  sheet = 1,
  skip = 6,
  col_names = c(
    "region",
    "sub_region",
    "territory",
    "source",
    "org",
    "indicator",
    "2000",
    "2001",
    "2002",
    "2003",
    "2004",
    "2005",
    "2006",
    "2007",
    "2008",
    "2009",
    "2010",
    "2011",
    "2012"
  )
)
```

Let’s take a look at the head.

``` r
df_raw %>% head
```

    ## # A tibble: 6 x 19
    ##   region sub_region territory source org   indicator `2000` `2001` `2002`
    ##   <chr>  <chr>      <chr>     <chr>  <chr> <chr>      <dbl>  <dbl>  <dbl>
    ## 1 <NA>   <NA>       <NA>      <NA>   <NA>  <NA>        2000   2001   2002
    ## 2 Africa Eastern A… Burundi   PH     WHO   Rate          NA     NA     NA
    ## 3 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA     NA     NA
    ## 4 <NA>   <NA>       Comoros   PH     WHO   Rate          NA     NA     NA
    ## 5 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA     NA     NA
    ## 6 <NA>   <NA>       Djibouti  PH     WHO   Rate          NA     NA     NA
    ## # … with 10 more variables: `2003` <dbl>, `2004` <dbl>, `2005` <dbl>,
    ## #   `2006` <dbl>, `2007` <dbl>, `2008` <dbl>, `2009` <chr>, `2010` <chr>,
    ## #   `2011` <chr>, `2012` <chr>

We’ll handle the implicit values with *lag-filling*.

Let’s also inspect the tail.

``` r
df_raw %>% tail(20)
```

    ## # A tibble: 20 x 19
    ##    region sub_region territory source org   indicator `2000` `2001` `2002`
    ##    <chr>  <chr>      <chr>     <chr>  <chr> <chr>      <dbl>  <dbl>  <dbl>
    ##  1 <NA>   <NA>       Palau     PH     WHO   Rate          NA   NA     NA  
    ##  2 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA   NA     NA  
    ##  3 <NA>   Polynesia  Cook Isl… PH     WHO   Rate          NA   NA     NA  
    ##  4 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA   NA     NA  
    ##  5 <NA>   <NA>       French P… CJ     Nati… Rate          NA   NA     NA  
    ##  6 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA   NA     NA  
    ##  7 <NA>   <NA>       Niue      PH     WHO   Rate          NA   NA     NA  
    ##  8 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA   NA     NA  
    ##  9 <NA>   <NA>       Samoa     PH     WHO   Rate          NA   NA     NA  
    ## 10 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA   NA     NA  
    ## 11 <NA>   <NA>       Tonga     CJ     CTS/… Rate           1    6.1    6.1
    ## 12 <NA>   <NA>       <NA>      <NA>   <NA>  Count          1    6      6  
    ## 13 <NA>   <NA>       Tuvalu    PH     WHO   Rate          NA   NA     NA  
    ## 14 <NA>   <NA>       <NA>      <NA>   <NA>  Count         NA   NA     NA  
    ## 15 <NA>   <NA>       <NA>      <NA>   <NA>  <NA>          NA   NA     NA  
    ## 16 x due… <NA>       <NA>      <NA>   <NA>  <NA>          NA   NA     NA  
    ## 17 * CTS… <NA>       <NA>      <NA>   <NA>  <NA>          NA   NA     NA  
    ## 18 + In … <NA>       <NA>      <NA>   <NA>  <NA>          NA   NA     NA  
    ## 19 <NA>   <NA>       <NA>      <NA>   <NA>  <NA>          NA   NA     NA  
    ## 20 # Not… <NA>       <NA>      <NA>   <NA>  <NA>          NA   NA     NA  
    ## # … with 10 more variables: `2003` <dbl>, `2004` <dbl>, `2005` <dbl>,
    ## #   `2006` <dbl>, `2007` <dbl>, `2008` <dbl>, `2009` <chr>, `2010` <chr>,
    ## #   `2011` <chr>, `2012` <chr>

We’ll handle the notes in the `region` column with an `NA` replacement.

### Sheet notes

<!-- ------------------------- -->

**Issue**: A minor annoyance in these sheets is the addition of
footnotes. These are important for understanding the data and its
caveats, but they make the data less machine-readable.

**Solution**: I inspected the notes and looked for keywords. I then use
an `if_else` call to replace those instances with missing values.
Remember that `NA_character_` is the correct missing value for
character-type vectors.

``` r
## Detect notes and fill with NA
df_no_notes <-
  df_raw %>%
  mutate(
    region = if_else(str_detect(region, "estimate|data"), NA_character_, region)
  )
```

### Lag-filling

<!-- ------------------------- -->

**Issue**: One of the most obnoxious issues with Excel sheets is that
important labeling entries are often left blank: These values are
implicit by location. While fine for human-readable formats, these
implicit values need to be made explicit for a machine-readable format.

**Solution**: We will repeatedly fill the implicit columns with lagged
values.

``` r
## Choose variables to lag-fill
vars_lagged <- c("region", "sub_region", "territory", "source", "org")

## Trim head and notes
df_filled <-
  df_no_notes %>%
  slice(-1) %>%
  slice(-(n()-5:-n()))

## Helper function to count num rows w/ NA in vars_lagged
countna <- function(df) {
  df %>%
    filter_at(vars(!!!vars_lagged), any_vars(is.na(.))) %>%
    dim %>%
    .[[1]]
}

## Repeatedly lag-fill until NA's are gone
while (countna(df_filled) > 0) {
  df_filled <-
    df_filled %>%
    mutate_at(vars(!!!vars_lagged), ~if_else(is.na(.), lag(.), .))
}
```

### Tidying

<!-- ------------------------- -->

The final steps are fairly standard wrangling; ensure all the years are
integer values and pivot to tidy the data.

``` r
## Cast all the estimates
df_tmp <-
  df_filled %>%
  mutate_at(
    vars(`2000`:`2012`),
    as.numeric
  )
```

    ## Warning: NAs introduced by coercion
    
    ## Warning: NAs introduced by coercion
    
    ## Warning: NAs introduced by coercion
    
    ## Warning: NAs introduced by coercion

``` r
## Reshape
df_tmp <-
  df_tmp %>%
  pivot_longer(
    `2000`:`2012`,
    names_to = "year",
    values_to = "value",
    names_ptypes = list(year = integer())
  ) %>%
  pivot_wider(
    names_from = indicator,
    values_from = "value"
  ) %>%
  glimpse
```

    ## Observations: 2,847
    ## Variables: 8
    ## $ region     <chr> "Africa", "Africa", "Africa", "Africa", "Africa", "Af…
    ## $ sub_region <chr> "Eastern Africa", "Eastern Africa", "Eastern Africa",…
    ## $ territory  <chr> "Burundi", "Burundi", "Burundi", "Burundi", "Burundi"…
    ## $ source     <chr> "PH", "PH", "PH", "PH", "PH", "PH", "PH", "PH", "PH",…
    ## $ org        <chr> "WHO", "WHO", "WHO", "WHO", "WHO", "WHO", "WHO", "WHO…
    ## $ year       <int> 2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008,…
    ## $ Rate       <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, 8, NA…
    ## $ Count      <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, 790, …

``` r
## Final cleaning
df_undoc <-
  df_tmp %>%
  rename_all(str_to_lower)
```

Let’s check some summaries.

``` r
df_undoc %>% summary
```

    ##     region           sub_region         territory        
    ##  Length:2847        Length:2847        Length:2847       
    ##  Class :character   Class :character   Class :character  
    ##  Mode  :character   Mode  :character   Mode  :character  
    ##                                                          
    ##                                                          
    ##                                                          
    ##                                                          
    ##     source              org                 year           rate       
    ##  Length:2847        Length:2847        Min.   :2000   Min.   : 0.000  
    ##  Class :character   Class :character   1st Qu.:2003   1st Qu.: 1.500  
    ##  Mode  :character   Mode  :character   Median :2006   Median : 3.800  
    ##                                        Mean   :2006   Mean   : 8.941  
    ##                                        3rd Qu.:2009   3rd Qu.:10.200  
    ##                                        Max.   :2012   Max.   :91.400  
    ##                                                       NA's   :1242    
    ##      count        
    ##  Min.   :    0.0  
    ##  1st Qu.:   37.0  
    ##  Median :  201.5  
    ##  Mean   : 2070.6  
    ##  3rd Qu.:  873.2  
    ##  Max.   :50108.0  
    ##  NA's   :1251
