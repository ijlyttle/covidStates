projthis demo: COVID-19 cases in US
================
Compiled at 2021-10-24 08:17:39 UTC

``` r
here::i_am("README.Rmd", uuid = "11c1d2d6-6424-429e-9312-e14f7b7b1e05")

# function to get path to previous data: path_source("99-publish", "sample.csv")
path_source <- projthis::proj_path_source("README")
```

    ## ℹ Reading workflow configuration from '/Users/runner/work/covidStates/covidStates/workflow/_projthis.yml'

This document outlines the steps to build a
[projthis](https://ijlyttle.github.io/projthis/) workflow. This
particular workflow is used to create COVID-related graphics.

``` r
library("conflicted")
```

## Workflow

In the context of the projthis package, a *workflow* consists of a
folder with:

  - an ordered set of RMarkdown files.
  - a `data` directory with a subdirectory dedicated to each RMarkdown
    file.

This particular workflow is composed of five steps:

  - [`00-import`](00-import.md): Import data from external sources.
  - [`01-clean`](01-clean.md): Clean and wrangle data into a consistent
    format.
  - [`02-analyze`](02-analyze.md): Build some maps.
  - [`99-publish`](99-publish.md): Create “externally-available” data.
  - [`README`](README.md): Show the highlights.

There is no fixed rule for how many steps are in a workflow; however, an
“earlier” RMarkdown file shall not read from the data directory of a
“later” RMarkdown file.

I have a personal habit of including `00-import`, `99-publish`, and
`README` in all of the workflows I create. In the following sections,
I’ll walk through the process of building the each of the steps - with
a focus on what the projthis package provides.

## Procedure

This directory was created in the [main procedure](../README.md), by
calling:

``` r
proj_use_workflow("workflow", git_ignore_data = FALSE)
```

1.  This creates a `README.Rmd` from a template. Edit and knit this file
    as you see fit.

2.  With *this* `README.Rmd` file open in the RStudio IDE, create a new
    workflow-component RMarkdown file from a template using:
    
    ``` r
    projthis::proj_workflow_use_rmd("00-import")
    ```
    
    If you have an RMarkdown file open in the RStudio IDE,
    `proj_workflow_use_rmd()` will use its path as the default path for
    the newly-created file.
    
    At this stage, `00-import.Rmd` looks like
    [this](https://github.com/ijlyttle/covidStates/blob/create-import/workflow/00-import.Rmd).
    In the YAML metadata, you’ll see:
    
    ``` yaml
    title: "00-import"
    date: "Compiled at `r format(Sys.time(), '%Y-%m-%d %H:%M:%S', tz = 'UTC')` UTC"
    output: github_document
    params:
      name: "00-import" # change if you rename file
    ```
    
    You’ll almost certainly want to change the `title`.
    
    We are using the
    [parameterized-reports](https://bookdown.org/yihui/rmarkdown/parameterized-reports.html)
    feature of RMarkdown; it is important that `params$name` be
    **identical** to the sans-extension name of the RMarkdown file. If
    you change the file name, you will need to change this as well.
    
    In the first code-chunk, you’ll see:
    
    ``` r
    here::i_am(paste0(params$name, ".Rmd"), uuid = "f8c9b430-542e-4eaa-b315-bad86866aa06")
    ```
    
    This is part of why `params$name` is important - `here::i_am()`
    verifies that it is being called from a directory that:
    
      - contains a file named what you claim it is named.
      - that file contains the `uuid` in its text.
    
    This makes sure that you are running the code where (in the
    filesystem) you *expect* to be running the code. Furthermore, it
    establishes the *root directory* as the directory containing this
    file. This becomes important in a following code chunk:
    
    ``` r
    # create or *empty* the target directory, used to write this file's data: 
    projthis::proj_create_dir_target(params$name, clean = TRUE)
    
    # function to get path to target directory: path_target("sample.csv")
    path_target <- projthis::proj_path_target(params$name)
    
    # function to get path to previous data: path_source("00-import", "sample.csv")
    path_source <- projthis::proj_path_source(params$name)
    ```
    
    The call to `proj_create_dir_target()` creates a directory called
    `data/00-import`, named for this file. If this directory already
    exists, and `clean = TRUE`, it is deleted then re-created, empty.
    
    One of the principles of this workflow philosophy is that all of the
    data written from `00-import.Rmd` shall be written to
    `data/00-import`. We call this directory the **target directory** -
    no other RMarkdown file shall write to it.
    
    To enforce this, a couple of helper functions are created:
    `path_source()` and `path_target()`. These functions help you make
    sure you’re reading data from the right place, and writing data to
    the right place. For example, `path_target("sample.csv")` to return
    the path needed to write `sample.csv` into the target directory. You
    can use `path_source()` to return the path for data written to
    “earlier” data directories; it makes sure that the path you
    provide is, indeed, “earlier” than the current file.
    
    If you using these path-accessor functions to follow the rules, you
    cannot “falls off the boat”.
    
    The final section in the template prints out a file-listing of the
    target directory. This can be useful to make sure you have written
    out all the files you expect.
    
    ``` r
    proj_dir_info(path_target())
    ```
    
    This is a wrapper to `fs::dir_info()`, which:
    
      - shows the files from the perspective of the target directory.
      - returns only a small number of (hopefully useful) columns.
      - shows datetimes using UTC.

3.  Next, we build out the `00-import` file; its purpose is to import
    all of the data this workflow will use.
    
    We get the [daily COVID data for US
    states](https://github.com/nytimes/covid-19-data/blob/master/us-states.csv)
    from the New York Times. We also get an estimate of the [population
    for the US
    states](https://github.com/JoshData/historical-state-population-csv/blob/primary/historical_state_population_by_year.csv)
    from [Josh Tauberer](https://github.com/JoshData).
    
    Here’s the finished form: as
    [RMarkdown](https://github.com/ijlyttle/covidStates/blob/main/workflow/00-import.Rmd),
    and rendered as a
    [`github_document`](https://github.com/ijlyttle/covidStates/blob/main/workflow/00-import.md).
    
    Every so often, I update the project dependencies, using:
    
    ``` r
    projthis::proj_update_deps()
    ```
    
    The result is that
    [DESCRIPTION](https://github.com/ijlyttle/covidStates/blob/main/DESCRIPTION)
    file now contains:
    
        Imports: 
            rmarkdown,
            conflicted,
            here,
            projthis
    
    Until the projthis package will be on CRAN, we also have to add a
    `Remotes:` field:
    
        Remotes:
            ijlyttle/projthis

4.  Create and build `01-clean.Rmd`; the goal is to pare down the
    imported datasets to what we need. Like before:
    
    ``` r
    projthis::proj_workflow_use_rmd("01-clean")
    ```
    
    Develop some code in the file, then:
    
    ``` r
    projthis::proj_update_deps()
    ```
    
    This procedure is repeated for:
    
      - `02-analyze.Rmd`, where we put together our maps
        ([link](https://github.com/ijlyttle/covidStates/blob/workflow-analyze/workflow/02-analyze.md)
        to compiled file).
      - `99-publish.Rmd`, where the data and materials are made
        available
        ([link](https://github.com/ijlyttle/covidStates/blob/workflow-publish/workflow/99-publish.md)
        to compiled file).
    
    In my other workflows, I use a `99-publish.Rmd` file to:
    
      - export “finished product” to an external file service, like Box.
      - create [external data for
        packages](https://r-pkgs.org/data.html).
      - keep data in a “known” place inside this workflow for other
        files to import. This is how this [project’s README
        file](https://raw.githubusercontent.com/ijlyttle/covidStates/master/README.md)
        accesses the plots.

5.  This entire workflow can be rendered using:
    
    ``` r
    projthis::proj_workflow_render("workflow")
    ```
