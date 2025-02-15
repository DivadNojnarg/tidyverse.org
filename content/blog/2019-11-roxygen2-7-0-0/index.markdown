---
title: roxygen2 7.0.0
author: Hadley Wickham
date: '2019-11-12'
slug: roxygen2-7-0-0
categories:
  - package
description: >
  A massive update to roxygen2 now on CRAN.
tags:
  - r-lib
  - devtools
  - roxygen2
photo:
  url: https://unsplash.com/photos/8XddFc6NkBY
  author: Art Lasovsky
---

We're exceedingly happy to announce the release of [roxygen2 7.0.0](https://roxygen2.r-lib.org). roxygen2 allows you to write specially formatted R comments that generate R documentation files (`man/*.Rd`) and a `NAMESPACE` file. roxygen2 is used by over 8,000 CRAN packages.

Install the latest version of roxygen2 with:


```r
install.packages("roxygen2")
```

This a huge release containing [many minor improvements and bug fixes](https://roxygen2.r-lib.org/news/index.html#roxygen2-7-0-0). This blog post focusses on seven major improvements:

* roxygen2 is no longer (ironically!) the worst documented package. It has a 
  fresh new website, <https://roxygen2.r-lib.org>, and the vignettes have 
  all been updated.

* We've made a number of tweaks to the rendering of `.Rd`. Most importantly
  you no longer needed to escape `%` in markdown, and functions with many
  arguments are displayed with one argument per line. We've also removed a 
  couple of old features that are no longer supported.

* You can create tables and headings from markdown.

* You can share text and code between your README/vignettes and your 
  documentation with the new `@includeRmd` tag.

* You can now document R6 classes!

* A package can declare how roxygen2 should load its code, making it easier to 
  use roxygen2 in wider variety of workflows.

* roxygen2 has a documented extension mechanism so that it's easy to implement
  new tags and new roclets in other packages.

## Improved documentation

roxygen2 finally (!!) has a [pkgdown](https://pkgdown.r-lib.org/) website! 

I used this as an opportunity to look at all the vignettes and make sure they are comprehensive and readable. These are now the best place to go if you want more details on any roxygen2 tag:

* [Rd](https://roxygen2.r-lib.org/articles/rd.html)
* [Inline Rd formatting](https://roxygen2.r-lib.org/articles/rd-formatting.html)
* [`NAMESPACE`](https://roxygen2.r-lib.org/articles/namespace.html)

Of course, documentation can always be improved, so if you find something hard to follow, please [file an issue](https://github.com/r-lib/roxygen2/issues/new)!

## Changes to `.Rd` output

When you run roxygen2 7.0.0 for the first time, you'll notice a number of changes to the rendered `.Rd`. The two most important are:

* `%` (the Rd comment symbol) is now automatically escaped in markdown text. That means if you previously escaped it with `\%`, you'll need to remove the backslash and take it back to `%`. 

    If you forget to do this, you'll see confusing `R CMD check` notes like:

    * `unknown macro '\item'`
    * `unexpected section header '\description'`
    * `unexpected END_OF_INPUT`

* In the "Usage" section, you'll notice the formatting of function calls has changed. Previously, function calls were wrapped to produce the smallest number of lines, e.g.:
      
    
    ```r
    parse_package(path = ".", env = env_package(path), 
      registry = default_tags(), global_options = list())
    ```
    
    Now long function calls are wrapped so that each argument gets its own line:
    
    
    ```r
    parse_package(
      path = ".",
      env = env_package(path),
      registry = default_tags(),
      global_options = list()
    )
    ```
    
    If you prefer the old behaviour you can put the following in your
    `DESCRIPTION`:
    
    ```
    Roxygen: list(old_usage = TRUE)
    ```

You'll also notice a number of small improvements:

* Markdown code is converted to to either `\code{}` or `\verb{}`, depending on 
  whether it not is valid R code. For example, `` `foofy()` `` will become 
  `\code{foofy()}` but `` `1 +` `` will become `\verb{1 + }`. This better 
  matches the intended usage of the `\code{}` and `\verb{}` macros, and should 
  make it easier to include arbitrary "code" snippets in documentation without 
  causing Rd failures.

* `@family` automatically adds `()` when linking to functions,
  and prints each link on its own line (to improve diffs).
  
We've also removed a few features to simplify the code and/or clearly advertise that certain features are no longer supported:

* `@S3method` has been removed. It was deprecated in roxygen2 4.0.0
  released 2014-05-02, over 5 years ago.

* Using the `wrap` option will now trigger a warning, as it hasn't worked
  for quite some time. Suppress the warning by remove the option from your
  `DESCRIPTION`.

* `@docType package` will no longer add `-name` to the file name. If you relied 
  on this, either switch to the 
  [new workflow](https://roxygen2.r-lib.org/articles/rd.html#packages), or
  use `@name` to manaully override the default file name.

## Markdown improvements

There are two major improvements to roxygen2's markdown support: you can now use markdown headers and tables.

Markdown headings can be used in top-level tags like `@description`, `@details`, and `@returns`. Level 1 headings create a new top-level `\section{}` and Level 2 headings (and below) create nested `\subsection{}`s:

```R
#' @details
#' First sentence
#'
#' ## Subheading
#' Second sentence
#'  
#' # New section
#' Third sentence
```

is translated to

```
\details{
  First sentence
  
  \subsection{Subheading}{
    Second sentence
  }
}
\section{New section}{
  Third sentence
}
```

Markdown tables use the [GFM table syntax](https://github.github.com/gfm/#tables-extension-) and are converted to `\tabular{}` macros. For example,

```R
#' | foo | bar |
#' | :-- | --: |
#' | baz | bim |
```

is translated to

```
\tabular{lr}{
   foo \tab bar \cr
   baz \tab bim \cr
}
```

Lastly, using unsupported markdown features (like blockquotes, inline HTML, and horizontal rules) will now produce an informative message.

## `@includeRmd`

`@includeRmd` provides a new tool that allows you to share text and code amongst `.Rmd`  and `.Rd`. For example, if you have `vignettes/common.Rmd`, you can include it:

*   In documentation, with:

    ````
    #' @includeRmd vignettes/common.Rmd
    ````
  
*   In other vignettes with:

    ````
    ```{r child = "common.Rmd"}
    ```
    ````
    
*   In `README.Rmd`, with:

    ````
    ```{r child = "vignettes/common.Rmd"}
    ```
    ````

Learn more in 
[`vignette("rd")`](https://roxygen2.r-lib.org/articles/rd.html#including-external--rmd-md-files)

## R6 documentation

You can now document R6 classes! The basic usage is straighforward and works similarly to documenting functions. The main difference is that methods require explicit `@description` and `@detail` tags.


```r
#' R6 Class pepresenting a person
#'
#' A person has a name and a hair color.
Person <- R6::R6Class("Person",
  public = list(
    #' @field name First or full name of the person.
    name = NULL,

    #' @field hair Hair color of the person.
    hair = NULL,

    #' @description
    #' Create a new person object.
    #' @param name Name.
    #' @param hair Hair color.
    #' @return A new `Person` object.
    initialize = function(name = NA, hair = NA) {
      self$name <- name
      self$hair <- hair
      self$greet()
    },

    #' @description
    #' Change hair color.
    #' @param val New hair color.
    #' @examples
    #' P <- Person("Ann", "black")
    #' P$hair
    #' P$set_hair("red")
    #' P$hair
    set_hair = function(val) {
      self$hair <- val
    },

    #' @description
    #' Say hi.
    greet = function() {
      cat(paste0("Hello, my name is ", self$name, ".\n"))
    }
  )
)
```

Learn more in [`vignette("rd")`](https://roxygen2.r-lib.org/articles/rd.html#r6).

R6 documentation is a work in progress, so please let us know if you find anything missing or confusing. If you document a package with many R6 classes, you will get many warnings about missing documentation. If you want to suppress those warnings, you can turn off R6 documetation with the `r6` option, i.e. put `Roxygen: list(r6 = FALSE)` in your `DESCRIPTION`.

## Code loading

roxygen2 now provides three strategies for loading your code:

* `load_pkgload()`, the default, uses [pkgload](https://www.github.com/r-lib/pkgload). 
  Compared to the previous release, this now automatically recompiles your 
  package if needed.

* `load_source()` attaches required packages and `source()`s all files in `R/`. 
  This is a cruder simulation of package loading than pkgload (and e.g. is 
  unreliable if you use S4 extensively), but it does not require that the 
  package be compiled. Try it if the default strategy (used in roxygen2 6.1.0 
  and above) causes you grief.

* `load_installed()` assumes you have installed the package. This is best
  used as part of a bigger automated workflow.

You can override the default either by calling (e.g.) `roxygenise(load_code = "source")` or by setting the `load` option in your DESCRIPTION: `Roxygen: list(load = "source")`.

## Extending roxygen2

The process for extending roxygen2 with new tags and new roclets has been completely overhauled, and is now documented in [`vignette("extending")`](https://roxygen2.r-lib.org/articles/extending.html). A big thanks goes to [Mikkel Meyer Andersen](https://github.com/mikldk) for starting on the vignette and motivating me to make the extension process much more pleasant.

If you're one of the few people who have written a roxygen2 extension, sorry for breaking your code! But I genuinely believe that the improvements to the documentation, object structure, and print methods make it worth it. If you have previously made a new roclet, see the major changes in the [news](https://github.com/r-lib/roxygen2/blob/master/NEWS.md#extending-roxygen2). Since this interface is now documented, it will not change again without warning and a deprecation cycle. 

## Acknowledgements

A big thanks to all 69 contributors who helped make this giant release possible!  [&#x0040;achubaty](https://github.com/achubaty), [&#x0040;adam52](https://github.com/adam52), [&#x0040;alaagalal](https://github.com/alaagalal), [&#x0040;andrewmarx](https://github.com/andrewmarx), [&#x0040;batpigandme](https://github.com/batpigandme), [&#x0040;billdenney](https://github.com/billdenney), [&#x0040;Bisaloo](https://github.com/Bisaloo), [&#x0040;coolbutuseless](https://github.com/coolbutuseless), [&#x0040;DominiqueMakowski](https://github.com/DominiqueMakowski), [&#x0040;drjxf](https://github.com/drjxf), [&#x0040;EmilBode](https://github.com/EmilBode), [&#x0040;espinielli](https://github.com/espinielli), [&#x0040;Freguglia](https://github.com/Freguglia), [&#x0040;gaborcsardi](https://github.com/gaborcsardi), [&#x0040;gdurif](https://github.com/gdurif), [&#x0040;gustavdelius](https://github.com/gustavdelius), [&#x0040;ha0ye](https://github.com/ha0ye), [&#x0040;hadley](https://github.com/hadley), [&#x0040;halldc](https://github.com/halldc), [&#x0040;HenrikBengtsson](https://github.com/HenrikBengtsson), [&#x0040;hongyuanjia](https://github.com/hongyuanjia), [&#x0040;huangwb8](https://github.com/huangwb8), [&#x0040;iembry](https://github.com/iembry), [&#x0040;IndrajeetPatil](https://github.com/IndrajeetPatil), [&#x0040;inmybrain](https://github.com/inmybrain), [&#x0040;jackwasey](https://github.com/jackwasey), [&#x0040;JazzyPierrot](https://github.com/JazzyPierrot), [&#x0040;jennybc](https://github.com/jennybc), [&#x0040;jeroen](https://github.com/jeroen), [&#x0040;jhchou](https://github.com/jhchou), [&#x0040;jimhester](https://github.com/jimhester), [&#x0040;jonthegeek](https://github.com/jonthegeek), [&#x0040;kenahoo](https://github.com/kenahoo), [&#x0040;kevinrue](https://github.com/kevinrue), [&#x0040;kevinushey](https://github.com/kevinushey), [&#x0040;krlmlr](https://github.com/krlmlr), [&#x0040;leej3](https://github.com/leej3), [&#x0040;leepface](https://github.com/leepface), [&#x0040;lorenzwalthert](https://github.com/lorenzwalthert), [&#x0040;mamueller](https://github.com/mamueller), [&#x0040;maurolepore](https://github.com/maurolepore), [&#x0040;michaelquinn32](https://github.com/michaelquinn32), [&#x0040;mikemc](https://github.com/mikemc), [&#x0040;mikldk](https://github.com/mikldk), [&#x0040;mjskay](https://github.com/mjskay), [&#x0040;mnazarov](https://github.com/mnazarov), [&#x0040;msberends](https://github.com/msberends), [&#x0040;msenn](https://github.com/msenn), [&#x0040;nealrichardson](https://github.com/nealrichardson), [&#x0040;NikKrieger](https://github.com/NikKrieger), [&#x0040;nteetor](https://github.com/nteetor), [&#x0040;paleolimbot](https://github.com/paleolimbot), [&#x0040;pat-s](https://github.com/pat-s), [&#x0040;peterdesmet](https://github.com/peterdesmet), [&#x0040;phargarten2](https://github.com/phargarten2), [&#x0040;ramiromagno](https://github.com/ramiromagno), [&#x0040;Robinlovelace](https://github.com/Robinlovelace), [&#x0040;sfirke](https://github.com/sfirke), [&#x0040;Shians](https://github.com/Shians), [&#x0040;SoerenXD](https://github.com/SoerenXD), [&#x0040;stefanoborini](https://github.com/stefanoborini), [&#x0040;SteveBronder](https://github.com/SteveBronder), [&#x0040;tbates](https://github.com/tbates), [&#x0040;thalesmello](https://github.com/thalesmello), [&#x0040;the-knife](https://github.com/the-knife), [&#x0040;thomasp85](https://github.com/thomasp85), [&#x0040;trestletech](https://github.com/trestletech), [&#x0040;vivekbhr](https://github.com/vivekbhr), and [&#x0040;wenjie2wang](https://github.com/wenjie2wang).
