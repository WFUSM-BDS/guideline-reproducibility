guideline-reproducibility
================

The goal of guideline-reproducibility is to introduce a method for
package management that is helpful for reproducibility when
collaborating in R.

<BR>

## Motivating Example

Collaboration in R may become frustrating if differences in installed
packages in collaborators’ system libraries cause code to fail. It is
not uncommon for code that runs without error in your R session, to
encounter an error on a collaborator’s machine due to package version
differences. This can result in time-consuming debugging that takes time
away from more important tasks.

<BR>

## renv

The `renv` package can solve this issue by creating separate a project
library for all packages that are installed and used in your project’s
code. `renv` stores metadata for each package in a lockfile that can be
shared and will allow others to install the same package versions into
their project library. Since the library is project-specific, you won’t
have to worry about `renv` messing with your global library of packages.

<BR>

### Starting a renv project

`renv` is managed directly in the R console through just a few
functions.

When choosing to add `renv` to your project for the first time, you’ll
run

``` r
renv::init()
```

This adds a [few
files](https://rstudio.github.io/renv/articles/renv.html#getting-started)
including the lockfile to your project. You’ll never need to interact or
edit these directly.

If you’ve already been working on your project for awhile, initiating
the project may take longer than if you are starting a project from
scratch because `renv` will install into your project specific library
and record in the lockfile all packages it already knows you use in your
project.

<BR>

### Using a new package

Once initiating `renv`, you’ll notice that packages you had installed
for other projects, are no longer found in this project. If you end up
needing a new package, you’ll need to install it again, this time into
your project library using either `install.packages()` or
`renv::install()` like normal.

Let’s say I want to use `dplyr` for the first time in the project.
Running `install.pacakges` installs `dplyr` and all of its dependencies
into my project library.

``` r
install.packages("dplyr")
```

    The following package(s) will be installed:
    - dplyr      [1.1.4]
    - fansi      [1.0.6]
    - generics   [0.1.3]
    - pillar     [1.9.0]
    - pkgconfig  [2.0.3]
    - tibble     [3.2.1]
    - tidyselect [1.2.1]
    - utf8       [1.2.4]
    - withr      [3.0.1]
    These packages will be installed into "./guideline-reproducibility/renv/library/R-4.2/x86_64-w64-mingw32".

    Do you want to proceed? [Y/n]: Y

    Installing packages --------------------------------------------------------
    - Installing generics ...                       OK [linked from cache]
    - Installing fansi ...                          OK [linked from cache]
    - Installing utf8 ...                           OK [linked from cache]
    - Installing pillar ...                         OK [linked from cache]
    - Installing pkgconfig ...                      OK [linked from cache]
    - Installing tibble ...                         OK [linked from cache]
    - Installing withr ...                          OK [linked from cache]
    - Installing tidyselect ...                     OK [linked from cache]
    - Installing dplyr ...                          OK [linked from cache]
    Successfully installed 9 packages in 0.18 seconds.

You may think the next step is recording the package and its
dependencies in the lockfile. You do this through taking a `snapshot` of
the project.

``` r
renv::snapshot()
```

    - The lockfile is already up to date.

However, `renv` tells us the lockfile is already up to date and does not
update the lockfile. This is because we actually aren’t using `dplyr`
anywhere in our code yet. This is nice because just having a package
installed in your project library does not mean that it will get
recorded in the lockfile which would later prompt your collaborator to
install the package even though it’s not used in the project.

If we add some code using `dplyr` and then call `renv::snapshot` again,
`renv` will ask permission to update the lockfile by adding `dplyr` and
its dependencies.

``` r
library(dplyr)

dplyr_code <- penguins %>%
  filter(species == "Gentoo")
```

``` r
renv::snapshot()
```

    The following package(s) will be updated in the lockfile:

    CRAN -----------------------------------------------------------------------
    - dplyr        [* -> 1.1.4]
    - fansi        [* -> 1.0.6]
    - tidyselect   [* -> 1.2.1]
    - utf8         [* -> 1.2.4]
    - withr        [* -> 3.0.1]

    RSPM -----------------------------------------------------------------------
    - generics     [* -> 0.1.3]
    - pillar       [* -> 1.9.0]
    - pkgconfig    [* -> 2.0.3]
    - tibble       [* -> 3.2.1]

    Do you want to proceed? [Y/n]: Y

    - Lockfile written to "./guideline-reproducibility/renv.lock".

Here is a look into what the lockfile is recording about the package
`dplyr`. You see the version number, where it was downloaded from, and
all of the dependencies it relies on. Each of those dependencies also
has a file in lockfile with this information. You can explore the whole
lockfile by looking in `renv.lock` in the project directory.

    "dplyr": {
          "Package": "dplyr",
          "Version": "1.1.4",
          "Source": "Repository",
          "Repository": "CRAN",
          "Requirements": [
            "R",
            "R6",
            "cli",
            "generics",
            "glue",
            "lifecycle",
            "magrittr",
            "methods",
            "pillar",
            "rlang",
            "tibble",
            "tidyselect",
            "utils",
            "vctrs"
          ],
          "Hash": "fedd9d00c2944ff00a0e2696ccf048ec"
        }

<BR>

If you are ever unsure if there is anything new to record with a
snapshot, you can run

``` r
renv::status()
```

    No issues found -- the project is in a consistent state.

<BR>

If we add in new code that relies on ggplot2, `renv::status` alerts us
that there are installed and used packages that are not recorded in the
lockfile.

``` r
library(ggplot2)

ggplot(penguins,
        aes(x = species, y = body_mass_g, color = species)) + 
  geom_boxplot() + 
  geom_point(position = "jitter")

renv::status()
```

    The following package(s) are in an inconsistent state:

     package      installed recorded used
     colorspace   y         n        y   
     farver       y         n        y   
     ggplot2      y         n        y   
     gtable       y         n        y   
     isoband      y         n        y   
     labeling     y         n        y   
     lattice      y         n        y   
     MASS         y         n        y   
     Matrix       y         n        y   
     mgcv         y         n        y   
     munsell      y         n        y   
     nlme         y         n        y   
     RColorBrewer y         n        y   
     scales       y         n        y   
     viridisLite  y         n        y   

The solution is to call `renv::snapshot` to update the lockfile again.

``` r
renv::snapshot()
```

    The following package(s) will be updated in the lockfile:

    CRAN -----------------------------------------------------------------------
    - colorspace     [* -> 2.1-1]
    - farver         [* -> 2.1.2]
    - gtable         [* -> 0.3.5]
    - isoband        [* -> 0.2.7]
    - labeling       [* -> 0.4.3]
    - lattice        [* -> 0.20-45]
    - MASS           [* -> 7.3-58.2]
    - Matrix         [* -> 1.5-3]
    - mgcv           [* -> 1.8-42]
    - munsell        [* -> 0.5.1]
    - nlme           [* -> 3.1-162]
    - viridisLite    [* -> 0.4.2]

    RSPM -----------------------------------------------------------------------
    - ggplot2        [* -> 3.5.1]
    - RColorBrewer   [* -> 1.1-3]
    - scales         [* -> 1.3.0]

    Do you want to proceed? [Y/n]: Y

    - Lockfile written to "./guideline-reproducibility/renv.lock".

<BR>

### Sharing with Collaborators

When your collaborator opens your shared project files on their own
computer for the first time with `renv`, `renv` will be automatically
installed, and they will be prompted to run

``` r
renv::restore()
```

The function will download and install all packages of the proper
versions recorded in the lockfile into the project-specific library.

<BR>

### Try it yourself

Clone `guidelines-reproducibility` on your own computer and run
`renv::restore()` to install the currently recorded packages into your
project library. When asked how you would like to proceed because the
project has not been activated yet, choose

    Activate the project and use the project library

Then experiment with adding new packages to `packages.R` or write R
scripts using new packages to this project. Explore how you can use
`renv::status` and `renv::snapshot` to get the lockfile back up to date.

<BR>

For more information on `renv`, see the following vignettes:

- [Introduction to
  renv](https://rstudio.github.io/renv/articles/renv.html)

- [FAQs](https://rstudio.github.io/renv/articles/faq.html)
