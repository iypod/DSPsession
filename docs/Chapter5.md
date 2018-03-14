Chapter5 分割表の解析
================

-   [5.1 分割表](#分割表)

5.1 分割表
----------

分割表とは、クロス集計表のことである。以上。

Rにはデータから分割表を作る便利な関数がある。
例えば、タイタニック号のデータセットを使って、いくつかクロス集計表を見てみよう。

``` r
Titanic
```

    ## , , Age = Child, Survived = No
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st     0      0
    ##   2nd     0      0
    ##   3rd    35     17
    ##   Crew    0      0
    ## 
    ## , , Age = Adult, Survived = No
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st   118      4
    ##   2nd   154     13
    ##   3rd   387     89
    ##   Crew  670      3
    ## 
    ## , , Age = Child, Survived = Yes
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st     5      1
    ##   2nd    11     13
    ##   3rd    13     14
    ##   Crew    0      0
    ## 
    ## , , Age = Adult, Survived = Yes
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st    57    140
    ##   2nd    14     80
    ##   3rd    75     76
    ##   Crew  192     20

``` r
# x <- read.table("clipboard", header=FALSE)  
# x
# chisq.test(x)
```
