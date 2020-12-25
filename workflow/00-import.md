00-import
================
Compiled at 2020-12-25 23:02:20 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "f8c9b430-542e-4eaa-b315-bad86866aa06")
```

    ## here() starts at /Users/sesa19001/Documents/repos/public/covidStates/workflow

``` r
library("conflicted")
library("projthis")
library("here")
```

The purpose of this document is …

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

## Files written

These files have been written to `data/00-import`:

``` r
proj_dir_info(path_target())
```

    ## # A tibble: 0 x 4
    ## # … with 4 variables: path <fs::path>, type <fct>, size <fs::bytes>,
    ## #   modification_time <dttm>
