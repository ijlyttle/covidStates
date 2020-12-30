Clean data
================
Compiled at 2020-12-30 01:00:02 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "9fa9049e-5898-494b-9b1a-0175496b3975")
```

The purpose of this document is to clean the data we imported in the
[previous step](00-import.md).

``` r
library("conflicted")
library("readr")
library("dplyr")

conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Will prefer dplyr::filter over any other package

``` r
# create or *empty* the target directory, used to write this file's data: 
projthis::proj_create_dir_target(params$name)

# function to get path to target directory: path_target("sample.csv")
path_target <- projthis::proj_path_target(params$name)

# function to get path to previous data: path_source("00-import", "sample.csv")
path_source <- projthis::proj_path_source(params$name)
```

## Read data

First, we will read in the data:

``` r
population_raw <- 
  read_csv(
    path_source("00-import", "population-states.csv"), 
    col_names = c("abbreviation", "year", "population"),
    col_types = cols(abbreviation = "c", year = "d", population = "d")
  ) %>%
  print()
```

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
    path_source("00-import", "covid-states.csv"),
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

    ## # A tibble: 16,569 x 5
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
    ## # … with 16,559 more rows

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

    ## # A tibble: 15,423 x 5
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
    ## # … with 15,413 more rows

We can see which states have the most cases, also verifying the recency
of the data:

``` r
covid %>%
  filter(date == max(date)) %>%
  arrange(desc(cases))
```

    ## # A tibble: 51 x 5
    ##    date       state          fips    cases deaths
    ##    <date>     <chr>          <chr>   <dbl>  <dbl>
    ##  1 2020-12-28 California     06    2208953  24529
    ##  2 2020-12-28 Texas          48    1711986  27245
    ##  3 2020-12-28 Florida        12    1280169  21307
    ##  4 2020-12-28 Illinois       17     944402  17476
    ##  5 2020-12-28 New York       36     938710  37118
    ##  6 2020-12-28 Ohio           39     675044   8571
    ##  7 2020-12-28 Pennsylvania   42     619425  15114
    ##  8 2020-12-28 Georgia        13     614716  10363
    ##  9 2020-12-28 Tennessee      47     552794   6524
    ## 10 2020-12-28 North Carolina 37     521990   6601
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
projthis::proj_dir_info(path_target())
```

    ## # A tibble: 2 x 4
    ##   path           type         size modification_time  
    ##   <fs::path>     <fct> <fs::bytes> <dttm>             
    ## 1 covid.csv      file         507K 2020-12-30 01:00:03
    ## 2 population.csv file          920 2020-12-30 01:00:03
