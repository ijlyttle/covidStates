Analyze data
================
Compiled at 2021-12-06 08:19:21 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "a4069103-4402-4559-ba03-cca3df086442")
```

The purpose of this document is to create some state-based maps that
show the current trajectory of COVID-19 cases. There will be two maps:

  - seven-day average of newly-reported cases
  - change in newly-reported cases vs. previous seven days

<!-- end list -->

``` r
library("conflicted")
library("readr")
library("dplyr")
library("albersusa")
library("ggplot2")
library("glue")

conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Will prefer dplyr::filter over any other package

``` r
conflict_prefer("lag", "dplyr")
```

    ## [conflicted] Will prefer dplyr::lag over any other package

``` r
# create or *empty* the target directory, used to write this file's data: 
projthis::proj_create_dir_target(params$name, clean = TRUE)

# function to get path to target directory: path_target("sample.csv")
path_target <- projthis::proj_path_target(params$name)

# function to get path to previous data: path_source("00-import", "sample.csv")
path_source <- projthis::proj_path_source(params$name)
```

    ## ℹ Reading workflow configuration from '/Users/runner/work/covidStates/covidStates/workflow/_projthis.yml'

## Read data

``` r
population <- 
  read_csv(
    path_source("01-clean", "population.csv"),
    col_types = cols(state = "c", population = "d")
  ) %>%
  print()
```

    ## # A tibble: 51 × 2
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
    path_source("01-clean", "covid.csv"),
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

    ## # A tibble: 32,865 × 5
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
    ## # … with 32,855 more rows

## Wrangle data

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
    cases_total = cases, 
    cases_total_per100k = per_100k(cases_total, population),
    cases_avg_week = (cases - lag(cases_total, 7)) / 7,
    cases_avg_week_per100k = per_100k(cases_avg_week, population),
    cases_week_growth = growth(cases_avg_week),
    deaths_total = deaths,
    deaths_total_per100k = per_100k(deaths_total, population),
    deaths_avg_week = (deaths - lag(deaths_total, 7)) / 7,
    deaths_avg_week_per100k = per_100k(deaths_avg_week, population),
    deaths_week_growth = growth(deaths_avg_week)
  ) %>%
  print()
```

    ## # A tibble: 32,865 × 12
    ## # Groups:   state [51]
    ##    date       state cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    <date>     <chr>       <dbl>            <dbl>          <dbl>            <dbl>
    ##  1 2020-01-21 Wash…           1            0.013             NA               NA
    ##  2 2020-01-22 Wash…           1            0.013             NA               NA
    ##  3 2020-01-23 Wash…           1            0.013             NA               NA
    ##  4 2020-01-24 Illi…           1            0.008             NA               NA
    ##  5 2020-01-24 Wash…           1            0.013             NA               NA
    ##  6 2020-01-25 Cali…           1            0.003             NA               NA
    ##  7 2020-01-25 Illi…           1            0.008             NA               NA
    ##  8 2020-01-25 Wash…           1            0.013             NA               NA
    ##  9 2020-01-26 Ariz…           1            0.014             NA               NA
    ## 10 2020-01-26 Cali…           2            0.005             NA               NA
    ## # … with 32,855 more rows, and 6 more variables: cases_week_growth <dbl>,
    ## #   deaths_total <dbl>, deaths_total_per100k <dbl>, deaths_avg_week <dbl>,
    ## #   deaths_avg_week_per100k <dbl>, deaths_week_growth <dbl>

It might also be useful to have files for the most-recent day, each for
cases and deaths.

``` r
covid_recent_cases <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("cases")) %>%
  arrange(desc(cases_avg_week_per100k)) %>%
  print()
```

    ## # A tibble: 51 × 7
    ## # Groups:   state [51]
    ##    date       state cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    <date>     <chr>       <dbl>            <dbl>          <dbl>            <dbl>
    ##  1 2021-12-05 Verm…       52889            8476.           681.            109. 
    ##  2 2021-12-05 New …      165574           12177.          1409             104. 
    ##  3 2021-12-05 Mich…     1514467           15165.          9988.            100. 
    ##  4 2021-12-05 Rhod…      194858           18394.           900.             84.9
    ##  5 2021-12-05 New …      320520           15286.          1767.             84.3
    ##  6 2021-12-05 Minn…      927055           16438.          4529.             80.3
    ##  7 2021-12-05 West…      300660           16777.          1393              77.7
    ##  8 2021-12-05 Wisc…      997712           17136.          4185.             71.9
    ##  9 2021-12-05 Indi…     1121965           16666.          4839.             71.9
    ## 10 2021-12-05 Kans…      475471           16321.          1903.             65.3
    ## # … with 41 more rows, and 1 more variable: cases_week_growth <dbl>

``` r
covid_recent_deaths <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("deaths")) %>%
  arrange(desc(deaths_avg_week_per100k)) %>%
  print()
