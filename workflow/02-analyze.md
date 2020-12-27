Analyze data
================
Compiled at 2020-12-27 05:03:35 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "a4069103-4402-4559-ba03-cca3df086442")
```

    ## here() starts at /Users/runner/work/covidStates/covidStates/workflow

``` r
library("conflicted")
library("projthis")
library("here")
library("readr")
library("dplyr")
library("albersusa")
library("ggplot2")
library("glue")
```

    ## 
    ## Attaching package: 'glue'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     collapse

``` r
conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Removing existing preference

    ## [conflicted] Will prefer [34mdplyr::filter[39m over any other package

``` r
conflict_prefer("lag", "dplyr")
```

    ## [conflicted] Will prefer [34mdplyr::lag[39m over any other package

The purpose of this document is to create some state-based maps that
show the current trajectory of COVID-19 cases. There will be two maps:

  - seven-day average of newly-reported cases
  - change in newly-reported cases vs. previous seven days

<!-- end list -->

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

    ## [90m# A tibble: 15,270 x 5[39m
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
    ## [90m# … with 15,260 more rows[39m

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

    ## [90m# A tibble: 15,270 x 12[39m
    ## [90m# Groups:   state [51][39m
    ##    date       state cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    [3m[90m<date>[39m[23m     [3m[90m<chr>[39m[23m       [3m[90m<dbl>[39m[23m            [3m[90m<dbl>[39m[23m          [3m[90m<dbl>[39m[23m            [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-01-21 Wash…           1            0.013             [31mNA[39m               [31mNA[39m
    ## [90m 2[39m 2020-01-22 Wash…           1            0.013             [31mNA[39m               [31mNA[39m
    ## [90m 3[39m 2020-01-23 Wash…           1            0.013             [31mNA[39m               [31mNA[39m
    ## [90m 4[39m 2020-01-24 Illi…           1            0.008             [31mNA[39m               [31mNA[39m
    ## [90m 5[39m 2020-01-24 Wash…           1            0.013             [31mNA[39m               [31mNA[39m
    ## [90m 6[39m 2020-01-25 Cali…           1            0.003             [31mNA[39m               [31mNA[39m
    ## [90m 7[39m 2020-01-25 Illi…           1            0.008             [31mNA[39m               [31mNA[39m
    ## [90m 8[39m 2020-01-25 Wash…           1            0.013             [31mNA[39m               [31mNA[39m
    ## [90m 9[39m 2020-01-26 Ariz…           1            0.014             [31mNA[39m               [31mNA[39m
    ## [90m10[39m 2020-01-26 Cali…           2            0.005             [31mNA[39m               [31mNA[39m
    ## [90m# … with 15,260 more rows, and 6 more variables: cases_week_growth [3m[90m<dbl>[90m[23m,[39m
    ## [90m#   deaths_total [3m[90m<dbl>[90m[23m, deaths_total_per100k [3m[90m<dbl>[90m[23m, deaths_avg_week [3m[90m<dbl>[90m[23m,[39m
    ## [90m#   deaths_avg_week_per100k [3m[90m<dbl>[90m[23m, deaths_week_growth [3m[90m<dbl>[90m[23m[39m

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

    ## [90m# A tibble: 51 x 7[39m
    ## [90m# Groups:   state [51][39m
    ##    date       state cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    [3m[90m<date>[39m[23m     [3m[90m<chr>[39m[23m       [3m[90m<dbl>[39m[23m            [3m[90m<dbl>[39m[23m          [3m[90m<dbl>[39m[23m            [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-12-25 Cali…     2[4m0[24m[4m6[24m[4m4[24m511            [4m5[24m225.         [4m3[24m[4m6[24m418              92.2
    ## [90m 2[39m 2020-12-25 Tenn…      [4m5[24m[4m3[24m[4m2[24m375            [4m7[24m796.          [4m6[24m010.             88.0
    ## [90m 3[39m 2020-12-25 Ariz…      [4m4[24m[4m8[24m[4m6[24m993            [4m6[24m691.          [4m6[24m136.             84.3
    ## [90m 4[39m 2020-12-25 Alab…      [4m3[24m[4m4[24m[4m2[24m426            [4m6[24m984.          [4m3[24m820.             77.9
    ## [90m 5[39m 2020-12-25 Okla…      [4m2[24m[4m7[24m[4m2[24m553            [4m6[24m888.          [4m2[24m970.             75.1
    ## [90m 6[39m 2020-12-25 Arka…      [4m2[24m[4m1[24m[4m3[24m267            [4m7[24m067.          [4m2[24m264.             75.0
    ## [90m 7[39m 2020-12-25 Indi…      [4m4[24m[4m9[24m[4m1[24m125            [4m7[24m295.          [4m5[24m014.             74.5
    ## [90m 8[39m 2020-12-25 West…       [4m7[24m[4m8[24m836            [4m4[24m399.          [4m1[24m298.             72.4
    ## [90m 9[39m 2020-12-25 Dela…       [4m5[24m[4m3[24m653            [4m5[24m510.           649.             66.7
    ## [90m10[39m 2020-12-25 Miss…      [4m2[24m[4m0[24m[4m4[24m178            [4m6[24m860.          [4m1[24m967.             66.1
    ## [90m# … with 41 more rows, and 1 more variable: cases_week_growth [3m[90m<dbl>[90m[23m[39m

