# Data Manipulation with `data.table`

* [Overview](#overview)
* [Installation](#installing-datatable)
* [Basics](#datatable-basics)
* [`i` and `j` notation](#overview)
* [Assigning Columns](#assigning-new-columns)
* [Subsetting](#subsetting-and-filtering)

## Overview

`data.table` is an `R` package built for efficient data manipulation. It is essentially an optimized version of the standard `data.frame`.

The primary benefits of `data.table` are improved and consistent syntax (once you learn how to use it) and speed/memory efficiency.

## Installing `data.table`

The code snippet below is intended to be an easy, transportable way to load packages within a script. It has the benefit of installing a package if it does not exist.

How it works: `require()` attempts to load a package. If successful, it returns `TRUE`; otherwise it will return `FALSE` and enter the body of the `if` loop, install the package, then load it.

```R
if(!require(data.table)) {
    install.packages("data.table")
    library(data.table)
}
```

## `data.table` Basics

### Creating a `data.table`

You can convert a `data.frame` to a `data.table` using the function `setDT()`, e.g.:

```R
my_data <- data.frame("col1"=1:100, "col2"=sample(100, size=10))
str(my_data)

setDT(my_data)
str(my_data)
```

Note that you don't need to assign the converted `data.table` to a variable because the original object is updated by-reference.

I tend to use `dat` or `DT` as my go-to variable name for messing around with data interactively. Of course, when writing a longer reproducible script, I'll name it something more informative.

### Creating combinations with `CJ`

You can quickly create a `data.table` with all combinations of variables using the function `CJ`.

```R
# Using ranges
dat <- CJ("month"=1:12, "year"=2001:2019)

# Using vectors
dat <- CJ("serve_in" = c("bowl", "cup", "cone"),
          "flavor" = c("chocolate", "vanilla", "strawberry"),
          "add-on" = c("sprinkles", "syrup")
         )
# Note that the multiple line format above is just for readibility.
# R ignores the white space and newlines.
```

### Combining data.table objects

You can join two `data.table` (with same column names) using an `rbind` command, e.g.:

```R
dat1 <- data.table("group" = "A", "name" = c("Apoorva", "Ilya", "Michael", "Cory"))

dat2 <- data.table("group" = "D", "name" = "Meru")

dat3 <- rbind(dat1, dat2)

# note: while simple to use, rbind() is slower than rbindlist()
# rbindlist() requires a list as input, so ideally you would use the following command:
dat4 <- rbindlist(list(dat1, dat2))

# this doesn't work if there are different column names unless fill=TRUE
dat1 <- data.table("group" = "A", "name" = c("Apoorva", "Ilya", "Michael", "Cory"))
dat2 <- data.table("group" = "D", "firstname" = "Meru")
dat5 <- rbindlist(list(dat1, dat2)) # gives an error
dat5 <- rbindlist(list(dat1, dat2), fill = TRUE) # combines, filling in  NA where data does not exist
```

Variables point to a `data.table` by reference, instead of making a copy. If you assign multiple variables to the same `data.table`, any edits you make in one will be reflected across the others.

```R
dat1 <- data.table("column1" = 1:10, "column2" = sample(100, size=10))
dat2 <- dat1

```

Selecting a subset of a `data.table` *will* make a copy.

```R
dat1 <- data.table("column1"=1:10, "column2"=sample(100, size=10))
dat2 <- dat1
dat1[, "newcolumn" := NA]
identical(dat1, dat2)
# checking dat2 shows newcolumn has been added,
# because dat1 and dat2 point to the same object


dat1 <- data.table("column1"=1:10, "column2"=sample(100, size=10))
dat2 <- copy(dat1) # explicitly call the copy() function
dat1[, "newcolumn" := NA]
identical(dat1, dat2)
# objects are not identical because copy() was called


dat1 <- data.table("column1"=1:10, "column2"=sample(100, size=10))
dat2 <- dat1[1:5] # subset the first 5 rows of dat1
dat1[, "newcolumn" := NA]
identical(dat1, dat2)
# objects are not identical, because subsetting
# forced data.table to make a copy
```

## Overview to `data.table` operations

## Assigning new columns with `:=`

## Subsetting and filtering

## Row operations in `i`

## Column operations in `j`

## Aggregate/group operations: `by`

## Chaining operations

## Advanced operations: `.SD`