```

    ## # A tibble: 51 × 7
    ## # Groups:   state [51]
    ##    date       state         deaths_total deaths_total_per100k deaths_avg_week
    ##    <date>     <chr>                <dbl>                <dbl>           <dbl>
    ##  1 2021-12-05 Wyoming               1428                 247.           11.6 
    ##  2 2021-12-05 Montana               2740                 256.           15.4 
    ##  3 2021-12-05 Oklahoma             11269                 285.           55.7 
    ##  4 2021-12-05 Michigan             26077                 261.          123.  
    ##  5 2021-12-05 West Virginia         4962                 277.           20.7 
    ##  6 2021-12-05 Kentucky             11210                 251.           44.3 
    ##  7 2021-12-05 Tennessee            17254                 253.           61.1 
    ##  8 2021-12-05 North Dakota          1953                 256.            5.43
    ##  9 2021-12-05 Arizona              22589                 310.           51.4 
    ## 10 2021-12-05 Minnesota             9740                 173.           39.7 
    ## # … with 41 more rows, and 2 more variables: deaths_avg_week_per100k <dbl>,
    ## #   deaths_week_growth <dbl>

## Plot data

Let’s make some choropleth maps using
[ggplot2](https://ggplot2.tidyverse.org/).

``` r
map_recent_cases <- 
  usa_sf("laea") %>%
  left_join(covid_recent_cases, by = c(name = "state"))

date <- max(covid_recent_cases$date)

gg_cases <-
  ggplot(map_recent_cases, aes(fill = cases_avg_week_per100k)) +
  geom_sf(size = 0.25, color = "white") + 
  scale_fill_distiller(
    palette = "Oranges", 
    direction = 1,
    limits = c(0, NA)
  ) +
  labs(
    title = glue("Newly-reported COVID-19 cases, seven-day average as of {date}"),
    subtitle = "Data from the New York Times",
    fill = "cases\nper 100k"
  ) +
  theme_void() + 
  theme(
    legend.text.align = 1, # right-justify
    plot.background = element_rect(fill = "#ffffff")
  )

gg_cases
```

![](02-analyze_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
# see https://github.com/tidyverse/ggplot2/issues/3738#issuecomment-583750802
mid_rescaler <- function(mid = 0) {
  function(x, to = c(0, 1), from = range(x, na.rm = TRUE)) {
    scales::rescale_mid(x, to, from, mid)
  }
}

gg_change <-
  ggplot(map_recent_cases, aes(fill = cases_week_growth)) +
  geom_sf(size = 0.25, color = "white") + 
  scale_fill_distiller(
    palette = "PuOr", 
    rescaler = mid_rescaler(),
    labels = scales::label_percent()
  ) +
  labs(
    title = glue("Week-over-week change in reported COVID-19 cases, as of {date}"),
    subtitle = "Data from the New York Times",
    fill = "change"
  ) +
  theme_void() +
  theme(
    legend.text.align = 1, # right-justify
    plot.background = element_rect(fill = "#ffffff")
  )

gg_change
```

![](02-analyze_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Write data

``` r
write_csv(covid_week, path_target("covid_week.csv"))
write_csv(covid_recent_cases, path_target("covid_recent_cases.csv"))
write_csv(covid_recent_deaths, path_target("covid_recent_deaths.csv"))

ggsave(path_target("cases.png"), plot = gg_cases, width = 7, height = 5)
ggsave(path_target("change.png"), plot = gg_change, width = 7, height = 5)
```

## Files written

These files have been written to `data/02-analyze`:

``` r
projthis::proj_dir_info(path_target())
```

    ## # A tibble: 5 × 4
    ##   path                    type         size modification_time  
    ##   <fs::path>              <fct> <fs::bytes> <dttm>             
    ## 1 cases.png               file      350.03K 2021-12-06 08:19:27
    ## 2 change.png              file      320.26K 2021-12-06 08:19:28
    ## 3 covid_recent_cases.csv  file        3.39K 2021-12-06 08:19:27
    ## 4 covid_recent_deaths.csv file        3.14K 2021-12-06 08:19:27
    ## 5 covid_week.csv          file        3.28M 2021-12-06 08:19:27
