# covidStates

<!-- badges: start -->
<!-- badges: end -->

The purpose of this repository is to show how to use the [projthis](https://ijlyttle.github.io/projthis/) package to manage a project-based workflow. 
The particular case will be to create a workflow that a daily update of COVID-related graphics.

## Procedure 

These are the steps I have taken to get to this point:

1. Created the project using:

   ```r
   projthis::proj_create("path/to/covidStates")
   ```
  
   At this point, a new RStudio IDE window opens with the new project.

1. Customized the [DESCRIPTION](DESCRIPTION) file, then:

   ```r
   # add license (pick one you like)
   usethis::use_mit_license()
   
   # establish git repository
   usethis::use_git()   
   ```

   The RStudio IDE is restarted, then:
   
   ```r
   # put repository on GitHub
   usethis::use_github()
   
   # create this README file
   usethis::use_readme_md()
   ```

   After beginning this README file, the repository is in [this state](https://github.com/ijlyttle/covidStates/tree/initialize).

1. Created a workflow directory. 
   This is meant to be a "data-universe" with defined points for importing and publishing data; between these points is where the action is.
   
   A workflow is composed of a sequence of RMarkdown files and a corresponding sequence of data directories.
   The default is that each RMarkdown file is rendered as a `github_document`, facilitiating easy browsing on the GitHub web portal, while still enabling "private-mode". Compared with current RMarkdown capabilities, this is a decidedsly minimalist approach. 
 
   The data is relatively small (maybe a few MB), so I will keep it as a part of the git repository, i.e. I will not git-ignore it.
   To create the directory:
  
   ```r
   proj_use_workflow("workflow", git_ignore_data = FALSE)
   ```
   
   This creates a directory called `workflow`, with a `README.Rmd`. 
   It also creates a `data` directory inside the `workflow` directory, which will not appear in the git repository until files are committed to it.
   
   At this point, the repository is in [this state](https://github.com/ijlyttle/covidStates/tree/create-workflow).
   You can also check out the [changes](https://github.com/ijlyttle/covidStates/pull/2/files) from the previous state.
   
1. To see the process of putting together the workflow directory, see its [README](workflow).
