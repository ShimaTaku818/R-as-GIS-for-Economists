# (APPENDIX) Appendix {-}

# Loop and Parallel Computing {#par-comp}







## Before you start {-}

Here we will learn how to program repetitive operations effectively and fast. We start from the basics of a loop for those who are not familiar with the concept. We then cover parallel computation using the `future.lapply` and `parallel` package. Those who are familiar with `lapply()` can go straight to Chapter \@ref(parcomp). 

Here are the specific learning objectives of this chapter.

1. Learn how to use **for loop** and `lapply()` to complete repetitive jobs 
2. Learn how not to loop things that can be easily vectorized
3. Learn how to parallelize repetitive jobs using the `future_lapply()` function from the `future.apply` package

### Direction for replication {-}

All the data in this Chapter is generated.  

### Packages to install and load

Here is the list of packages to install (if you have not) and load to run the codes in this chapter.


```r
library(dplyr) # for data wrangling
library(data.table) # for data wrangling
```

There are other packages that will be loaded during the demonstration.

## Repetitive processes and looping

### What is looping? 

We sometimes need to run the same process over and over again often with slight changes in parameters. In such a case, it is very time-consuming and messy to write all of the steps one bye one. For example, suppose you are interested in knowing the square of 1 through 5 in with a step of 1 ($[1, 2, 3, 4, 5]$). The following code certainly works:


```r
1^2 
```

```
[1] 1
```

```r
2^2 
```

```
[1] 4
```

```r
3^2 
```

```
[1] 9
```

```r
4^2 
```

```
[1] 16
```

```r
5^2 
```

```
[1] 25
```

However, imagine you have to do this for 1000 integers. Yes, you don't want to write each one of them one by one as that would occupy 1000 lines of your code, and it would be time-consuming. Things will be even worse if you need to repeat much more complicated processes like Monte Carlo simulations. So, let's learn how to write a program to do repetitive jobs effectively using loop. 

Looping is repeatedly evaluating the same (except parameters) process over and over again. In the example above, the **same** process is the action of squaring. This does not change among the processes you run. What changes is what you square. Looping can help you write a concise code to implement these repetitive processes.

### For loop

Here is how **for loop** works in general:


```r
#--- NOT RUN ---#
for (x in a_list_of_values){
  you do what you want to do with x
}
```

As an example, let's use this looping syntax to get the same results as the manual squaring of 1 through 5:


```r
for (x in 1:5){
  print(x^2)
}
```

```
[1] 1
[1] 4
[1] 9
[1] 16
[1] 25
```

Here, a list of values is $1, 2, 3, 4, 5]$. For each value in the list, you square it (`x^2`) and then print it (`print()`). If you want to get the square of $1:1000$, the only thing you need to change is the list of values to loop over as in:


```r
#--- evaluation not reported as it's too long ---#
for (x in 1:1000){
  print(x^2)
}
```

So, the length of the code does not depend on how many repeats you do, which is an obvious improvement over manual typing of every single process one by one. Note that you do not have to use $x$ to refer to an object you are going to use. It could be any combination of letters as long as you use it when you code what you want to do inside the loop. So, this would work just fine,


```r
for (bluh_bluh_bluh in 1:5){
  print(bluh_bluh_bluh^2)
}
```

```
[1] 1
[1] 4
[1] 9
[1] 16
[1] 25
```

### For loop using the `lapply()` function

