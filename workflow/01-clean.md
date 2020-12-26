Clean data
================
Compiled at 2020-12-26 15:38:07 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "9fa9049e-5898-494b-9b1a-0175496b3975")
```

    ## here() starts at /Users/sesa19001/Documents/repos/public/covidStates/workflow

``` r
library("conflicted")
library("projthis")
library("here")
library("readr")
library("dplyr")

conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Will prefer dplyr::filter over any other package

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
    col_names = c("abbreviation", "year", "population")
  ) %>%
  print()
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   abbreviation = col_character(),
    ##   year = col_double(),
    ##   population = col_double()
    ## )

    ## # A tibble: 6,020 x 3
    ##    abbreviation  year population
    ##    <chr>        <dbl>      <dbl>
    ##  1 AK            1950     135000
    ##  2 AK            1951     158000
    ##  3 AK            1952     189000
    ##  4 AK            1953     205000
    ##  5 AK            1954     215000
    ##  6 AK            1955     222000
    ##  7 AK            1956     224000
    ##  8 AK            1957     231000
    ##  9 AK            1958     224000
    ## 10 AK            1959     224000
    ## # … with 6,010 more rows

``` r
covid_raw <- 
  read_csv(
    path_data("00-import", "covid-states.csv")
  ) %>%
  print()
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   date = col_date(format = ""),
    ##   state = col_character(),
    ##   fips = col_character(),
    ##   cases = col_double(),
    ##   deaths = col_double()
    ## )

    ## # A tibble: 16,404 x 5
    ##    date       state      fips  cases deaths
    ##    <date>     <chr>      <chr> <dbl>  <dbl>
    ##  1 2020-01-21 Washington 53        1      0
    ##  2 2020-01-22 Washington 53        1      0
    ##  3 2020-01-23 Washington 53        1      0
    ##  4 2020-01-24 Illinois   17        1      0
    ##  5 2020-01-24 Washington 53        1      0
    ##  6 2020-01-25 California 06        1      0
    ##  7 2020-01-25 Illinois   17        1      0
    ##  8 2020-01-25 Washington 53        1      0
    ##  9 2020-01-26 Arizona    04        1      0
    ## 10 2020-01-26 California 06        2      0
    ## # … with 16,394 more rows

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

    ## # A tibble: 51 x 2
    ##    state       abbreviation
    ##    <chr>       <chr>       
    ##  1 Alabama     AL          
    ##  2 Alaska      AK          
    ##  3 Arizona     AZ          
    ##  4 Arkansas    AR          
    ##  5 California  CA          
    ##  6 Colorado    CO          
    ##  7 Connecticut CT          
    ##  8 Delaware    DE          
    ##  9 Florida     FL          
    ## 10 Georgia     GA          
    ## # … with 41 more rows

We want to work with:

-   2019 population
-   50 US states, plus District of Columbia

``` r
population <- 
  population_raw %>%
  filter(year == 2019) %>%
  right_join(states, by = "abbreviation") %>%
  select(state, population) %>%
  print()
```

    ## # A tibble: 51 x 2
    ##    state                population
    ##    <chr>                     <dbl>
    ##  1 Alaska                   731545
    ##  2 Alabama                 4903185
    ##  3 Arkansas                3017804
    ##  4 Arizona                 7278717
    ##  5 California             39512223
    ##  6 Colorado                5758736
    ##  7 Connecticut             3565287
    ##  8 District of Columbia     705749
    ##  9 Delaware                 973764
    ## 10 Florida                21477737
    ## # … with 41 more rows

``` r
covid <- 
  covid_raw %>%
  filter(state %in% states$state) %>%
  print()
```

    ## # A tibble: 15,270 x 5
    ##    date       state      fips  cases deaths
    ##    <date>     <chr>      <chr> <dbl>  <dbl>
    ##  1 2020-01-21 Washington 53        1      0
    ##  2 2020-01-22 Washington 53        1      0
    ##  3 2020-01-23 Washington 53        1      0
    ##  4 2020-01-24 Illinois   17        1      0
    ##  5 2020-01-24 Washington 53        1      0
    ##  6 2020-01-25 California 06        1      0
    ##  7 2020-01-25 Illinois   17        1      0
    ##  8 2020-01-25 Washington 53        1      0
    ##  9 2020-01-26 Arizona    04        1      0
    ## 10 2020-01-26 California 06        2      0
    ## # … with 15,260 more rows

We can see which states have the most cases, also verifying the recency
of the data:

``` r
covid %>%
  filter(date == max(date)) %>%
  arrange(desc(cases))
```

    ## # A tibble: 51 x 5
    ##    date       state        fips    cases deaths
    ##    <date>     <chr>        <chr>   <dbl>  <dbl>
    ##  1 2020-12-25 California   06    2064511  23964
    ##  2 2020-12-25 Texas        48    1663094  27042
    ##  3 2020-12-25 Florida      12    1247538  20994
    ##  4 2020-12-25 Illinois     17     932427  17155
    ##  5 2020-12-25 New York     36     909123  36739
    ##  6 2020-12-25 Ohio         39     653650   8456
    ##  7 2020-12-25 Georgia      13     603246  10303
    ##  8 2020-12-25 Pennsylvania 42     602605  14892
    ##  9 2020-12-25 Tennessee    47     532375   6367
    ## 10 2020-12-25 Michigan     26     508171  12406
    ## # … with 41 more rows

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

    ## # A tibble: 2 x 4
    ##   path           type         size modification_time  
    ##   <fs::path>     <fct> <fs::bytes> <dttm>             
    ## 1 covid.csv      file         502K 2020-12-26 15:38:08
    ## 2 population.csv file          920 2020-12-26 15:38:08