--- 
title: "Github actions with R"
author: "Chris Brown, Murray Cadzow, Paula A Martinez, Rhydwyn McGuire, David Neuzerling, David Wilkinson, Saras Windecker"
date: "2019-12-16"
site: bookdown::bookdown_site
documentclass: book
bibliography: [packages.bib]
biblio-style: apalike
link-citations: yes
description: "An introduction to using github actions with R."
rmd_files: ["index.Rmd", "02-package_ci.Rmd", "03-understanding_yaml.Rmd", "04-testing_with_renv.Rmd", "05-contributions.Rmd"]
---



# Introduction

## What are GitHub Actions?

[GitHub actions](https://github.com/features/actions) allow us to trigger automated steps after we launch GitHub interactions such as when we push, pull, submit a pull request, or write an issue. 

For example, there are actions that will automatically trigger:

- continuous integration (CI)
- messages in response to issues or pull requests
- rendering/compiling e.g. of rmarkdown, bookdown, blogdowns etc

GitHub actions follow the steps designated in a `yaml` file, which we place in the `.github/workflows` folder of the repo. 
We can add these `yaml` files to our repo either by clicking on a series of steps on GitHub.com, or using wrapper functions provided by the `usethis` package, depending on which actions you which to include.
We describe both ways here. 

### Usethis Wrappers 

[Jim Hester](https://github.com/jimhester) is working to add GitHub action functionalities to the development version of the [`usethis` package](https://usethis.r-lib.org/reference/github_actions.html).
To use these functions now, you'll need to install the development version, as follows:


```r
# install.packages("devtools")
devtools::install_github("r-lib/usethis")
```

Two specific GitHub actions related to continuous integration can be implemented with either of the following: 


```r
usethis::use_github_action_check_release()
usethis::use_github_action_check_full()
```


More details are in chapter \@ref(packageci). 


There are a range of other R actions available in the [r-lib library](https://github.com/r-lib/actions/tree/master/examples). 
You can add these example `yaml` files using the following function (demonstrated here with the check-release action):


```r
usethis:::use_github_action('check-release.yaml')
```

### Marketplace Actions

There are a huge selection of other actions that you can choose from in the [Marketplace](https://github.com/marketplace?type=actions) that automate not only GitHub processes but also programming-language-specific options.
To implement these, go to any repo you _own_ and you will find "Actions" on the top menu. 
Click on "New Workflow" and pick one from the templates provided.

In some cases the `yaml` will need to be modified. More detail on understanding the `yaml` files can be found in chapter \@ref(understanding-yaml). 

## Extensions

We experimented with setting up continuous integration with a reproducible environment using `renv` in chapter \@ref(testing-with-renev).


## More information and useful links

- R specific GitHub actions [examples](https://github.com/r-lib/actions/tree/master/examples) from r-lib
- `use_github_action` function and similar functions help [documentation](https://usethis.r-lib.org/reference/github_actions.html?q=#arguments) 
- [Glossary of actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/core-concepts-for-github-actions)
- [Workflow syntax for GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)

:tada: 
Now you know that are GitHub actions, and where to find more information about those!

---

Note: the [r-lib/ghactions](https://github.com/r-lib/ghactions) repo is deprecated!
