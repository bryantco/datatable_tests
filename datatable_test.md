# Testing data.table
Bryant Cong
2024-03-04

# Setup

First, following the `data.table` vignette, we load `data.table` and the
`NYC-flights14` dataset.

``` r
library(data.table) # 1.14.8 
```

``` r
input <- if (file.exists("flights14.csv")) {
   "flights14.csv"
} else {
  "https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv"
}
flights <- fread(input)
```

# Wrangling

Let’s view the fields in the data.

``` r
head(flights)
```

       year month day dep_delay arr_delay carrier origin dest air_time distance
    1: 2014     1   1        14        13      AA    JFK  LAX      359     2475
    2: 2014     1   1        -3        13      AA    JFK  LAX      363     2475
    3: 2014     1   1         2         9      AA    JFK  LAX      351     2475
    4: 2014     1   1        -8       -26      AA    LGA  PBI      157     1035
    5: 2014     1   1         2         1      AA    JFK  LAX      350     2475
    6: 2014     1   1         4         0      AA    EWR  LAX      339     2454
       hour
    1:    9
    2:   11
    3:   19
    4:    7
    5:   13
    6:   18

## Subset based on position:

All flights on January 3, 2014, that landed at LAX:

``` r
head(flights[dest == "LAX" & month == 1 & day == 3])
```

       year month day dep_delay arr_delay carrier origin dest air_time distance
    1: 2014     1   3        79        72      AA    JFK  LAX      332     2475
    2: 2014     1   3        42        26      AA    EWR  LAX      335     2454
    3: 2014     1   3        46        36      AA    JFK  LAX      323     2475
    4: 2014     1   3        93        86      B6    JFK  LAX      343     2475
    5: 2014     1   3       407       396      B6    JFK  LAX      330     2475
    6: 2014     1   3       358       350      DL    JFK  LAX      365     2475
       hour
    1:   13
    2:   19
    3:   16
    4:   22
    5:   18
    6:   21

## Arranging

Sorting by flights that had the highest air time, and also by carrier:

``` r
head(flights[order(-air_time, carrier)])
```

       year month day dep_delay arr_delay carrier origin dest air_time distance
    1: 2014     3   2        24        87      UA    EWR  HNL      706     4963
    2: 2014     3   2        -1        64      HA    JFK  HNL      704     4983
    3: 2014     3   4        -1        61      UA    EWR  HNL      697     4963
    4: 2014     3   3       -11        75      HA    JFK  HNL      690     4983
    5: 2014     3  22        -3        40      HA    JFK  HNL      689     4983
    6: 2014     1  26        13        58      UA    EWR  HNL      688     4963
       hour
    1:    9
    2:    8
    3:    9
    4:    8
    5:    9
    6:    9

## Returning data subsets

Returning the air time column as a vector:

``` r
head(flights[, air_time])
```

    [1] 359 363 351 157 350 339

Returning the air time column as a `data.table`:

``` r
head(flights[, .(air_time)])
```

       air_time
    1:      359
    2:      363
    3:      351
    4:      157
    5:      350
    6:      339

Returning columns as `data.table` and renaming:

``` r
head(flights[, .(port_of_origin = origin, port_of_destination = dest)])
```

       port_of_origin port_of_destination
    1:            JFK                 LAX
    2:            JFK                 LAX
    3:            JFK                 LAX
    4:            LGA                 PBI
    5:            JFK                 LAX
    6:            EWR                 LAX

Computing on `j`: how many trips were over 4,000 miles?

``` r
flights[, sum(distance > 4000)]
```

    [1] 561

Subset on `i` and compute in `j`: count the number of flights from JFK
that were delayed

``` r
flights[origin == "JFK", .N]
```

    [1] 81483

## Column selection

Selecting columns using a vector:

``` r
cols <- c("origin", "dest", "air_time")

head(flights[, ..cols]) # note that the selection happens in j
```

       origin dest air_time
    1:    JFK  LAX      359
    2:    JFK  LAX      363
    3:    JFK  LAX      351
    4:    LGA  PBI      157
    5:    JFK  LAX      350
    6:    EWR  LAX      339

Deselecting columns:

``` r
head(flights[, -cols, with = FALSE])
```

       year month day dep_delay arr_delay carrier distance hour
    1: 2014     1   1        14        13      AA     2475    9
    2: 2014     1   1        -3        13      AA     2475   11
    3: 2014     1   1         2         9      AA     2475   19
    4: 2014     1   1        -8       -26      AA     1035    7
    5: 2014     1   1         2         1      AA     2475   13
    6: 2014     1   1         4         0      AA     2454   18

``` r
# I slightly prefer this to 
# head(flights[, -..cols])
```

# Aggregating

## Grouping using by

Number of flights per month?

``` r
flights[, .(n = .N), by = month]
```

        month     n
     1:     1 22796
     2:     2 20813
     3:     3 26423
     4:     4 25588
     5:     5 25522
     6:     6 26488
     7:     7 27003
     8:     8 27450
     9:     9 25190
    10:    10 26043

Use `i`, `j`, and `k` at the same time:

Number of flights for United Airlines that were delayed in 2014, by
month

