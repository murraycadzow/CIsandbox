
# Testing with a reproducible environment {#testing-with-renev}




We can take testing a step further by setting up an environment that mimics the one we used to train out model, and is set up the same way each time. First we need to set up a *lockfile* for our package that tells R exactly what packages --- including the versions --- should be used to reproduce this 

We'll be using the `renv` package, but a word of warning first: this package is still under active development, so this information may quickly become outdated.

Once you've installed `renv`, open the R project containing your package and run `renv::snapshot()`. There will be a prompt for your consent to alter some files in your project and system. This function will automatically determine the dependencies used in your package, as well as what packages (and versions) are installed on your system, and will record this information in a lock file.

When you open your project in the future, RStudio will automatically load the packages that are recorded in the snapshot, down to the version number. If the specific versions aren't available, it will download them and install them from source, or using available binaries. Try restarting your R environment to see this happen. We can trigger this behaviour manually with `renv::restore()`. To learn more about the `renv` package, [read the introductory article](https://rstudio.github.io/renv/articles/renv.html).

Now we need to configure GitHub actions to use the snapshot we've just set up. In the previous action, we used macOS so that we could take advantage of available package binaries for quick installation. I'm going to be using Ubuntu for this to replicate my person Linux environment, and this means that packages will be installed from source.

Here's the workflow that I'm using. Read through it and see if you can work out what's going on, and then we'll go through the finer details:

```
on: [push, pull_request]

name: R-CMD-check

jobs:
  R-CMD-check:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1
      - uses: r-lib/actions/setup-r@master
        with:
            r-version: '3.6.1'
      - name: Install libcurl
        run: sudo apt-get install libcurl4-openssl-dev
      - name: Install renv 0.9.2
        run: Rscript -e "install.packages('https://cran.r-project.org/src/contrib/renv_0.9.2.tar.gz', repos = NULL, type = 'source')"
      - name: Install rcmdcheck 1.3.3
        run: Rscript -e "install.packages('https://cran.r-project.org/src/contrib/rcmdcheck_1.3.3.tar.gz', repos = NULL, type = 'source')"
      - name: Check
        run: Rscript -e "renv::restore();rcmdcheck::rcmdcheck(args = '--no-manual', error_on = 'error')"
```

There are a few differences with this workflow:

* I've specified a version of R to install, for better reproducibility.
* I've installing a package for the operating system, `libcurl4-openssl-dev`. Because we're installing R packages from source, we need to be sure that our operating system has all of the tools it needs to compile the source files. This particular package is required to use `curl` within R; this is pretty important, since if you're doing anything involving the internet, there's a good chance you're using `curl` *somewhere*. Your specific package may not need this, but I'm including it here as an example of how to run `apt-get` in GitHub Actions.
* We're installing two packages individually here: `renv` and `rcmdcheck`. We need `renv` to `restore` our lockfile, so we need to specifically install it before we think about package dependencies. Finally, `rcmdcheck` is not (usually) a package dependency, and we need it to check our package. Note that we're manually specifying version numbers here for reproducibility.
* Our final step looks a little different. We're calling `renv::restore()`, which automatically handles all of our package dependencies. Then we run `rcmdcheck` as we did before.

On Ubuntu, packages are installed from source. This is **much** slower; on one of my packages, testing in a reproducible environment on Ubuntu takes 35 minutes, as opposed to 4 minutes. Caching can make life easier, so let's take a look at that now.

The idea behind caching is to record our installed R packages so that, for future runs, we can have them readily available instead of installing from source. In one of my packages, caching decreased testing time from 35 minutes to 4 minutes. GitHub caches last a week, so if you push commits infrequently you may not realise the benefits of caching.

We implement caching by adding two steps to the above YAML:

* We install and use the `remotes` package to determine the packages that our repository depends upon.
* We cache our R libraries folder, given by the `R_LIBS_USER` environment variable, using the hash of the dependencies as the cache key. If the package dependencies change, so too does the hash, and our cache becomes invalid.

There's a third step that GitHub handles automatically --- before the job is complete, the cache is created and our installed packages are stored. On future runs, the cache is unpacked. R will see that the packages are installed and won't install them again from source.

You can read more about caching in GitHub actions in the [official documentation](https://github.com/actions/cache). An example yaml file is given below:

```
on: [push, pull_request]

name: R-CMD-check

jobs:
  R-CMD-check:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1
      - uses: r-lib/actions/setup-r@master
        with:
            r-version: '3.6.1'
      - name: Install libcurl
        run: sudo apt-get install libcurl4-openssl-dev
      - name: Query dependencies
        run: Rscript -e "install.packages('remotes')" -e "saveRDS(remotes::dev_package_deps(dependencies = TRUE), 'depends.Rds')"
      - name: Cache R packages
        if: runner.os != 'Windows'
        uses: actions/cache@v1
        with:
          path: ${{ env.R_LIBS_USER }}
          key: ${{ runner.os }}-r-3.6.1-${{ hashFiles('depends.Rds') }}
          restore-keys: ${{ runner.os }}-r-3.6.1-
      - name: Install renv 0.9.2
        run: Rscript -e "install.packages('https://cran.r-project.org/src/contrib/renv_0.9.2.tar.gz', repos = NULL, type = 'source')"
      - name: Install rcmdcheck 1.3.3
        run: Rscript -e "install.packages('https://cran.r-project.org/src/contrib/rcmdcheck_1.3.3.tar.gz', repos = NULL, type = 'source')"
      - name: Check
        run: Rscript -e "renv::restore();rcmdcheck::rcmdcheck(args = '--no-manual', error_on = 'error')"
```

