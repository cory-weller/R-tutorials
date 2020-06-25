# Data Manipulation with `data.table`

* [overview](#overview)
* [installation](#installing-datatable)
* [basics](#datatable-basics)
* [creating a data.table](#creating-a-datatable)
* [combintions with CJ](#creating-combinations-with-cj)
* [combining data.table objects](#combining-datatable-objects)
* [data.table operations](#datatable-operations)
* [subsetting data](#subsetting-and-filtering)
* [advanced filtering](#advanced-filtering)
* [column operations](#column-operations)
* [merging data](#merging-data)
* [more](#sd-and-apply)

***

## Overview

`data.table` is an `R` package built for efficient data manipulation. It is essentially an optimized version of the standard `data.frame`.

The primary benefits of `data.table` are improved and consistent syntax (once you learn how to use it) and speed/memory efficiency.

***

## Installing `data.table`

The code snippet below is intended to be an easy, transportable way to load packages within a script. It has the benefit of installing a package if it does not exist.

How it works: `require()` attempts to load a package. If successful, it returns `TRUE`; otherwise it will return `FALSE` and enter the body of the `if` loop, install the package, then load it.

```R
if(!require(data.table)) {
    install.packages("data.table")
    library(data.table)
}
```

Tip: You can auto-load `data.table` (or any number of packages you want) every time you start up R by editing your R Profile. By default, R looks for the file `~/.Rprofile`. Any commands in this file are ran when you start R.

```sh
# append a line to .Rprofile that auto-loads data.table
echo "library(data.table)" >> ~/.Rprofile
```

***

## `data.table` Basics

### Creating a `data.table`

Creating a `data.table` is pretty much the same as creating a `data.frame`. It's still a collection of named equal-length vectors (columns that each contain one type of data). Initializing a `data.table` is simply defining the column names and contents.

*NOTE*: I tend to use `dat` or `DT` as my go-to variable name for messing around with data interactively. Of course, when writing a longer reproducible script, I'll name it something more informative.

```R
# assigning a new data.table object
dat1 <- data.table("columnName1" = c(1, 2, 4, 6, 8), "columnName2" = c("A", "B", "C", "D", "E"))

# assigning another data.table
dat2 <- data.table("randomNumber" = sample(1000), "randomDecimal" = runif(1000))

# call dat1 and dat2 (run as command) to see how they look
dat1
dat2
```

You'll notice that a `data.table` will automatically truncate output to show the first five and last five rows, which is handy for quickly checking out a `data.table` without it taking up your entire window.

An easier way to get an overview of your `data.table` is with the structure command, `str()`

```R
# check the structure of your data.tables
str(dat1)
str(dat2)
```

If you already have a `data.frame` object, you can convert it to a `data.table` using the function `setDT()`.

```R
my_data <- data.frame("col1"=1:100, "col2"=sample(100, size=10))
# check structure/format of my_data
str(my_data)

setDT(my_data)
# re-check structure/format of my_data
str(my_data)
```

Two important notes to make here:

1. You don't need to assign the converted `data.table` to a variable because the original object is updated by-reference.

2. A `data.table` does not have row names. This is intentional, and for consistency. Some `data.frame` have row names, and some do not. A row name is ultimately unnecessary, as row names is just an oddly-implemented column. `data.table` was designed with consistency in mind, so row names do not exist. You can add the option `keep.rownames = TRUE` when using `setDT()` to include row names as a new column when converting to `data.table`:

```R
dat <- copy(mtcars)
# view mtcars object
dat

# convert dat object from data.frame to data.table
setDT(dat)

# view your data.frame object
dat

# note that the column names were lost! Try again
dat <- copy(mtcars)
setDT(dat, keep.rownames = TRUE)

# view your data.frame object
dat

# see that row names are kept, as row named 'rn'
```

### Creating combinations with `CJ`

You can quickly create a `data.table` with all combinations of variables using the function `CJ`. This is useful when you want to expand two or more sets, generating every permutation of items within those sets.

```R
# using ranges
dat <- CJ("month"=1:12, "year"=2001:2019)
dat

# using vectors
dat <- CJ("serve_in" = c("bowl", "cup", "cone"),
          "flavor" = c("chocolate", "vanilla", "strawberry"),
          "add-on" = c("sprinkles", "syrup")
         )
dat
# Note that the multiple line format above is just for readibility.

# using variables
dat <- CJ("uppercase" = LETTERS, "lowercase" = letters)
dat

# R ignores the white space and newlines.
```

### Combining data.table objects

You can join two `data.table` (with same column names) using an `rbind` command. Think of this as stacking multiple tables on top of each other, increasing the total number of rows, but keeping columns the same:

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

***

## `data.table` operations

`data.table` operations are placed in square brackets `[]` and come after the `data.table`. Inside these brackets is three 'spots' for commands. In order, these are called `i`, `j` and `by`.

I like to think of it like this:

1. subset rows according to `i`

2. perform calculations according to `j`

3. separated into groups according to `by`

Like as follows: `dat[i, j, by]`. These respectively correspond to rows, columns, and groups.

Trailing commas can be omitted. That is, if you only need to perform row operations in `i`, you don't need to include any commas.

If you only want to perform column operations in `j`, then you need to include the leading comma (whether or not you are doing any row operations in `i`, e.g. `dat[, sum(N)]`.

Essentially, commands in `i` are for subsetting rows, while commands in `j` are for assigning or performing operations on columns. When `by` is included, all operations are aggregated within `by` groups (explanation coming later).

***

## Subsetting and filtering

Filtering your data (subsetting rows) in `i` is done by providing either a vector of row numbers to include, or a vector of boolean (`TRUE`, `FALSE`) values.

```R
dat <- data.table("N" = 1:1000, "col1" = runif(1000))
dat
```

You can look at a subset of your data or run filters using equality testing statements, such as `==`, `>=` and `!=`. These are placed in `i` because the test evaluates for which rows your test returns `TRUE`. Think of it as multiple steps:

1. First your test is evaluated on every row, returning either `TRUE` OR `FALSE`

2. All of your tests return a single vector (with length equal to the number of rows in your `data.table`) containing these boolean values

3. The boolean vector is plugged into `i` for your `data.table` and only those which are `TRUE` are returned.

```R
set.seed(40)
dat <- data.table("col1" = 1:1000, "col2" = sample(1000))

# for which rows is col2 > col1
# easy way to do it
dat[col2 > col1]

# harder way to do it, how it 'works' behind the scenes
boolean_values <- dat$col2 > dat$col1

# check what boolean_values looks like
dat[boolean_values]
```

You can also include numbers in `i` to subset only the row numbers you want.

```R
# initialize data.table
dat <- data.table("N" = 1:1000, "value" = runif(1000))

# only print odd rows 300 through 500:
dat[300:500]
```

***

## Advanced filtering

Two advanced ways of filtering includes the operators `%like%` and `%in%`. These can be used similar to `==` and `>=` operators.

`%like%` performs a substring search, returning `TRUE` if a match (or partial match) is found.

```R
dat <- data.table(
                "N" = 1:1000,
                "fruit" = sample(c("apricot", "apple", "orange", "banana"), size = 1000, replace = TRUE)
                )

dat[fruit %like% "ap"]
dat[fruit %like% "a"]

# regular expressions are supported
dat[fruit %like% "^o"]
```

`%in%` returns `TRUE` for rows that contain a value within a larger specified set:

```R
dat <- data.table(
                "N" = 1:1000,
                "fruit" = sample(c("apricot", "apple", "orange", "banana"), size = 1000, replace = TRUE)
                )

dat[fruit %in% c("apple", "banana")]

# note this is essentially a short-hand way of checking many == operations separated by OR
dat[fruit == "apple" | fruit == "banana"]
```

## Column operations

`j` is where the 'meat' of your operations will go. This is where you define calculations to run, such as calculating means or quantiles, adding values together, finding a maximum, etc. It can be combined with `by` to do calculations for each combination of groups in `by`.

You can specify a set of column names to simply subset your data by column

```R
# build example data set
dat <- copy(mtcars)
setDT(dat, keep.rownames = TRUE)

# subset by column by specifying list of column names
dat[, list(rn, mpg, disp, carb)]
dat[, list(rn)]

# shorthand to return a column as a VECTOR
dat[, rn]

# is the same as
dat$rn
```

Examples of performing calculations in `j`:

```R
dat <- data.table("N" = 1:30,
            "color" = sample(c("red", "blue", "yellow"), size = 30, replace = TRUE),
            "size" = sample(c("S", "M", "L", "XL"), size = 30, replace = TRUE))

# what is the highest value of N for each color?
dat[, list("highest_N" = max(N)), by=list(color)]

# what is the highest value of N for each size?
dat[, list("highest_N" = max(N)), by=list(size)]

# what is the highest value of N for each combination of color and size?
dat[, list("highest_N" = max(N)), by=list(color, size)]
```

You're not limited to just one operation in `j`. You can include as many as you want:

```R
# doing a whole bunch of stuff in i, j, and by
dat[N %% 2 == 0, list(
    "highest_N" = max(N),
    "lowest_N" = min(N),
    "median_N" = median(N),
    "sum_of_N" = sum(N)
    ), by=list(color, size)]
```

Operations in `j` is also how you assign new columns with `:=`

```R
dat <- data.table("N" = 1:1000)
dat[, "column_2" := "new_value"]

# when referring to columns as variables within j,
# you do not put quotes around them
dat[, "column_3" := N**2]
dat[, "column_4" := N - column_3]
```

What if we also include subsetting in `i` when assigning a new column?

```R
dat <- data.table("N" = 1:1000)
# what do you think will happen?
dat[N %% 5 == 0, "column 2" := "multiple of five"]
```

Putting it together, you can assign values to a new column, conditional on other column values:

```R
# Initialize a data.table with 500 individuals
dat <- data.table("indiv_id" = 1:500)

# let's simulate some genotypes
dat[, genotype := "AA"]

# Let's simulate more realistic genotypes.
# First, delete the genotypes column
dat[, genotype := NULL]

# Draw alleles
dat[, allele1 := sample(c("A", "a"), replace=TRUE, size=500)]
dat[, allele2 := sample(c("A", "a"), replace=TRUE, size=500)]

# assign genotypes conditionally
dat[allele1 == "A" & allele2 == "A", genotype := "HomMajor"]
dat[allele1 == "A" & allele2 == "a", genotype := "Het"]
dat[allele1 == "a" & allele2 == "A", genotype := "Het"]
dat[allele1 == "a" & allele2 == "a", genotype := "HomMinor"]
```

### Merging data

You can quickly merge two `data.table` together using one or more columns as keys. Think of keys as group identifiers, or explanatory variables. Measurements or response variables are unlikely to be keys. With genome data, it's common to use chromosome and base pair position as keys, because these pieces of information (together) uniquely identify a genome position.

```R
# Make a fake VCF-style file
dat <- CJ("CHR" = 1:5, "POS" = 1:10000)

# assign random reference and alternate alleles
dat[, REF := sample(c("C","G","T","A"), size=nrow(dat), replace = TRUE)]
dat[, ALT := sample(c("C","G","T","A"), size=nrow(dat), replace = TRUE)]

# remove rows where ref == alt
dat <- dat[REF != ALT]

# subset random 2% of rows
dat_subset<- dat[sample(.N, size=nrow(dat)*0.02)]

# at this point, rows are all out of order.
# set key on CHR
setkey(dat_subset, CHR)

# CHR is now ordered appropriately... but POS isn't
setkey(dat_subset, POS)

# Now only POS is in order. IMPORTANT NOTE: all keys must be
# defined at the same time. using setkey() removes previous keys.

# setkey on both CHR and POS, in that order
setkey(dat_subset, CHR, POS)

# check out your data.table
dat_subset

# say you have a separate file with sites of interest, with p-values
sites_of_interest <- dat[sample(.N, size=nrow(dat)*0.5)]
sites_of_interest[, c("REF","ALT") := NULL]
sites_of_interest[, P := runif(.N)]
sites_of_interest <- sites_of_interest[P < 0.05]
setkey(sites_of_interest, CHR, POS)

dat.merged <- merge(dat_subset, sites_of_interest)
```

***

### `.SD` and `apply`

Advanced topics. Release TBD. there's already too much