``` r
flights[carrier == "UA", .(num_delays = sum(dep_delay > 0)), by = month]
```

        month num_delays
     1:     1       2410
     2:     2       2280
     3:     3       2322
     4:     4       2187
     5:     5       2289
     6:     6       2767
     7:     7       2822
     8:     8       2504
     9:     9       1804
    10:    10       2477

Using `keyby` to sort:

Number of flights delayed by carrier in January 2014

``` r
flights[month == 1, .(num_delays = sum(dep_delay > 0)), keyby = carrier]
```

        carrier num_delays
     1:      AA       1056
     2:      AS         24
     3:      B6       2047
     4:      DL       1816
     5:      EV       2077
     6:      F9         19
     7:      FL        123
     8:      HA         18
     9:      MQ        741
    10:      UA       2410
    11:      US        396
    12:      VX        215
    13:      WN        583

Chaining: filtering for proportion delayed over 20%

``` r
flights[, .(p_delayed = sum(dep_delay > 0) / .N), by = carrier][p_delayed > 0.2]
```

        carrier p_delayed
     1:      AA 0.2973158
     2:      AS 0.3066202
     3:      B6 0.3706243
     4:      DL 0.3696711
     5:      EV 0.4269068
     6:      F9 0.6363636
     7:      FL 0.5443645
     8:      HA 0.2230769
     9:      MQ 0.3162886
    10:      VX 0.3516781
    11:      WN 0.5073097
    12:      UA 0.5157456
    13:      US 0.2373134
    14:      OO 0.3400000

Expressions: number of flights with air time over 300 minutes?

``` r
flights[, .N, .(air_time > 300)]
```

       air_time      N
    1:     TRUE  38722
    2:    FALSE 214594

## Practicing .SD

Compute the mean air time by carrier

``` r
flights[, lapply(.SD, mean), by = carrier, .SDcols = "air_time"]
```

        carrier  air_time
     1:      AA 194.37936
     2:      AS 325.17073
     3:      B6 150.83696
     4:      DL 175.59631
     5:      EV  90.18441
     6:      F9 226.02960
     7:      FL 101.05356
     8:      HA 625.00385
     9:      MQ  86.28725
    10:      VX 337.67375
    11:      WN 145.04436
    12:      UA 215.26611
    13:      US  86.93355
    14:      OO 112.22500

# Reference Semantics

Adding columns by reference: concatenate year, month, and day

``` r
# the empty [] after allows you to view the results
flights[, `:=` (ymd = paste0(year, "_", month, "_", day))][]
```

            year month day dep_delay arr_delay carrier origin dest air_time
         1: 2014     1   1        14        13      AA    JFK  LAX      359
         2: 2014     1   1        -3        13      AA    JFK  LAX      363
         3: 2014     1   1         2         9      AA    JFK  LAX      351
         4: 2014     1   1        -8       -26      AA    LGA  PBI      157
         5: 2014     1   1         2         1      AA    JFK  LAX      350
        ---                                                                
    253312: 2014    10  31         1       -30      UA    LGA  IAH      201
    253313: 2014    10  31        -5       -14      UA    EWR  IAH      189
    253314: 2014    10  31        -8        16      MQ    LGA  RDU       83
    253315: 2014    10  31        -4        15      MQ    LGA  DTW       75
    253316: 2014    10  31        -5         1      MQ    LGA  SDF      110
            distance hour        ymd
         1:     2475    9   2014_1_1
         2:     2475   11   2014_1_1
         3:     2475   19   2014_1_1
         4:     1035    7   2014_1_1
         5:     2475   13   2014_1_1
        ---                         
    253312:     1416   14 2014_10_31
    253313:     1400    8 2014_10_31
    253314:      431   11 2014_10_31
    253315:      502   11 2014_10_31
    253316:      659    8 2014_10_31

Deleting columns:

``` r
flights[, ymd := NULL]
```

Adding a grouped-by column (not sure `tidyverse` can do this)!

Average delay by carrier:

``` r
flights[, avg_delay := mean(dep_delay), by = .(carrier)][]
```

            year month day dep_delay arr_delay carrier origin dest air_time
         1: 2014     1   1        14        13      AA    JFK  LAX      359
         2: 2014     1   1        -3        13      AA    JFK  LAX      363
         3: 2014     1   1         2         9      AA    JFK  LAX      351
         4: 2014     1   1        -8       -26      AA    LGA  PBI      157
         5: 2014     1   1         2         1      AA    JFK  LAX      350
        ---                                                                
    253312: 2014    10  31         1       -30      UA    LGA  IAH      201
    253313: 2014    10  31        -5       -14      UA    EWR  IAH      189
    253314: 2014    10  31        -8        16      MQ    LGA  RDU       83
    253315: 2014    10  31        -4        15      MQ    LGA  DTW       75
    253316: 2014    10  31        -5         1      MQ    LGA  SDF      110
            distance hour avg_delay
         1:     2475    9  8.510532
         2:     2475   11  8.510532
         3:     2475   19  8.510532
         4:     1035    7  8.510532
         5:     2475   13  8.510532
        ---                        
    253312:     1416   14 14.296086
    253313:     1400    8 14.296086
    253314:      431   11  8.059324
    253315:      502   11  8.059324
    253316:      659    8  8.059324

Make sure to be careful in passing a `data.table` to a function, since
you might update it if you make any changes to it within a function.

# Reshaping
