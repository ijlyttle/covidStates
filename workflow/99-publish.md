Publish
================
Compiled at 2021-02-07 08:11:19 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "ec845588-783a-4d74-9389-81c54875c3c3")
```

The purpose of this document is to pick-and-choose from the data-files
written earlier in this workflow in order to make them more-widely
available. In other circumstances, this may involve uploading data to an
external service.

Here, we will:

  - put CSV files into a directory where we will feel confident they can
    be found going forward.
  - put PNG files into a directory where the parent-project’s README
    file can expect to find them.

<!-- end list -->

``` r
library("conflicted")
```

``` r
# create or *empty* the target directory, used to write this file's data: 
projthis::proj_create_dir_target(params$name)

# function to get path to target directory: path_target("sample.csv")
path_target <- projthis::proj_path_target(params$name)

# function to get path to previous data: path_source("00-import", "sample.csv")
path_source <- projthis::proj_path_source(params$name)
```

## Transfer files

We are going to copy some files in to the target directory; we can write
a wee function to make things easier:

``` r
copy_target <- function(path) {
  fs::file_copy(path, path_target())
}
```

We use our function to copy CSV files:

``` r
copy_target(path_source("02-analyze", "covid_recent_cases.csv"))
copy_target(path_source("02-analyze", "covid_recent_deaths.csv"))
copy_target(path_source("02-analyze", "covid_week.csv"))
```

We use our function to copy PNG files:

``` r
copy_target(path_source("02-analyze", "cases.png"))
copy_target(path_source("02-analyze", "change.png"))
```

## Files written

These files have been written to `data/99-publish`:

``` r
projthis::proj_dir_info(path_target())
```

    ## # A tibble: 5 x 4
    ##   path                    type         size modification_time  
    ##   <fs::path>              <fct> <fs::bytes> <dttm>             
    ## 1 cases.png               file      353.35K 2021-02-07 08:11:19
    ## 2 change.png              file      335.65K 2021-02-07 08:11:19
    ## 3 covid_recent_cases.csv  file        3.38K 2021-02-07 08:11:19
    ## 4 covid_recent_deaths.csv file        3.18K 2021-02-07 08:11:19
    ## 5 covid_week.csv          file        1.69M 2021-02-07 08:11:19