You can do for loop using the `lapply()` function as well.^[`lpply()` in only one of the family of `apply()` functions. We do not talk about other types of `apply()` functions here (e.g., `apply()`, `spply()`, `mapply()`,, `tapply()`). Personally, I found myself only rarely using them. But, if you are interested in learning those, take a look at [here](https://www.datacamp.com/community/tutorials/r-tutorial-apply-family#gs.b=aW_Io) or [here](https://www.r-bloggers.com/using-apply-sapply-lapply-in-r/).] Here is how it works:


```r
#--- NOT RUN ---#  
lapply(A,B)
```

where $A$ is the list of values you go through one by one in the order the values are stored, and $B$ is the function you would like to apply to each of the values in $A$. For example, the following code does exactly the same thing as the above for loop example.


```r
lapply(1:5, function(x){x^2})
```

```
[[1]]
[1] 1

[[2]]
[1] 4

[[3]]
[1] 9

[[4]]
[1] 16

[[5]]
[1] 25
```

Here, $A$ is $[1, 2, 3, 4, 5]$. In $B$ you have a function that takes $x$ and square it. So, the above code applies the function to each of $[1, 2, 3, 4, 5]$ one by one. In many circumstances, you can write the same looping actions in a much more concise manner using the `lapply` function than explicitly writing out the loop process as in the above for loop examples. You might have noticed that the output is a list. Yes, `lapply()` returns the outcomes in a list. That is where **l** in `lapply()` comes from.  

When the operation you would like to repeat becomes complicated (almost always the case), it is advisable that you create a function of that process first. 


```r
#--- define the function first ---#
square_it <- function(x){
  return(x^2)
}

#--- lapply using the pre-defined function ---#
lapply(1:5, square_it)
```

```
[[1]]
[1] 1

[[2]]
[1] 4

[[3]]
[1] 9

[[4]]
[1] 16

[[5]]
[1] 25
```

Finally, it is a myth that you should always use `lapply()` instead of the explicit for loop syntax because `lapply()` (or other `apply()` families) is faster. They are basically the same.^[Check this [discussion](https://stackoverflow.com/questions/7142767/why-are-loops-slow-in-r) on StackOverflow. You might want to check out [this video](https://www.youtube.com/watch?v=GyNqlOjhPCQ) at 6:10 as well.]  

### Looping over multiple variables using lapply()  

`lapply()` allows you to loop over only one variable. However, it is often the case that you want to loop over multiple variables^[the `map()` function from the `purrr` package allows you to loop over two variable.]. However, it is easy to achieve this. The trick is to create a `data.frame` of the variables where the complete list of the combinations of the variables are stored, and then loop over row of the `data.frame`. As an example, suppose we are interested in understanding the sensitivity of corn revenue to corn price and applied nitrogen amount. We consider the range of $3.0/bu to $5.0/bu for corn price and 0 lb/acre to 300/acre for applied nitrogen applied. 


```r
#--- corn price vector ---#
corn_price_vec <- seq(3, 5, by = 1)

#--- nitrogen vector ---#
nitrogen_vec <- seq(0, 300, by = 100)
```

After creating vectors of the parameters, you combine them to create a complete combination of the parameters using the `expand.grid()` function, and then convert it to a `data.frame` object^[Converting to a `data.frame` is not strictly necessary.].


```r
#--- crate a data.frame that holds parameter sets to loop over ---#
parameters_data <- expand.grid(corn_price = corn_price_vec, nitrogen = nitrogen_vec) %>% 
  #--- convert the matrix to a data.frame ---#
  data.frame()

#--- take a look ---#
parameters_data
```

```
   corn_price nitrogen
1           3        0
2           4        0
3           5        0
4           3      100
5           4      100
6           5      100
7           3      200
8           4      200
9           5      200
10          3      300
11          4      300
12          5      300
```

We now define a function that takes a row number, refer to `parameters_data` to extract the parameters stored at the row number, and then calculate corn yield and revenue based on the extracted parameters. 


```r
gen_rev_corn <- function(i) {

  #--- define corn price ---#
  corn_price <- parameters_data[i,'corn_price']

  #--- define nitrogen  ---#
  nitrogen <- parameters_data[i,'nitrogen']

  #--- calculate yield ---#
  yield <- 240 * (1 - exp(0.4 - 0.02 * nitrogen))

  #--- calculate revenue ---#
  revenue <- corn_price * yield 

  #--- combine all the information you would like to have  ---#
  data_to_return <- data.frame(
    corn_price = corn_price,
    nitrogen = nitrogen,
    revenue = revenue
  )

  return(data_to_return)
}
```

This function takes $i$ (act as a row number within the function), extract corn price and nitrogen from the $i$th row of `parameters_mat`, which are then used to calculate yield and revenue^[Yield is generated based on the Mitscherlich-Baule functional form. Yield increases at the decreasing rate as you apply more nitrogen, and yield eventually hits the plateau.]. Finally, it returns a `data.frame` of all the information you used (the parameters and the outcomes).


```r
#--- loop over all the parameter combinations ---#
rev_data <- lapply(1:nrow(parameters_data), gen_rev_corn)

#--- take a look ---#
rev_data
```

```
[[1]]
  corn_price nitrogen   revenue
1          3        0 -354.1138

[[2]]
  corn_price nitrogen   revenue
1          4        0 -472.1517

[[3]]
  corn_price nitrogen   revenue
1          5        0 -590.1896

[[4]]
  corn_price nitrogen  revenue
1          3      100 574.6345

[[5]]
  corn_price nitrogen  revenue
1          4      100 766.1793

[[6]]
  corn_price nitrogen  revenue
1          5      100 957.7242

[[7]]
  corn_price nitrogen  revenue
1          3      200 700.3269

[[8]]
  corn_price nitrogen  revenue
1          4      200 933.7692

[[9]]
  corn_price nitrogen  revenue
1          5      200 1167.212

[[10]]
  corn_price nitrogen  revenue
1          3      300 717.3375

[[11]]
  corn_price nitrogen  revenue
1          4      300 956.4501

[[12]]
  corn_price nitrogen  revenue
1          5      300 1195.563
```

Successful! Now, for us to use the outcome for other purposes like further analysis and visualization, we would need to have all the results combined into a single `data.frame` instead of a list of `data.frame`s. To do this, use either `bind_rows()` from the `dplyr` package or `rbindlist()` from the `data.table` package.


```r
#--- bind_rows ---#
bind_rows(rev_data)
```

```
   corn_price nitrogen   revenue
1           3        0 -354.1138
2           4        0 -472.1517
3           5        0 -590.1896
4           3      100  574.6345
5           4      100  766.1793
6           5      100  957.7242
7           3      200  700.3269
8           4      200  933.7692
9           5      200 1167.2115
10          3      300  717.3375
11          4      300  956.4501
12          5      300 1195.5626
```

```r
#--- rbindlist ---#
rbindlist(rev_data)
```

```
    corn_price nitrogen   revenue
 1:          3        0 -354.1138
 2:          4        0 -472.1517
 3:          5        0 -590.1896
 4:          3      100  574.6345
 5:          4      100  766.1793
 6:          5      100  957.7242
 7:          3      200  700.3269
 8:          4      200  933.7692
 9:          5      200 1167.2115
10:          3      300  717.3375
11:          4      300  956.4501
12:          5      300 1195.5626
```

### Do you really need to loop? 

Actually, we should not have used for loop or `lapply()` in any of the examples above in practice^[By the way, note that `lapply()` is no magic. It's basically a for loop and not rally any faster than for loop.] This is because they can be easily vectorized. Vectorized operations are those that take vectors as inputs and work on each element of the vectors in parallel^[This does not mean that the process is parallelized by using multiple cores.]. 

A typical example of a vectorized operation would be this:


```r
#--- define numeric vectors ---#
x <- 1:1000
y <- 1:1000

#--- element wise addition ---#
z_vec <- x + y 
```

A non-vectorized version of the same calculation is this:


```r
z_la <- lapply(1:1000, function(i) x[i] + y[i]) %>%  unlist

#--- check if identical with z_vec ---#
all.equal(z_la, z_vec)
```

```
[1] TRUE
```

Both produce the same results. However, R is written in a way that is much better at doing vectorized operations. Let's time them using the `microbenchmark()` function from the `microbenchmark` package. Here, we do not `unlist()` after `lapply()` to just focus on the multiplication part.


```r
library(microbenchmark)

microbenchmark(
  #--- vectorized ---#
  "vectorized" = { x + y }, 
  #--- not vectorized ---#
  "not vectorized" = { lapply(1:1000, function(i) x[i] + y[i])},
  times = 100, 
  unit = "ms"
)
```

```
Unit: milliseconds
           expr      min        lq       mean    median       uq      max neval
     vectorized 0.002434 0.0026495 0.00288204 0.0028125 0.003022 0.005622   100
 not vectorized 0.479522 0.5108405 0.53967215 0.5213405 0.537388 2.170841   100
```

As you can see, the vectorized version is faster. The time difference comes from R having to conduct many more internal checks and hidden operations for the non-vectorized one^[See [this](http://www.noamross.net/archives/2014-04-16-vectorization-in-r-why/) or [this](https://stackoverflow.com/questions/7142767/why-are-loops-slow-in-r) to have a better understanding of why non-vectorized operations can be slower than vectorized operations.]. Yes, we are talking about a fraction of milliseconds here. But, as the objects to operate on get larger, the difference between vectorized and non-vectorized operations can become substantial^[See [here](http://www.win-vector.com/blog/2019/01/what-does-it-mean-to-write-vectorized-code-in-r/) for a good example of such a case. R is often regarded very slow compared to other popular software. But, many of such claims come from not vectorizing operations that can be vectorized. Indeed, many of the base and old R functions are written in C. More recent functions relies on C++ via the `Rcpp` package.]. 
    
The `lapply()` examples can be easily vectorized.        

Instead of this:


```r
lapply(1:1000 square_it)
```

You can just do this:


```r
square_it(1:1000)
```

You can also easily vectorize the revenue calculation demonstrated above. First, define the function differently so that revenue calculation can take corn price and nitrogen vectors and return a revenue vector.


```r
gen_rev_corn_short <- function(corn_price, nitrogen) {

  #--- calculate yield ---#
  yield <- 240 * (1 - exp(0.4 - 0.02 * nitrogen))

  #--- calculate revenue ---#
  revenue <- corn_price * yield 

  return(revenue)
}
```

Then use the function to calculate revenue and assign it to a new variable in the `parameters_data` data.


```r
rev_data_2 <- mutate(
  parameters_data,
  revenue = gen_rev_corn_short(corn_price, nitrogen)
) 
```

Let's compare the two:


```r
microbenchmark(

  #--- vectorized ---#
  "vectorized" = { rev_data <- mutate(parameters_data, revenue = gen_rev_corn_short(corn_price, nitrogen)) },
  #--- not vectorized ---#
  "not vectorized" = { parameters_data$revenue <- lapply(1:nrow(parameters_data), gen_rev_corn) },
  times = 100, 
  unit = "ms"
)
```

```
Unit: milliseconds
           expr      min       lq      mean   median       uq      max neval
     vectorized 0.244492 0.263505 0.3390336 0.283367 0.314990 4.324208   100
 not vectorized 1.897384 2.045786 2.3041810 2.111303 2.300479 6.644897   100
```

Yes, the vectorized version is faster. So, the lesson here is that if you can vectorize, then vectorize instead of using `lapply()`. But, of course, things cannot be vectorized in many cases. 


## Parallelization of embarrassingly parallel processes {#parcomp}

Parallelization of computation involves distributing the task at hand to multiple cores so that multiple processes are done in parallel. Here, we learn how to parallelize computation in R. Our focus is on the so called **embarrassingly** parallel processes. Embarrassingly parallel processes refer to a collection of processes where each process is completely independent of any another. That is, one process does not use the outputs of any of the other processes. The example of integer squaring is embarrassingly parallel. In order to calculate $1^2$, you do not need to use the result of $2^2$ or any other squares. Embarrassingly parallel processes are very easy to parallelize because you do not have to worry about which process to complete first to make other processes happen. Fortunately, most of the processes you are interested in parallelizing fall under this category^[A good example of non-embarrassingly parallel process is dynamic optimization via backward induction. You need to know the optimal solution at $t = T$, before you find the optimal solution at $t = T-1$.].  

We will use the `future.apply` package for parallelization^[There are many other options including the `parallel`, `foreach` packages.]. Using the package, parallelizing is really a piece of cake as it is basically the same syntactically as `lapply()`. 


```r
#--- load packages ---#
library(future.apply) 
```

You can find out how many cores you have available for parallel computation on your computer using the `get_num_procs()` function from the `RhpcBLASctl` package.


```r
library(RhpcBLASctl)

#--- number of all cores ---#
get_num_procs()
```

```
[1] 16
```

Before we implement parallelized `lapply()`, we need to declare what backend process we will be using by `plan()`. Here, we use `plan(multiprocess)`^[If you are a Mac or Linux user, then the `multicore` process will be used, while the `multisession` process will be used if you are using a Windows machine. The `multicore` process is superior to the `multisession` process. See [this lecture note](https://raw.githack.com/uo-ec607/lectures/master/12-parallel/12-parallel.html) on parallel programming using R by Dr. Grant McDermott's at the University of Oregon for the distinctions between the two and many other useful concepts for parallelization. At the time of this writing, if you run R through RStudio, `multiprocess` option is always redirected to `multisession` option because of the instability in doing `multiprocess`. If you use Linux or Mac and want to take the full advantage of `future_apply`, you should not run your R programs  through RStudio at least for now.]. In the `plan()` function, we can specify the number of workers. Here I will use the total number of cores less 1^[This way, you can have one more core available to do other tasks comfortably. However, if you don't mind having your computer completely devoted to the processing task at hand, then there is no reason not to use all the cores.].  


```r
plan(multiprocess, workers = get_num_procs() - 1)
```

`future_lapply()` works exactly like `lapply()`. 


```r
sq_ls <- future_lapply(1:1000, function(x) x^2)
```

This is it. The only difference you see from the serialized processing using `lapply()` is that you changed the function name to `future_lapply()`. 

Okay, now we know how we parallelize computation. Let's check how much improvement in implementation time we got by parallelization. 


```r
microbenchmark(
  #--- parallelized ---#
  "parallelized" = { sq_ls <- future_lapply(1:1000, function(x) x^2) }, 
  #--- non-parallelized ---#
  "not parallelized" = { sq_ls <- lapply(1:1000, function(x) x^2) },
  times = 100, 
  unit = "ms"
)
```

```
Unit: milliseconds
             expr       min         lq        mean      median         uq
     parallelized 95.402429 99.5338020 106.3683670 104.1453890 107.934685
 not parallelized  0.393408  0.4256175   0.4588749   0.4398625   0.470778
        max neval
 257.110338   100
   1.591763   100
```

Hmmmm, okay, so parallelization made the code slower... How could this be? This is because communicating jobs to each core takes some time as well. So, if each of the iterative processes is super fast (like this example where you just square a number), the time spent on communicating with the cores outweighs the time saving due to parallel computation. Parallelization is more beneficial when each of the repetitive processes takes long. 

One of the very good use cases of parallelization is MC simulation. The following MC simulation tests whether the correlation between an independent variable and error term would cause bias (yes, we know the answer). The `MC_sim` function first generates a dataset (50,000 observations) according to the following data generating process:

$$
 y = 1 + x + v
$$

where $\mu \sim N(0,1)$, $x \sim N(0,1) + \mu$, and $v \sim N(0,1) + \mu$. The $\mu$ term cause correlation between $x$ (the covariate) and $v$ (the error term). It then estimates the coefficient on $x$ vis OLS, and return the estimate. We would like to repeat this process 1,000 times to understand the property of the OLS estimators under the data generating process. This Monte Carlo simulation is embarrassingly parallel because each process is independent of any other. 


```r
#--- repeat steps 1-3 B times ---#
MC_sim <- function(i){

  N <- 50000 # sample size

  #--- steps 1 and 2:  ---#
  mu <- rnorm(N) # the common term shared by both x and u
  x <- rnorm(N) + mu # independent variable
  v <- rnorm(N) + mu # error
  y <- 1 + x + v # dependent variable
  data <- data.table(y = y, x = x)

  #--- OLS ---# 
  reg <- lm(y~x, data = data) # OLS

  #--- return the coef ---#
  return(reg$coef['x'])
}
```

Let's run one iteration,


```r
tic()
MC_sim(1)
toc()
```


```
       x 
1.503353 
```

```
elapsed 
  0.023 
```

Okay, so it takes 0.023 second for one iteration. Now, let's run this 1000 times with or without parallelization.

**Not parallelized**


```r
#--- non-parallel ---#
tic()
MC_results <- lapply(1:1000, MC_sim)
toc()
```


```
elapsed 
 24.655 
```

**Parallelized**


```r
#--- parallel ---#
tic()
MC_results <- future_lapply(1:1000, MC_sim)
toc()
```


```
elapsed 
  2.785 
```

As you can see, parallelization makes it much quicker with a noticeable difference in elapsed time. We made the code 8.85 times faster. However, we did not make the process 15 times faster even though we used 15 cores for the parallelized process. This is because of the overhead associated with distributing tasks to the cores. The relative advantage of parallelization would be greater if each iteration took more time. For example, if you are running a process that takes about 2 minutes for 1000 times, it would take approximately 33 hours and 20 minutes. But, it may take only 4 hours if you parallelize it on 15 cores, or maybe even 2 hours if you run it on 30 cores. 

### Mac or Linux users

For Mac users, `parallel::mclapply()` is just as compelling (or `pbmclapply::pbmclapply()` if you want to have a nice progress report, which is very helpful particularly when the process is long). It is just as easy to use as `future_lapply()` because its syntax is the same as `lapply()`. You can control the number of cores to employ by adding `mc.cores` option. Here is an example code that does the same MC simulations we conducted above: 


```r
#--- mclapply ---#
library(parallel)
MC_results <- mclapply(1:1000, MC_sim, mc.cores = get_num_procs() - 1)

#--- or with progress bar ---#
library(pbmclapply)
MC_results <- pbmclapply(1:1000, MC_sim, mc.cores = get_num_procs() - 1)
```

### Some background on the parallelization packages 

For Mac and Linux users the coding cost of parallelization was minimal since when `parallel::mclapply()` was available. Parallelization is really a piece of cake for those who know how to use `lapply()` because the syntax is identical. For Windows users, that had not been the case until the arrival of the `future.apply` package. Windows machines create new threads to run on multiple cores, which do not inherit any of the R objects you have created on the environment. This means that if you need to use a dataset (or any other objects that you are not creating within the loop internally) inside the loop, you have to tell R explicitly what objects you want it to carry to the new threads so they can use those objects themselves. This hassle was eliminated by the `future.apply` package.^[To be honest, I do not completely understand how it does what it does.] On Mac and Linux machines, it was not even an issue because parallelization is done by forking (which Windows machines cannot do), which inherits all the available R objects on the environment. Since the `future.apply` package works for all the platforms, I focused on this package. 


