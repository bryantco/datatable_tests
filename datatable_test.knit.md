---
title: "Testing data.table"
author: "Bryant Cong"
date: today
format: html
---


# Setup 

First, following the `data.table` vignette, we load `data.table` and the `NYC-flights14` dataset.


::: {.cell}

```{.r .cell-code}
library(data.table) # 1.14.8 
```
:::

::: {.cell}

```{.r .cell-code}
input <- if (file.exists("flights14.csv")) {
   "flights14.csv"
} else {
  "https://raw.githubusercontent.com/Rdatatable/data.table/master/vignettes/flights14.csv"
}
flights <- fread(input)
```
:::


# Wrangling 

Let's view the fields in the data.


::: {.cell}

```{.r .cell-code}
head(flights)
```

::: {.cell-output .cell-output-stdout}
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
```
:::
:::


## Subset based on position:

All flights on January 3, 2014, that landed at LAX:


::: {.cell}

```{.r .cell-code}
head(flights[dest == "LAX" & month == 1 & day == 3])
```

::: {.cell-output .cell-output-stdout}
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
```
:::
:::


## Arranging 

Sorting by flights that had the highest air time, and also by carrier:


::: {.cell}

```{.r .cell-code}
head(flights[order(-air_time, carrier)])
```

::: {.cell-output .cell-output-stdout}
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
```
:::
:::


## Returning data subsets 

Returning the air time column as a vector:


::: {.cell}

```{.r .cell-code}
head(flights[, air_time])
```

::: {.cell-output .cell-output-stdout}
```
[1] 359 363 351 157 350 339
```
:::
:::


Returning the air time column as a `data.table`: 


::: {.cell}

```{.r .cell-code}
head(flights[, .(air_time)])
```

::: {.cell-output .cell-output-stdout}
```
   air_time
1:      359
2:      363
3:      351
4:      157
5:      350
6:      339
```
:::
:::


Returning columns as `data.table` and renaming:


::: {.cell}

```{.r .cell-code}
head(flights[, .(port_of_origin = origin, port_of_destination = dest)])
```

::: {.cell-output .cell-output-stdout}
```
   port_of_origin port_of_destination
1:            JFK                 LAX
2:            JFK                 LAX
3:            JFK                 LAX
4:            LGA                 PBI
5:            JFK                 LAX
6:            EWR                 LAX
```
:::
:::


Computing on `j`: how many trips were over 4,000 miles?


::: {.cell}

```{.r .cell-code}
flights[, sum(distance > 4000)]
```

::: {.cell-output .cell-output-stdout}
```
[1] 561
```
:::
:::


Subset on `i` and compute in `j`: count the number of flights from JFK that were delayed


::: {.cell}

```{.r .cell-code}
flights[origin == "JFK", .N]
```

::: {.cell-output .cell-output-stdout}
```
[1] 81483
```
:::
:::


## Column selection 

Selecting columns using a vector:


::: {.cell}

```{.r .cell-code}
cols <- c("origin", "dest", "air_time")

head(flights[, ..cols]) # note that the selection happens in j
```

::: {.cell-output .cell-output-stdout}
```
   origin dest air_time
1:    JFK  LAX      359
2:    JFK  LAX      363
3:    JFK  LAX      351
4:    LGA  PBI      157
5:    JFK  LAX      350
6:    EWR  LAX      339
```
:::
:::


Deselecting columns:


::: {.cell}

```{.r .cell-code}
head(flights[, -cols, with = FALSE])
```

::: {.cell-output .cell-output-stdout}
```
   year month day dep_delay arr_delay carrier distance hour
1: 2014     1   1        14        13      AA     2475    9
2: 2014     1   1        -3        13      AA     2475   11
3: 2014     1   1         2         9      AA     2475   19
4: 2014     1   1        -8       -26      AA     1035    7
5: 2014     1   1         2         1      AA     2475   13
6: 2014     1   1         4         0      AA     2454   18
```
:::

```{.r .cell-code}
# I slightly prefer this to 
# head(flights[, -..cols])
```
:::


# Aggregating 

## Grouping using by 

Number of flights per month?


::: {.cell}

```{.r .cell-code}
flights[, .(n = .N), by = month]
```

::: {.cell-output .cell-output-stdout}
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
```
:::
:::


Use `i`, `j`, and `k` at the same time:

Number of flights for United Airlines that were delayed in 2014, by month


::: {.cell}

```{.r .cell-code}
flights[carrier == "UA", .(num_delays = sum(dep_delay > 0)), by = month]
```

::: {.cell-output .cell-output-stdout}
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
```
:::
:::


Using `keyby` to sort: 

Number of flights delayed by carrier in January 2014 


::: {.cell}

```{.r .cell-code}
flights[month == 1, .(num_delays = sum(dep_delay > 0)), keyby = carrier]
```

::: {.cell-output .cell-output-stdout}
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
```
:::
:::



Chaining: filtering for proportion delayed over 20%



::: {.cell}

```{.r .cell-code}
flights[, .(p_delayed = sum(dep_delay > 0) / .N), by = carrier][p_delayed > 0.2]
```

::: {.cell-output .cell-output-stdout}
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
```
:::
:::

