Advanced dplyr Quiz
================
John Mount
9/29/2017

Advanced dplyr Quiz (author: John Mount)
========================================

Being able to effectively perform meaningful work *using* [`R`](https://www.r-project.org) programming involves being able to both know how various packages work and anticipate package method outcomes in basic situations. Any mismatch there (be it a knowledge gap in the programmer, an implementation gap in a package, or a difference between programmer opinion and package doctrine) can lead to confusion, bugs and incorrect results.

Below is our advanced [`dplyr`](https://CRAN.R-project.org/package=dplyr) quiz. Can you anticipate the result of each of the example operations? Can you anticipate which commands are in error and which are valid `dplyr`?

Or another phrasing: here are our notes on `dplyr` corner-cases (in my *opinion*). You may not need to know how any of these work (it is often good to avoid corner-cases), but you should at least be confident you are avoiding the malformed ones.

![](Pop_Quiz_Hot_Shot.jpg)

"Pop dplyr quiz, hot-shot! There is data in a pipe. What does each verb do?"

Start
=====

With the current version of `dplyr` in mind, please anticipate the result of each example command. Note: we don't claim all of the examples below are correct `dplyr` code. However, effective programming requires knowledge of what happens in some incorrect cases (at least knowing which throw usable errors, and which perform quiet mal-calculations).

``` r
suppressPackageStartupMessages(library("dplyr"))
```

Now for the examples/quiz.

Please take a moment to [try the quiz](https://github.com/WinVector/Examples/blob/master/dplyr/dplyrQuiz.md), and write down your answers before moving on to the [solutions](https://github.com/WinVector/Examples/blob/master/dplyr/dplyrQuiz_solutions.md). This should give you a much more open mind as to what constitutes "[surprising behavior](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)."

You can also run the quiz yourself by downloading and knitting the [source document](https://github.com/WinVector/Examples/blob/master/dplyr/dplyrQuiz.Rmd).

Please keep in mind while "you never want errors" you do sometimes want exceptions (which are unfortunately called "`Error:`" in `R`). Exceptions are an important way of stopping off-track computation and preventing later incorrect results. Exceptions can often be the desired outcome of a malformed calculation.

Not all of the questions are "trick", some of them are just convenient ways to remember different `dplyr` conventions.

Local data.frames
=================

Column selection
----------------

``` r
data.frame(x = 1) %>% 
  select(x)

# Two questions: 
#  1) Should this next one work?
#  2) Does this next one work?
data.frame(x = 1) %>% 
  select('x')

y <- 'x' # value used in later examples

# Hint: this is new for dplyr 0.7.* 
# (dplyr 0.5.0 only did this for integers).
# This is going to be a massive source
# of unexpected results, bugs, and
# incorrect analyses.
data.frame(x = 1) %>% 
  select(y)

data.frame(x = 1, y = 2) %>% 
  select(y)

rm(list='y') # clean up
```

(From [`dplyr` issue 2904](https://github.com/tidyverse/dplyr/issues/2904).)

rename
------

``` r
data.frame(x = 1) %>%
  rename(!!rlang::sym('y') := x)
```

``` r
data.frame(x = 1) %>%
  rename((!!rlang::sym('y')) := x)
```

distinct
--------

``` r
data.frame(x = c(1, 1)) %>% 
  select(one_of(character(0))) %>%
  distinct() %>%
  nrow()
```

(From [`dplyr` issue 2954](https://github.com/tidyverse/dplyr/issues/2954).)

tally
-----

``` r
data.frame(n = 1:3) %>% 
  tally
```

[`dplyr` issue 3070](https://github.com/tidyverse/dplyr/issues/3070).

``` r
data.frame(n = as.raw(1:3)) %>% 
  select(-n) %>% 
  tally
```

[`dplyr` issue 3071](https://github.com/tidyverse/dplyr/issues/3071).

rename and mutate
-----------------

``` r
data.frame(x=1, y=2) %>% rename(x=y, y=x)

data.frame(x=1, y=2) %>% mutate(x=y, y=x)
```

``` r
data.frame(q = 1:3, 
           z = as.raw(1:3)) %>% 
  mutate(constant=1.0)
```

[`dplyr` issue 3069](https://github.com/tidyverse/dplyr/issues/3069).

Column re-use and column chaining
---------------------------------

``` r
db <- DBI::dbConnect(RSQLite::SQLite(), ":memory:")
d <- copy_to(db, data.frame(a = 1))

d %>% 
    mutate(a2 = a, a3 = a2, a4 = a3)
```

[`dplyr` issue 3095](https://github.com/tidyverse/dplyr/issues/3095).

NULL (constant versus in a variable)
------------------------------------

``` r
data.frame(x=1, y=2) %>% mutate(x = NULL)

z <- NULL # value used in later examples
data.frame(x=1, y=2) %>% mutate(x = z)
```

``` r
data.frame(x=1, y=2) %>% mutate(x = !!z)

rm(list='z') # clean up
```

(From [`dplyr` issue 2945](https://github.com/tidyverse/dplyr/issues/2945).)

Column grouping
---------------

``` r
y <- 'x' # value used in later examples

data.frame(x = 1) %>% 
  group_by(.data[[y]]) %>% 
  summarize(count = n()) %>% 
  colnames()

rm(list='y') # clean up
```

``` r
world <- "homeworld"

starwars %>%
  group_by(.data[[world]]) %>%
  summarise_at(vars(height:mass), mean, na.rm = TRUE) %>%
  head()
```

``` r
homeworld <- "homeworld"

starwars %>%
  group_by(.data[[homeworld]]) %>%
  summarise_at(vars(height:mass), mean, na.rm = TRUE) %>%
  head()
```

``` r
homeworld <- "homeworld"

starwars %>%
  group_by(!!homeworld) %>%
  colnames() %>%
  setdiff(colnames(starwars))
```

Re-filed as [`dplyr` issue `3056`](https://github.com/tidyverse/dplyr/issues/3056).

``` r
grouping_column <- "homeworld"

starwars %>%
  select(.data[[grouping_column]]) %>%
  group_by(.data[[grouping_column]]) %>%
  select(.data[[grouping_column]])
```

(From [`dplyr` issue 2916](https://github.com/tidyverse/dplyr/issues/2916) and [`dplyr` issue 2991](https://github.com/tidyverse/dplyr/issues/2991), notation taken from [here](https://blog.rstudio.org/2017/06/13/dplyr-0-7-0/)). A note on how the behavior of the "pronouns" has not matched their description since the announcement of `dplyr` `0.7.0` can be found [here](http://www.win-vector.com/blog/2017/08/is-dplyr-easily-comprehensible/#comment-66674).

Piping into different targets (functions, blocks expressions):
--------------------------------------------------------------

`magrittr` pipe details:

``` r
data.frame(x = 1)  %>%  { bind_rows(list(., .)) }

data.frame(x = 1)  %>%    bind_rows(list(., .))

data.frame(x = 1)  %>%  ( bind_rows(list(., .)) )
```

enquo rules
-----------

This section is about `enquo()`, or passing unquoted variable names to functions that use `dplyr` methods. Please skip it if you don't write such code, or if you use something like [`wrapr::let()`](https://github.com/WinVector/wrapr/blob/master/README.md) for such replacements.

``` r
(function(z) select(data.frame(x = 1), !!enquo(z)))(x)

(function(z) data.frame(x = 1) %>% select(!!enquo(z)))(x)
```

(From [`dplyr` issue 2726](https://github.com/tidyverse/dplyr/issues/2726).)

``` r
y <- NULL # value used in later examples

(function(z) mutate(data.frame(x = 1), !!quo_name(enquo(z)) := 2))(y)

(function() mutate(data.frame(x = 1), !!quo_name(enquo(y)) := 2))()
```

(From [`rlang` issue 203](https://github.com/tidyverse/rlang/issues/203).)

functions
---------

``` r
f <- . %>% { sum(!is.na(.)) }
dplyr::summarise_all(data.frame(wat = letters), 
                     dplyr::funs(f))

dplyr::summarise_all(data.frame(wat = letters), 
                     dplyr::funs(. %>% { sum(!is.na(.)) }))

f <- function(col) { sum(!is.na(col)) }
dplyr::summarise_all(data.frame(wat = letters), 
                     dplyr::funs(f))

dplyr::summarise_all(data.frame(wat = letters), 
                     dplyr::funs(function(col) { sum(!is.na(col)) }))
```

(From [`dplyr` issue 3094](https://github.com/tidyverse/dplyr/issues/3094).)

Databases
=========

Setup:

``` r
# values used in later examples
db <- DBI::dbConnect(RSQLite::SQLite(), 
                     ":memory:")
dL <- data.frame(x = 3.077, 
                k = 'a', 
                stringsAsFactors = FALSE)
dR <- dplyr::copy_to(db, dL, 'dR')
```

nrow()
------

``` r
nrow(dL)

nrow(dR)
```

(From [`dplyr` issue 2871](https://github.com/tidyverse/dplyr/issues/2871).)

union\_all()
------------

``` r
union_all(dR, dR)

union_all(dL, head(dL))

union_all(dR, head(dR))
```

(From [`dplyr` issue 2858](https://github.com/tidyverse/dplyr/issues/2858).)

mutate\_all funs()
------------------

``` r
dR %>% 
  mutate_all(funs(round(., 2)))

dL %>% select(x) %>% 
  mutate_all(funs(round(., digits = 2)))

dR %>% select(x) %>% 
  mutate_all(funs(round(., digits = 2)))
```

(From [`dplyr` issue 2890](https://github.com/tidyverse/dplyr/issues/2890) and [`dplyr` issue 2908](https://github.com/tidyverse/dplyr/issues/2908).)

rename again
------------

``` r
dR %>% 
  rename(x2 = x) %>% rename(k2 = k)

dL %>% 
  rename(x2 = x, k2 = k)

dR %>% 
  rename(x2 = x, k2 = k)
```

(From [`dplyr` issue 2860](https://github.com/tidyverse/dplyr/issues/2860) and [`dplyr` issue 3129](https://github.com/tidyverse/dplyr/issues/3129).)

if\_else
--------

``` r
v <- glue::glue('a')
print(v)
if_else(TRUE, v, v)
```

[`dplyr` issue 3109](https://github.com/tidyverse/dplyr/issues/3109).

Conclusion
==========

The above quiz is really my working notes on both how things work (so many examples are correct), and corner-cases to avoid. Some of the odd cases are simple bugs (which will likely be fixed), and some are legacy behaviors from earlier versions of `dplyr`. In many cases you can and should re-arrange your `dplyr` pipelines to avoid triggering the above issues. But to do that, you have to know what to avoid (hence the notes).

My quiz-grading priniciple comes from *Software for Data Analysis: Programming with R* by John Chambers (Springer 2008):

> ... the computations and the software for data analysis should be trustworthy: they should do what the claim, and be seen to do so. Neither those how view the results of data analysis nor, in many cases, the statisticians performing the analysis can directly validate extensive computations on large and complicated data processes. Ironically, the steadily increasing computer power applied to data analysis often distances the result further from direct checking by the recipient. The many computational steps between original data source and displayed results must all be truthful, or the effect of the analysis may be worthless, if not pernicious. This places an obligation on all creators of software to program in such a way that the computations can be understood and trusted.

The point is: to know a long calculation is correct, we must at least know all the small steps did what we (the analyst) intended (and not something else). To go back to one of our examples: the analyst must know the column selected in their analysis was *always* the one they intended.

Also: please understand, some of these may *not* represent problems with the above packages. They may instead represent mistakes and misunderstandings on my part. Or opinions of mine that may differ from the considered opinions and experience of the people who have authored and who have to maintain these packages. Some things that might seem "easy to fix" to an outsider may already be set at a "best possible compromise" among many other considerations.

I may or may not keep these up to date depending on the utility of such a list going forward.

<img src="3a0.jpg" >

"Realizing column names are just strings."

\[ [quiz](https://github.com/WinVector/Examples/blob/master/dplyr/dplyrQuiz.md) \] \[ [solutions](https://github.com/WinVector/Examples/blob/master/dplyr/dplyrQuiz_solutions.md) \]
