
[![Build Status](https://travis-ci.com/vrunge/M2algorithmique.svg?branch=main)](https://travis-ci.com/vrunge/M2Algorithmique)

# M2algorithmique Vignette

### Vincent Runge

#### LaMME, Evry Paris-Saclay University

### November 3, 2020

> [Quick Start](#qs)

> [The 4 algorithms at fixed data length](#com)

> [Microblenchmark](#micro)

> [Time complexity](#time)

<a id="qs"></a>

## Quick Start

The `M2algorithmique` R package is an **example package** developed for students building their own R/Rcpp package as part of the **algorithmic M2 courses in Data Science master's program at Evry Paris-Saclay University**. This package contains two algorithmic strategies (**insertion sort** and **heap sort**) implemented in R and in Rcpp.

Insertion sort is of time complexity ***O*(*n*<sup>2</sup>)** as heap sort is ***O*(*n*log(*n*))** (worst case complexity). We aim at highlighting two important features with this package:

1.  Rcpp algorithms are **much more efficient** than their R counterpart
2.  Time complexities **can be compared to** one another

*All the simulations presented in this README file are available in the `myTests.R` file in the forStudents folder which also contains the Rmd file generating this README.md.*

Details on the heapsort algorithm can be found on [its wikipedia page](https://en.wikipedia.org/wiki/Heapsort). This gif provides a graphical representation of its mechanisms.

![](Heap_sort_example.gif)

The insertion sort algorithm has a simpler structure:

![](Insertion-sort-example.gif)

### Package installation

You first need to install the `devtools` package, it can be done easily from Rstudio. We install the package from Github (remove the \# sign):

``` r
#devtools::install_github("vrunge/M2algorithmique")
library(M2algorithmique)
```

### A first simple test

We simulate simple data as follows, with `v` a vector as size `n` containing all the integers from `1` to `n` (exactly one time) in any order.

``` r
n <- 10
v <- sample(n)
```

We've implemeted 4 algorithms:

-   `insertion_sort`
-   `heap_sort`
-   `insertion_sort_Rcpp`
-   `heap_sort_Rcpp`

They all have a unique argument: the unsorted vector `v`.

``` r
v
```

    ##  [1]  1  8  2  9  7 10  5  6  4  3

``` r
insertion_sort(v)
```

    ##  [1]  1  2  3  4  5  6  7  8  9 10

`insertion_sort(v)` returns the sorted vector from `v`.

<a id="com"></a>

## The 4 algorithms at fixed data length

We run all the following examples at fixed vector length `n = 10000`.

### One simulation

We define a function `one.simu` to simplify the simulation study for time complexity.

``` r
one.simu <- function(n, type = "sample", func = "insertion_sort")
{
  if(type == "sample"){v <- sample(n)}else{v <- n:1}
  if(func == "insertion_sort"){t <- system.time(insertion_sort(v))[[1]]}
  if(func == "heap_sort"){t <- system.time(heap_sort(v))[[1]]} 
  if(func == "insertion_sort_Rcpp"){t <- system.time(insertion_sort_Rcpp(v))[[1]]}
  if(func == "heap_sort_Rcpp"){t <- system.time(heap_sort_Rcpp(v))[[1]]}
  return(t)
}
```

We evaluate the time with a given `n` over the 4 algorithms. We choose

``` r
n <- 10000
```

and we get:

``` r
one.simu(n, func = "insertion_sort")
```

    ## [1] 3.133

``` r
one.simu(n, func = "heap_sort")
```

    ## [1] 0.256

``` r
one.simu(n, func = "insertion_sort_Rcpp")
```

    ## [1] 0.028

``` r
one.simu(n, func = "heap_sort_Rcpp")
```

    ## [1] 0.001

### Some comparisons

we compare the running time with repeated executions (`nbSimus` times)

``` r
nbSimus <- 10
time1 <- 0; time2 <- 0; time3 <- 0; time4 <- 0

for(i in 1:nbSimus){time1 <- time1 + one.simu(n, func = "insertion_sort")}
for(i in 1:nbSimus){time2 <- time2 + one.simu(n, func = "heap_sort")}
for(i in 1:nbSimus){time3 <- time3 + one.simu(n, func = "insertion_sort_Rcpp")}
for(i in 1:nbSimus){time4 <- time4 + one.simu(n, func = "heap_sort_Rcpp")}
```

**Rcpp is 100 to 200 times faster than R for our 2 algorithms.**

``` r
#gain R -> Rcpp
time1/time3
```

    ## [1] 116.0378

``` r
time2/time4
```

    ## [1] 191.2857

**With the data length of `10000`, heap\_sort runs 10 to 20 times faster than insert\_sort.**

``` r
#gain insertion -> heap
time1/time2
```

    ## [1] 12.60904

``` r
time3/time4
```

    ## [1] 20.78571

**The gain between the slow insertsort R algorithm and the faster heapsort Rcpp algorithm is of order 2000 !!!**

``` r
#max gain
time1/time4
```

    ## [1] 2411.929

<a id="micro"></a>

## Microblenchmark

You need the packages `microbenchmark` and `ggplot2` to run the simulations and plot the results (in violin plots). We compare `insertion_sort_Rcpp` with `heap_sort_Rcpp` for data lengths `n = 1000` and `n = 10000`.

``` r
library(microbenchmark)
library(ggplot2)
n <- 1000
res <- microbenchmark(one.simu(n, func = "insertion_sort_Rcpp"), one.simu(n, func = "heap_sort_Rcpp"), times = 50)
autoplot(res)
```

    ## Coordinate system already present. Adding new coordinate system, which will replace the existing one.

![](README_files/figure-markdown_github/unnamed-chunk-11-1.png)

``` r
res
```

    ## Unit: milliseconds
    ##                                       expr      min       lq     mean   median
    ##  one.simu(n, func = "insertion_sort_Rcpp") 43.89660 44.72201 45.18743 45.19974
    ##       one.simu(n, func = "heap_sort_Rcpp") 42.95278 44.29912 45.06825 44.90356
    ##        uq      max neval cld
    ##  45.61574 46.66871    50   a
    ##  45.56159 49.80956    50   a

``` r
n <- 10000
res <- microbenchmark(one.simu(n, func = "insertion_sort_Rcpp"), one.simu(n, func = "heap_sort_Rcpp"), times = 50)
autoplot(res)
```

    ## Coordinate system already present. Adding new coordinate system, which will replace the existing one.

![](README_files/figure-markdown_github/unnamed-chunk-12-1.png)

``` r
res
```

    ## Unit: milliseconds
    ##                                       expr      min       lq     mean   median
    ##  one.simu(n, func = "insertion_sort_Rcpp") 136.3762 138.8541 140.5156 139.8834
    ##       one.simu(n, func = "heap_sort_Rcpp") 109.1030 111.3678 112.9967 112.1167
    ##      uq      max neval cld
    ##  142.34 145.8987    50   b
    ##  114.33 136.2545    50  a

At this data length `10000` we start having a robust difference in running time.

<a id="time"></a>

## Time complexity

We run `nbRep = 50` times the `heap_sort_Rcpp` algorithm of each value of the `vector_n` vector of length `nbSimus = 20`. We show the plot of the mean running time with respect to data length.

``` r
nbSimus <- 20
vector_n <- seq(from = 10000, to = 100000, length.out = nbSimus)
nbRep <- 50
res_Heap <- data.frame(matrix(0, nbSimus, nbRep + 1))
colnames(res_Heap) <- c("n", paste0("Rep",1:nbRep))

j <- 1
for(i in vector_n)
{
  res_Heap[j,] <- c(i, replicate(nbRep, one.simu(i, func = "heap_sort_Rcpp")))  
  #print(j)
  j <- j + 1
}

res <- rowMeans(res_Heap[,-1])
plot(vector_n, res, type = 'b', xlab = "data length", ylab = "mean time in seconds")
```

![](README_files/figure-markdown_github/unnamed-chunk-13-1.png)

Same strategy but with the `insertion_sort_Rcpp` algorithm. We get the power in complexity model *O*(*n*<sup>*r*</sup>) by fitting a linear model in log scale. The slope coefficient *r* is very close to 2 as expected.

``` r
nbSimus <- 20
vector_n <- seq(from = 5000, to = 50000, length.out = nbSimus)
nbRep <- 50
res_Insertion <- data.frame(matrix(0, nbSimus, nbRep + 1))
colnames(res_Insertion) <- c("n", paste0("Rep",1:nbRep))

j <- 1
for(i in vector_n)
{
  res_Insertion[j,] <- c(i, replicate(nbRep, one.simu(i, func = "insertion_sort_Rcpp")))  
  #print(j)
  j <- j + 1
}

res <- rowMeans(res_Insertion[,-1])
plot(vector_n, res, type = 'b', xlab = "data length", ylab = "mean time in seconds")
```

![](README_files/figure-markdown_github/unnamed-chunk-14-1.png)

``` r
lm(log(res) ~ log(vector_n))
```

    ## 
    ## Call:
    ## lm(formula = log(res) ~ log(vector_n))
    ## 
    ## Coefficients:
    ##   (Intercept)  log(vector_n)  
    ##       -21.148          1.907



[Back to Top](#top)