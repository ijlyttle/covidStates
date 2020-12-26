Analyze data
================
Compiled at 2020-12-26 20:04:22 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "a4069103-4402-4559-ba03-cca3df086442")
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

``` r
conflict_prefer("lag", "dplyr")
```

    ## [conflicted] Will prefer dplyr::lag over any other package

The purpose of this document is to create some state-based maps that
show the current trajectory of COVID-19 cases. There will be two maps:

-   seven-day average of newly-reported cases
-   change in newly-reported cases vs. previous seven days

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

``` r
population <- 
  read_csv(
    path_data("01-clean", "population.csv"),
    col_types = cols(state = "c", population = "d")
  ) %>%
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
  read_csv(
    path_data("01-clean", "covid.csv"),
    col_types = cols(
      date = "D", 
      state = "c", 
      fips = "c", 
      cases = "d", 
      deaths = "d"
    )
  ) %>%
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

``` r
growth <- function(x) {
  # week-over-week growth, normalized by previous week
  
  # use "+ 1" in denominator to avoid division by zero
  result <- (x - dplyr::lag(x, 7)) / (dplyr::lag(x, 7) + 1) 
  
  # three digits should be more than enough
  round(result, 3)
}

per_100k <- function(x, pop) {
  result <- x / (pop / 1e5)
  
  round(result, 3)
}

covid_week <-
  covid %>%
  left_join(population, by = "state") %>%
  group_by(state) %>%
  arrange(date) %>%
  transmute(
    date,
    state,
    cases, 
    deaths,
    cases_week = cases - lag(cases, 7),
    deaths_week = deaths - lag(deaths, 7),
    cases_week_per100k = per_100k(cases_week, population),
    deaths_week_per100k = per_100k(deaths_week, population),
    cases_week_growth = growth(cases_week),
    deaths_week_growth = growth(deaths_week)
  ) %>%
  print()
```

    ## # A tibble: 15,270 x 10
    ## # Groups:   state [51]
    ##    date       state cases deaths cases_week deaths_week cases_week_per1…
    ##    <date>     <chr> <dbl>  <dbl>      <dbl>       <dbl>            <dbl>
    ##  1 2020-01-21 Wash…     1      0         NA          NA               NA
    ##  2 2020-01-22 Wash…     1      0         NA          NA               NA
    ##  3 2020-01-23 Wash…     1      0         NA          NA               NA
    ##  4 2020-01-24 Illi…     1      0         NA          NA               NA
    ##  5 2020-01-24 Wash…     1      0         NA          NA               NA
    ##  6 2020-01-25 Cali…     1      0         NA          NA               NA
    ##  7 2020-01-25 Illi…     1      0         NA          NA               NA
    ##  8 2020-01-25 Wash…     1      0         NA          NA               NA
    ##  9 2020-01-26 Ariz…     1      0         NA          NA               NA
    ## 10 2020-01-26 Cali…     2      0         NA          NA               NA
    ## # … with 15,260 more rows, and 3 more variables: deaths_week_per100k <dbl>,
    ## #   cases_week_growth <dbl>, deaths_week_growth <dbl>

It might also be useful to have files for the most-recent day, each for
cases and deaths.

``` r
covid_recent_cases <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("cases")) %>%
  arrange(desc(cases_week_per100k)) %>%
  print()
```

    ## # A tibble: 51 x 6
    ## # Groups:   state [51]
    ##    date       state         cases cases_week cases_week_per100k cases_week_grow…
    ##    <date>     <chr>         <dbl>      <dbl>              <dbl>            <dbl>
    ##  1 2020-12-25 California  2064511     254926               645.           -0.094
    ##  2 2020-12-25 Tennessee    532375      42071               616.           -0.346
    ##  3 2020-12-25 Arizona      486993      42955               590.           -0.064
    ##  4 2020-12-25 Alabama      342426      26743               545.           -0.006
    ##  5 2020-12-25 Oklahoma     272553      20793               525.           -0.072
    ##  6 2020-12-25 Arkansas     213267      15846               525.            0.003
    ##  7 2020-12-25 Indiana      491125      35097               521.           -0.145
    ##  8 2020-12-25 West Virgi…   78836       9085               507.           -0.003
    ##  9 2020-12-25 Delaware      53653       4544               467.           -0.141
    ## 10 2020-12-25 Mississippi  204178      13767               463.           -0.09 
    ## # … with 41 more rows

``` r
covid_recent_deaths <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("deaths")) %>%
  arrange(desc(deaths_week_per100k)) %>%
  print()
```

    ## # A tibble: 51 x 6
    ## # Groups:   state [51]
    ##    date       state      deaths deaths_week deaths_week_per10… deaths_week_grow…
    ##    <date>     <chr>       <dbl>       <dbl>              <dbl>             <dbl>
    ##  1 2020-12-25 South Dak…   1430         100              11.3             -0.165
    ##  2 2020-12-25 Arkansas     3438         299               9.91             0.132
    ##  3 2020-12-25 Pennsylva…  14892        1228               9.59            -0.101
    ##  4 2020-12-25 Iowa         3744         293               9.29             0.135
    ##  5 2020-12-25 West Virg…   1247         156               8.70             0.019
    ##  6 2020-12-25 New Mexico   2309         181               8.63            -0.242
    ##  7 2020-12-25 Arizona      8409         590               8.11             0.037
    ##  8 2020-12-25 Alabama      4680         384               7.83             0.825
    ##  9 2020-12-25 Missouri     5633         463               7.54            -0.006
    ## 10 2020-12-25 Indiana      7770         505               7.50            -0.147
    ## # … with 41 more rows

## Write data

``` r
write_csv(covid_week, path_target("covid_week.csv"))
write_csv(covid_recent_cases, path_target("covid_recent_cases.csv"))
write_csv(covid_recent_deaths, path_target("covid_recent_deaths.csv"))
```

## Files written

These files have been written to `data/02-analyze`:

``` r
proj_dir_info(path_target())
```

    ## # A tibble: 3 x 4
    ##   path                    type         size modification_time  
    ##   <fs::path>              <fct> <fs::bytes> <dttm>             
    ## 1 covid_recent_cases.csv  file        2.44K 2020-12-26 20:04:27
    ## 2 covid_recent_deaths.csv file        2.15K 2020-12-26 20:04:27
    ## 3 covid_week.csv          file      938.06K 2020-12-26 20:04:27
