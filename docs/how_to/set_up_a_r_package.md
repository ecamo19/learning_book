sThis note was created following this [video](https://www.youtube.com/watch?v=XjolVT16YNw)


## Functions only used at the beginning of the package development

```r
usethis::use_pipe()
```

```r
create_package()
```

```r
use_git()
```

```r
use_mit_license()
```

```r
use_testthat()
```

```r
use_github()
```

```r
use_readme_md()
```

```R
usethis::use_data()
```
## Functions used on a regular basis

```r
# Change the version of the package
usethis::use_version() 
```

```r
use_r()
```

```r
# Ignore files that rpackage don't need
use_build_ignore(c("./pixi.lock"))
```

Create a test file that will be stored testthat folder. Use the same name as the one used in the R script. The function will automatically add _test-_ to the file name 

```r
use_test()
```

```r
use_package()
usethis::use_package("dplyr", min_version = TRUE)
```


## Functions used frequently throughout a day 

```r
load_all()
```

```r
document()
```

```r
test()
devtools:::test(fresh=TRUE)
```

```r
devtools::run_examples()
```

```r
devtools::test_coverage()
```

```r
check()
```

```R
build_readme()
```