``` r
covid_recent_deaths <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("deaths")) %>%
  arrange(desc(deaths_avg_week_per100k)) %>%
  print()
```

    ## [90m# A tibble: 51 x 7[39m
    ## [90m# Groups:   state [51][39m
    ##    date       state deaths_total deaths_total_pe… deaths_avg_week
    ##    [3m[90m<date>[39m[23m     [3m[90m<chr>[39m[23m        [3m[90m<dbl>[39m[23m            [3m[90m<dbl>[39m[23m           [3m[90m<dbl>[39m[23m
    ## [90m 1[39m 2020-12-25 Sout…         [4m1[24m430            162.             14.3
    ## [90m 2[39m 2020-12-25 Arka…         [4m3[24m438            114.             42.7
    ## [90m 3[39m 2020-12-25 Penn…        [4m1[24m[4m4[24m892            116.            175. 
    ## [90m 4[39m 2020-12-25 Iowa          [4m3[24m744            119.             41.9
    ## [90m 5[39m 2020-12-25 West…         [4m1[24m247             69.6            22.3
    ## [90m 6[39m 2020-12-25 New …         [4m2[24m309            110.             25.9
    ## [90m 7[39m 2020-12-25 Ariz…         [4m8[24m409            116.             84.3
    ## [90m 8[39m 2020-12-25 Alab…         [4m4[24m680             95.4            54.9
    ## [90m 9[39m 2020-12-25 Miss…         [4m5[24m633             91.8            66.1
    ## [90m10[39m 2020-12-25 Indi…         [4m7[24m770            115.             72.1
    ## [90m# … with 41 more rows, and 2 more variables: deaths_avg_week_per100k [3m[90m<dbl>[90m[23m,[39m
    ## [90m#   deaths_week_growth [3m[90m<dbl>[90m[23m[39m

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
    legend.text.align = 1 # right-justify
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
    legend.text.align = 1 # right-justify
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
proj_dir_info(path_target())
```

    ## [90m# A tibble: 5 x 4[39m
    ##   path                    type         size modification_time  
    ##   [3m[90m<fs::path>[39m[23m              [3m[90m<fct>[39m[23m [3m[90m<fs::bytes>[39m[23m [3m[90m<dttm>[39m[23m             
    ## [90m1[39m [01;35mcases.png[0m               file      354.88K 2020-12-27 [90m05:03:40[39m
    ## [90m2[39m [01;35mchange.png[0m              file         339K 2020-12-27 [90m05:03:41[39m
    ## [90m3[39m covid_recent_cases.csv  file        3.42K 2020-12-27 [90m05:03:40[39m
    ## [90m4[39m covid_recent_deaths.csv file        3.19K 2020-12-27 [90m05:03:40[39m
    ## [90m5[39m covid_week.csv          file        1.46M 2020-12-27 [90m05:03:40[39m
