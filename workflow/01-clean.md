Clean data
================
Compiled at 2020-12-27 08:14:43 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "9fa9049e-5898-494b-9b1a-0175496b3975")
```

    ## here() starts at /Users/runner/work/covidStates/covidStates/workflow

``` r
library("conflicted")
library("projthis")
library("here")
library("readr")
library("dplyr")
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Will prefer [34mdplyr::filter[39m over any other package

The purpose of this document is to clean the data we imported in the
[previous step](00-import.md).

``` r
# create target directory to write *this* file's data: 
#  - all data written by this file should be written here
proj_create_dir_target(params$name)

# create accessor functions for data directories:
#  - get path to target directory: path_target("sample.csv")
#  - get path to previous data: path_data("00-import", "sample.csv")
path_target <- proj_path_target(params$name)
path_data <- proj_path_data(params$name)
```

## Read data

First, we will read in the data:

``` r
population_raw <- 
  read_csv(
    path_data("00-import", "population-states.csv"), 
    col_names = c("abbreviation", "year", "population"),
    col_types = cols(abbreviation = "c", year = "d", population = "d")
  ) %>%
  print()
```

    ## [90m# A tibble: 6,020 x 3[39m
    ##    abbreviation  year population
    ##    [3m[90m<chr>[39m[23m        [3m[90m<dbl>[39m[23m      [3m[90m<dbl>[39m[23m
    ## [90m 1[39m AK            [4m1[24m950     [4m1[24m[4m3[24m[4m5[24m000
    ## [90m 2[39m AK            [4m1[24m951     [4m1[24m[4m5[24m[4m8[24m000
    ## [90m 3[39m AK            [4m1[24m952     [4m1[24m[4m8[24m[4m9[24m000
    ## [90m 4[39m AK            [4m1[24m953     [4m2[24m[4m0[24m[4m5[24m000
    ## [90m 5[39m AK            [4m1[24m954     [4m2[24m[4m1[24m[4m5[24m000
    ## [90m 6[39m AK            [4m1[24m955     [4m2[24m[4m2[24m[4m2[24m000
    ## [90m 7[39m AK            [4m1[24m956     [4m2[24m[4m2[24m[4m4[24m000
    ## [90m 8[39m AK            [4m1[24m957     [4m2[24m[4m3[24m[4m1[24m000
    ## [90m 9[39m AK            [4m1[24m958     [4m2[24m[4m2[24m[4m4[24m000
    ## [90m10[39m AK            [4m1[24m959     [4m2[24m[4m2[24m[4m4[24m000
    ## [90m# … with 6,010 more rows[39m

``` r
covid_raw <- 
  read_csv(
    path_data("00-import", "covid-states.csv"),
    col_types = cols(
      date = col_date(format = ""),
      state = col_character(),
      fips = col_character(),
      cases = col_double(),
      deaths = col_double()
    )
  ) %>%
  print()
```

    ## [90m# A tibble: 16,459 x 5[39m
    ##    date       state      fips  cases deaths
    ##    [3m[90m<date>[39m[23m     [3m[90m<chr>[39m[23m      [3m[90m<chr>[39m[23m [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-01-21 Washington 53        1      0
    ## [90m 2[39m 2020-01-22 Washington 53        1      0
    ## [90m 3[39m 2020-01-23 Washington 53        1      0
    ## [90m 4[39m 2020-01-24 Illinois   17        1      0
    ## [90m 5[39m 2020-01-24 Washington 53        1      0
    ## [90m 6[39m 2020-01-25 California 06        1      0
    ## [90m 7[39m 2020-01-25 Illinois   17        1      0
    ## [90m 8[39m 2020-01-25 Washington 53        1      0
    ## [90m 9[39m 2020-01-26 Arizona    04        1      0
    ## [90m10[39m 2020-01-26 California 06        2      0
    ## [90m# … with 16,449 more rows[39m

Reading in the data using `readr::read_csv()`, we use the `cols()`
function with the `col_types` argument to assert the types of the
columns that we parse. If there is some data that is *not* of this sort,
we want to know about it.

## Wrangle data

It looks like we could use a way to link state names with their
abbreviations:

``` r
states <- 
  tibble(
    state = c(state.name, "District of Columbia"),
    abbreviation = c(state.abb, "DC")
  ) %>%
  print()
```

    ## [90m# A tibble: 51 x 2[39m
    ##    state       abbreviation
    ##    [3m[90m<chr>[39m[23m       [3m[90m<chr>[39m[23m       
    ## [90m 1[39m Alabama     AL          
    ## [90m 2[39m Alaska      AK          
    ## [90m 3[39m Arizona     AZ          
    ## [90m 4[39m Arkansas    AR          
    ## [90m 5[39m California  CA          
    ## [90m 6[39m Colorado    CO          
    ## [90m 7[39m Connecticut CT          
    ## [90m 8[39m Delaware    DE          
    ## [90m 9[39m Florida     FL          
    ## [90m10[39m Georgia     GA          
    ## [90m# … with 41 more rows[39m

We want to work with:

  - 2019 population
  - 50 US states, plus District of Columbia

<!-- end list -->

``` r
population <- 
  population_raw %>%
  filter(year == 2019) %>%
  right_join(states, by = "abbreviation") %>%
  select(state, population) %>%
  print()
```

    ## [90m# A tibble: 51 x 2[39m
    ##    state                population
    ##    [3m[90m<chr>[39m[23m                     [3m[90m<dbl>[39m[23m
    ## [90m 1[39m Alaska                   [4m7[24m[4m3[24m[4m1[24m545
    ## [90m 2[39m Alabama                 4[4m9[24m[4m0[24m[4m3[24m185
    ## [90m 3[39m Arkansas                3[4m0[24m[4m1[24m[4m7[24m804
    ## [90m 4[39m Arizona                 7[4m2[24m[4m7[24m[4m8[24m717
    ## [90m 5[39m California             39[4m5[24m[4m1[24m[4m2[24m223
    ## [90m 6[39m Colorado                5[4m7[24m[4m5[24m[4m8[24m736
    ## [90m 7[39m Connecticut             3[4m5[24m[4m6[24m[4m5[24m287
    ## [90m 8[39m District of Columbia     [4m7[24m[4m0[24m[4m5[24m749
    ## [90m 9[39m Delaware                 [4m9[24m[4m7[24m[4m3[24m764
    ## [90m10[39m Florida                21[4m4[24m[4m7[24m[4m7[24m737
    ## [90m# … with 41 more rows[39m

``` r
covid <- 
  covid_raw %>%
  filter(state %in% states$state) %>%
  print()
```

    ## [90m# A tibble: 15,321 x 5[39m
    ##    date       state      fips  cases deaths
    ##    [3m[90m<date>[39m[23m     [3m[90m<chr>[39m[23m      [3m[90m<chr>[39m[23m [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-01-21 Washington 53        1      0
    ## [90m 2[39m 2020-01-22 Washington 53        1      0
    ## [90m 3[39m 2020-01-23 Washington 53        1      0
    ## [90m 4[39m 2020-01-24 Illinois   17        1      0
    ## [90m 5[39m 2020-01-24 Washington 53        1      0
    ## [90m 6[39m 2020-01-25 California 06        1      0
    ## [90m 7[39m 2020-01-25 Illinois   17        1      0
    ## [90m 8[39m 2020-01-25 Washington 53        1      0
    ## [90m 9[39m 2020-01-26 Arizona    04        1      0
    ## [90m10[39m 2020-01-26 California 06        2      0
    ## [90m# … with 15,311 more rows[39m

We can see which states have the most cases, also verifying the recency
of the data:

``` r
covid %>%
  filter(date == max(date)) %>%
  arrange(desc(cases))
```

    ## [90m# A tibble: 51 x 5[39m
    ##    date       state        fips    cases deaths
    ##    [3m[90m<date>[39m[23m     [3m[90m<chr>[39m[23m        [3m[90m<chr>[39m[23m   [3m[90m<dbl>[39m[23m  [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-12-26 California   06    2[4m1[24m[4m2[24m[4m9[24m566  [4m2[24m[4m4[24m220
    ## [90m 2[39m 2020-12-26 Texas        48    1[4m6[24m[4m6[24m[4m8[24m843  [4m2[24m[4m7[24m062
    ## [90m 3[39m 2020-12-26 Florida      12    1[4m2[24m[4m6[24m[4m4[24m580  [4m2[24m[4m1[24m134
    ## [90m 4[39m 2020-12-26 Illinois     17     [4m9[24m[4m3[24m[4m5[24m849  [4m1[24m[4m7[24m225
    ## [90m 5[39m 2020-12-26 New York     36     [4m9[24m[4m2[24m[4m0[24m171  [4m3[24m[4m6[24m870
    ## [90m 6[39m 2020-12-26 Ohio         39     [4m6[24m[4m6[24m[4m4[24m668   [4m8[24m476
    ## [90m 7[39m 2020-12-26 Pennsylvania 42     [4m6[24m[4m0[24m[4m9[24m682  [4m1[24m[4m4[24m915
    ## [90m 8[39m 2020-12-26 Georgia      13     [4m6[24m[4m0[24m[4m7[24m133  [4m1[24m[4m0[24m352
    ## [90m 9[39m 2020-12-26 Tennessee    47     [4m5[24m[4m4[24m[4m6[24m245   [4m6[24m382
    ## [90m10[39m 2020-12-26 Michigan     26     [4m5[24m[4m1[24m[4m5[24m484  [4m1[24m[4m2[24m680
    ## [90m# … with 41 more rows[39m

## Write data

Let’s write out our population and covid data:

``` r
write_csv(population, path_target("population.csv"))
```

``` r
write_csv(covid, path_target("covid.csv"))
```

## Files written

These files have been written to `data/01-clean`:

``` r
proj_dir_info(path_target())
```

    ## [90m# A tibble: 2 x 4[39m
    ##   path           type         size modification_time  
    ##   [3m[90m<fs::path>[39m[23m     [3m[90m<fct>[39m[23m [3m[90m<fs::bytes>[39m[23m [3m[90m<dttm>[39m[23m             
    ## [90m1[39m covid.csv      file         503K 2020-12-27 [90m08:14:44[39m
    ## [90m2[39m population.csv file          920 2020-12-27 [90m08:14:44[39m
