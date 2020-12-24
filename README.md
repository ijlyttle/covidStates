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

   After beginning this README file, the repoisitory is in [this state](https://github.com/ijlyttle/covidStates/tree/initialize).
