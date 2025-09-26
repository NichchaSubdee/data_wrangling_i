Tidy data
================

``` r
library(tidyr)
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ purrr     1.1.0
    ## ✔ forcats   1.0.0     ✔ readr     2.1.5
    ## ✔ ggplot2   4.0.0     ✔ stringr   1.5.1
    ## ✔ lubridate 1.9.4     ✔ tibble    3.3.0
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
options(tibble.print_min = 5)
```

## `pivot_longer`

Load the PULSE data

``` r
pulse_df = 
  haven::read_sas("C:/Users/prize/Dropbox/Prize/CUIMC_MPH2YR/Fall 2025/Data Science I/Week 2/data_manipulation/tidy data/data/public_pulse_data.sas7bdat") |> 
  janitor::clean_names()

pulse_df
```

    ## # A tibble: 1,087 × 7
    ##      id   age sex   bdi_score_bl bdi_score_01m bdi_score_06m bdi_score_12m
    ##   <dbl> <dbl> <chr>        <dbl>         <dbl>         <dbl>         <dbl>
    ## 1 10003  48.0 male             7             1             2             0
    ## 2 10015  72.5 male             6            NA            NA            NA
    ## 3 10022  58.5 male            14             3             8            NA
    ## 4 10026  72.7 male            20             6            18            16
    ## 5 10035  60.4 male             4             0             1             2
    ## # ℹ 1,082 more rows

Wide format to long format…

``` r
library(tidyr)

pulse_tidy_df = 
  pivot_longer(
    pulse_df, 
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    values_to = "bdi")

pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##      id   age sex   visit           bdi
    ##   <dbl> <dbl> <chr> <chr>         <dbl>
    ## 1 10003  48.0 male  bdi_score_bl      7
    ## 2 10003  48.0 male  bdi_score_01m     1
    ## 3 10003  48.0 male  bdi_score_06m     2
    ## 4 10003  48.0 male  bdi_score_12m     0
    ## 5 10015  72.5 male  bdi_score_bl      6
    ## # ℹ 4,343 more rows

names_to – create a new variable to put bdi_score_bl

clean more prefix…

``` r
pulse_tidy_df = 
  pivot_longer(
    pulse_df, 
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi")

pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##      id   age sex   visit   bdi
    ##   <dbl> <dbl> <chr> <chr> <dbl>
    ## 1 10003  48.0 male  bl        7
    ## 2 10003  48.0 male  01m       1
    ## 3 10003  48.0 male  06m       2
    ## 4 10003  48.0 male  12m       0
    ## 5 10015  72.5 male  bl        6
    ## # ℹ 4,343 more rows

Rewrite, combine, and extend (to add a mutate)

``` r
library(dplyr)

pulse_data = 
  haven::read_sas("C:/Users/prize/Dropbox/Prize/CUIMC_MPH2YR/Fall 2025/Data Science I/Week 2/data_manipulation/tidy data/data/public_pulse_data.sas7bdat") |>
  janitor::clean_names() |>
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) |>
  relocate(id,visit) |> 
  mutate(visit = recode(visit, "bl" = "00m"))
```

## `pivot_wider`

Make up some data!

``` r
analysis_result = 
  tibble(
    group = c("treatment", "treament", "placebo", "placebo"),
    time = c("pre", "post", "pre", "post"),
    mean = c(4, 8, 3.5, 4)
  )

analysis_result
```

    ## # A tibble: 4 × 3
    ##   group     time   mean
    ##   <chr>     <chr> <dbl>
    ## 1 treatment pre     4  
    ## 2 treament  post    8  
    ## 3 placebo   pre     3.5
    ## 4 placebo   post    4

``` r
pivot_wider(
    analysis_result,
    names_from = "time",
    values_from = "mean"
  )
```

    ## # A tibble: 3 × 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4      NA
    ## 2 treament   NA       8
    ## 3 placebo     3.5     4

## Binding rows

Using the LotR data. First step: import each table.

``` r
fellowship_ring = 
  readxl::read_excel("C:/Users/prize/Dropbox/Prize/CUIMC_MPH2YR/Fall 2025/Data Science I/Week 2/data_manipulation/tidy data/data/LotR_Words.xlsx", range = "B3:D6") |>
  mutate(movie = "fellowship_ring")


two_towers = 
  readxl::read_excel("C:/Users/prize/Dropbox/Prize/CUIMC_MPH2YR/Fall 2025/Data Science I/Week 2/data_manipulation/tidy data/data/LotR_Words.xlsx", range = "F3:h6") |>
  mutate(movie = "two_towers")


Return_king = 
  readxl::read_excel("C:/Users/prize/Dropbox/Prize/CUIMC_MPH2YR/Fall 2025/Data Science I/Week 2/data_manipulation/tidy data/data/LotR_Words.xlsx", range = "J3:L6") |>
  mutate(movie = "return_king")
```

Bind all the rows together

``` r
lotr_tidy =
  bind_rows(fellowship_ring, two_towers, Return_king) |> 
  janitor::clean_names() |> 
  relocate(movie) |> 
  pivot_longer(
    female:male,
    names_to = "gender",
    values_to = "words"
  )

lotr_tidy
```

    ## # A tibble: 18 × 4
    ##    movie           race   gender words
    ##    <chr>           <chr>  <chr>  <dbl>
    ##  1 fellowship_ring Elf    female  1229
    ##  2 fellowship_ring Elf    male     971
    ##  3 fellowship_ring Hobbit female    14
    ##  4 fellowship_ring Hobbit male    3644
    ##  5 fellowship_ring Man    female     0
    ##  6 fellowship_ring Man    male    1995
    ##  7 two_towers      Elf    female   331
    ##  8 two_towers      Elf    male     513
    ##  9 two_towers      Hobbit female     0
    ## 10 two_towers      Hobbit male    2463
    ## 11 two_towers      Man    female   401
    ## 12 two_towers      Man    male    3589
    ## 13 return_king     Elf    female   183
    ## 14 return_king     Elf    male     510
    ## 15 return_king     Hobbit female     2
    ## 16 return_king     Hobbit male    2673
    ## 17 return_king     Man    female   268
    ## 18 return_king     Man    male    2459
