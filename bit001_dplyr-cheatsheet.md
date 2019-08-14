# Cheatsheet for dplyr join functions
Jenny Bryan  



#### Why the cheatsheet

Examples for those of us who don't speak SQL so good. There are lots of [Venn diagrams re: SQL joins on the interwebs](//encrypted.google.com/search?q=sql+join&tbm=isch), but I wanted R examples.

[Full documentation](http://www.rdocumentation.org/packages/dplyr) for the dplyr package, which is developed by Hadley Wickham and Romain Francois on [GitHub](https://github.com/hadley/dplyr). The [vignette on Two-table verbs](https://cran.r-project.org/web/packages/dplyr/vignettes/two-table.html) covers the joins shown here.

Working with two small data.frames, `superheroes` and `publishers`.


```r
suppressPackageStartupMessages(library(dplyr))
library(readr)

superheroes <- "
    name, alignment, gender,         publisher
 Magneto,       bad,   male,            Marvel
   Storm,      good, female,            Marvel
Mystique,       bad, female,            Marvel
  Batman,      good,   male,                DC
   Joker,       bad,   male,                DC
Catwoman,       bad, female,                DC
 Hellboy,      good,   male, Dark Horse Comics
"
superheroes <- read_csv(superheroes, trim_ws = TRUE, skip = 1)

publishers <- "
  publisher, yr_founded
         DC,       1934
     Marvel,       1939
      Image,       1992
"
publishers <- read_csv(publishers, trim_ws = TRUE, skip = 1)
```

Sorry, cheat sheet does not illustrate "multiple match" situations terribly well.

Sub-plot: watch the row and variable order of the join results for a healthy reminder of why it's dangerous to rely on any of that in an analysis.

#### inner_join(superheroes, publishers)

> inner_join(x, y): Return all rows from x where there are matching values in y, and all columns from x and y. If there are multiple matches between x and y, all combination of the matches are returned. This is a mutating join.


```r
(ijsp <- inner_join(superheroes, publishers))
#> Joining, by = "publisher"
#> # A tibble: 6 × 5
#>       name alignment gender publisher yr_founded
#>      <chr>     <chr>  <chr>     <chr>      <int>
#> 1  Magneto       bad   male    Marvel       1939
#> 2    Storm      good female    Marvel       1939
#> 3 Mystique       bad female    Marvel       1939
#> 4   Batman      good   male        DC       1934
#> 5    Joker       bad   male        DC       1934
#> 6 Catwoman       bad female        DC       1934
```

We lose Hellboy in the join because, although he appears in `x = superheroes`, his publisher Dark Horse Comics does not appear in `y = publishers`. The join result has all variables from `x = superheroes` plus `yr_founded`, from `y`.




<table border = 1>
<tr>
<td valign="top">
  superheroes
  

name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
<td valign="top">
  publishers
  

publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
<td valign="top">
  inner_join(x = superheroes, y = publishers)
  

name       alignment   gender   publisher    yr_founded
---------  ----------  -------  ----------  -----------
Magneto    bad         male     Marvel             1939
Storm      good        female   Marvel             1939
Mystique   bad         female   Marvel             1939
Batman     good        male     DC                 1934
Joker      bad         male     DC                 1934
Catwoman   bad         female   DC                 1934


</td>
</tr>
</table>
  
#### semi_join(superheroes, publishers)

> semi_join(x, y): Return all rows from x where there are matching values in y, keeping just columns from x. A semi join differs from an inner join because an inner join will return one row of x for each matching row of y, where a semi join will never duplicate rows of x. This is a filtering join.


```r
(sjsp <- semi_join(superheroes, publishers))
#> Joining, by = "publisher"
#> # A tibble: 6 × 4
#>       name alignment gender publisher
#>      <chr>     <chr>  <chr>     <chr>
#> 1   Batman      good   male        DC
#> 2    Joker       bad   male        DC
#> 3 Catwoman       bad female        DC
#> 4  Magneto       bad   male    Marvel
#> 5    Storm      good female    Marvel
#> 6 Mystique       bad female    Marvel
```

We get a similar result as with `inner_join()` but the join result contains only the variables originally found in `x = superheroes`. But note the row order has changed.



<table border = 1>
  <tr>
  <td valign="top">
  superheroes
  

name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
  <td valign="top">
  publishers
  

publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
  <td valign="top">
  semi-join(x = superheroes, y = publishers)
  

name       alignment   gender   publisher 
---------  ----------  -------  ----------
Batman     good        male     DC        
Joker      bad         male     DC        
Catwoman   bad         female   DC        
Magneto    bad         male     Marvel    
Storm      good        female   Marvel    
Mystique   bad         female   Marvel    


</td>
</tr>
</table>

#### left_join(superheroes, publishers)

> left_join(x, y): Return all rows from x, and all columns from x and y. If there are multiple matches between x and y, all combination of the matches are returned. This is a mutating join.


```r
(ljsp <- left_join(superheroes, publishers))
#> Joining, by = "publisher"
#> # A tibble: 7 × 5
#>       name alignment gender         publisher yr_founded
#>      <chr>     <chr>  <chr>             <chr>      <int>
#> 1  Magneto       bad   male            Marvel       1939
#> 2    Storm      good female            Marvel       1939
#> 3 Mystique       bad female            Marvel       1939
#> 4   Batman      good   male                DC       1934
#> 5    Joker       bad   male                DC       1934
#> 6 Catwoman       bad female                DC       1934
#> 7  Hellboy      good   male Dark Horse Comics         NA
```

We basically get `x = superheroes` back, but with the addition of variable `yr_founded`, which is unique to `y = publishers`. Hellboy, whose publisher does not appear in `y = publishers`, has an `NA` for `yr_founded`.



<table border = 1>
  <tr>
  <td valign="top">
  superheroes
  

name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
  <td valign="top">
  publishers
  

publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
  <td valign="top">
  left_join(x = superheroes, y = publishers)
  

name       alignment   gender   publisher            yr_founded
---------  ----------  -------  ------------------  -----------
Magneto    bad         male     Marvel                     1939
Storm      good        female   Marvel                     1939
Mystique   bad         female   Marvel                     1939
Batman     good        male     DC                         1934
Joker      bad         male     DC                         1934
Catwoman   bad         female   DC                         1934
Hellboy    good        male     Dark Horse Comics            NA


</td>
</tr>
</table>

#### anti_join(superheroes, publishers)

> anti_join(x, y): Return all rows from x where there are not matching values in y, keeping just columns from x. This is a filtering join.


```r
(ajsp <- anti_join(superheroes, publishers))
#> Joining, by = "publisher"
#> # A tibble: 1 × 4
#>      name alignment gender         publisher
#>     <chr>     <chr>  <chr>             <chr>
#> 1 Hellboy      good   male Dark Horse Comics
```

We keep __only__ Hellboy now (and do not get `yr_founded`).



<table border = 1>
  <tr>
  <td valign="top">
  superheroes
  

name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
  <td valign="top">
  publishers
  

publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
  <td valign="top">
  anti_join(x = superheroes, y = publishers)
  

name      alignment   gender   publisher         
--------  ----------  -------  ------------------
Hellboy   good        male     Dark Horse Comics 


</td>
</tr>
</table>

#### inner_join(publishers, superheroes)

> inner_join(x, y): Return all rows from x where there are matching values in y, and all columns from x and y. If there are multiple matches between x and y, all combination of the matches are returned. This is a mutating join.


```r
(ijps <- inner_join(publishers, superheroes))
#> Joining, by = "publisher"
#> # A tibble: 6 × 5
#>   publisher yr_founded     name alignment gender
#>       <chr>      <int>    <chr>     <chr>  <chr>
#> 1        DC       1934   Batman      good   male
#> 2        DC       1934    Joker       bad   male
#> 3        DC       1934 Catwoman       bad female
#> 4    Marvel       1939  Magneto       bad   male
#> 5    Marvel       1939    Storm      good female
#> 6    Marvel       1939 Mystique       bad female
```

In a way, this does illustrate multiple matches, if you think about it from the `x = publishers` direction. Every publisher that has a match in `y = superheroes` appears multiple times in the result, once for each match. In fact, we're getting the same result as with `inner_join(superheroes, publishers)`, up to variable order (which you should also never rely on in an analysis).



<table border = 1>
<tr>
<td valign="top">
  publishers
  

publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
<td valign="top">
  superheroes
  

name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
<td valign="top">
  inner_join(x = publishers, y = superheroes)
  

publisher    yr_founded  name       alignment   gender 
----------  -----------  ---------  ----------  -------
DC                 1934  Batman     good        male   
DC                 1934  Joker      bad         male   
DC                 1934  Catwoman   bad         female 
Marvel             1939  Magneto    bad         male   
Marvel             1939  Storm      good        female 
Marvel             1939  Mystique   bad         female 


</td>
</tr>
</table>
  
#### semi_join(publishers, superheroes)

> semi_join(x, y): Return all rows from x where there are matching values in y, keeping just columns from x. A semi join differs from an inner join because an inner join will return one row of x for each matching row of y, where a semi join will never duplicate rows of x. This is a filtering join.


```r
(sjps <- semi_join(x = publishers, y = superheroes))
#> Joining, by = "publisher"
#> # A tibble: 2 × 2
#>   publisher yr_founded
#>       <chr>      <int>
#> 1    Marvel       1939
#> 2        DC       1934
```

Now the effects of switching the `x` and `y` roles is more clear. The result resembles `x = publishers`, but the publisher Image is lost, because there are no observations where `publisher == "Image"` in `y = superheroes`.




<table border = 1>
<tr>
<td valign="top">
publishers


publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
<td valign="top">
superheroes


name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
<td valign="top">
semi-join(x = publishers, y = superheroes)


publisher    yr_founded
----------  -----------
Marvel             1939
DC                 1934


</td>
</tr>
</table>

#### left_join(publishers, superheroes)

> left_join(x, y): Return all rows from x, and all columns from x and y. If there are multiple matches between x and y, all combination of the matches are returned. This is a mutating join.


```r
(ljps <- left_join(publishers, superheroes))
#> Joining, by = "publisher"
#> # A tibble: 7 × 5
#>   publisher yr_founded     name alignment gender
#>       <chr>      <int>    <chr>     <chr>  <chr>
#> 1        DC       1934   Batman      good   male
#> 2        DC       1934    Joker       bad   male
#> 3        DC       1934 Catwoman       bad female
#> 4    Marvel       1939  Magneto       bad   male
#> 5    Marvel       1939    Storm      good female
#> 6    Marvel       1939 Mystique       bad female
#> 7     Image       1992     <NA>      <NA>   <NA>
```

We get a similar result as with `inner_join()` but the publisher Image survives in the join, even though no superheroes from Image appear in `y = superheroes`. As a result, Image has `NA`s for `name`, `alignment`, and `gender`.



<table border = 1>
<tr>
<td valign="top">
publishers


publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
<td valign="top">
superheroes


name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
<td valign="top">
left_join(x = publishers, y = superheroes)


publisher    yr_founded  name       alignment   gender 
----------  -----------  ---------  ----------  -------
DC                 1934  Batman     good        male   
DC                 1934  Joker      bad         male   
DC                 1934  Catwoman   bad         female 
Marvel             1939  Magneto    bad         male   
Marvel             1939  Storm      good        female 
Marvel             1939  Mystique   bad         female 
Image              1992  NA         NA          NA     


</td>
</tr>
</table>

#### anti_join(publishers, superheroes)

> anti_join(x, y): Return all rows from x where there are not matching values in y, keeping just columns from x. This is a filtering join.


```r
(ajps <- anti_join(publishers, superheroes))
#> Joining, by = "publisher"
#> # A tibble: 1 × 2
#>   publisher yr_founded
#>       <chr>      <int>
#> 1     Image       1992
```

We keep __only__ publisher Image now (and the variables found in `x = publishers`).



<table border = 1>
  <tr>
<tr>
<td valign="top">
publishers


publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
<td valign="top">
superheroes


name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
<td valign="top">
anti_join(x = publishers, y = superheroes)


publisher    yr_founded
----------  -----------
Image              1992


</td>
</tr>
</table>

#### full_join(superheroes, publishers)

> full_join(x, y): Return all rows and all columns from both x and y. Where there are not matching values, returns NA for the one missing. This is a mutating join.


```r
(fjsp <- full_join(superheroes, publishers))
#> Joining, by = "publisher"
#> # A tibble: 8 × 5
#>       name alignment gender         publisher yr_founded
#>      <chr>     <chr>  <chr>             <chr>      <int>
#> 1  Magneto       bad   male            Marvel       1939
#> 2    Storm      good female            Marvel       1939
#> 3 Mystique       bad female            Marvel       1939
#> 4   Batman      good   male                DC       1934
#> 5    Joker       bad   male                DC       1934
#> 6 Catwoman       bad female                DC       1934
#> 7  Hellboy      good   male Dark Horse Comics         NA
#> 8     <NA>      <NA>   <NA>             Image       1992
```

We get all rows of `x = superheroes` plus a new row from `y = publishers`, containing the publisher Image. We get all variables from `x = superheroes` AND all variables from `y = publishers`. Any row that derives solely from one table or the other carries `NA`s in the variables found only in the other table.



<table border = 1>
<tr>
<td valign="top">
  superheroes
  

name       alignment   gender   publisher         
---------  ----------  -------  ------------------
Magneto    bad         male     Marvel            
Storm      good        female   Marvel            
Mystique   bad         female   Marvel            
Batman     good        male     DC                
Joker      bad         male     DC                
Catwoman   bad         female   DC                
Hellboy    good        male     Dark Horse Comics 


</td>
<td valign="top">
  publishers
  

publisher    yr_founded
----------  -----------
DC                 1934
Marvel             1939
Image              1992


</td>
<td valign="top">
  full_join(x = superheroes, y = publishers)
  

name       alignment   gender   publisher            yr_founded
---------  ----------  -------  ------------------  -----------
Magneto    bad         male     Marvel                     1939
Storm      good        female   Marvel                     1939
Mystique   bad         female   Marvel                     1939
Batman     good        male     DC                         1934
Joker      bad         male     DC                         1934
Catwoman   bad         female   DC                         1934
Hellboy    good        male     Dark Horse Comics            NA
NA         NA          NA       Image                      1992


</td>
</tr>
</table>

#### sessionInfo()


```r
devtools::session_info()
#> Session info -------------------------------------------------------------
#>  setting  value                       
#>  version  R version 3.3.1 (2016-06-21)
#>  system   x86_64, darwin13.4.0        
#>  ui       X11                         
#>  language (EN)                        
#>  collate  en_CA.UTF-8                 
#>  tz       America/Vancouver           
#>  date     2016-10-06
#> Packages -----------------------------------------------------------------
#>  package    * version     date       source                            
#>  assertthat   0.1         2013-12-06 CRAN (R 3.2.0)                    
#>  DBI          0.4-1       2016-05-08 cran (@0.4-1)                     
#>  devtools     1.12.0.9000 2016-09-26 Github (hadley/devtools@26c507b)  
#>  digest       0.6.10      2016-08-02 cran (@0.6.10)                    
#>  dplyr      * 0.5.0       2016-06-24 CRAN (R 3.3.0)                    
#>  evaluate     0.9         2016-04-29 CRAN (R 3.3.0)                    
#>  formatR      1.4         2016-05-09 CRAN (R 3.3.0)                    
#>  highr        0.6         2016-05-09 CRAN (R 3.3.0)                    
#>  htmltools    0.3.5       2016-03-21 CRAN (R 3.2.4)                    
#>  knitr        1.14.2      2016-09-07 Github (yihui/knitr@f02600d)      
#>  magrittr     1.5         2014-11-22 CRAN (R 3.2.0)                    
#>  memoise      1.0.0       2016-01-29 CRAN (R 3.2.3)                    
#>  R6           2.1.3       2016-08-19 cran (@2.1.3)                     
#>  Rcpp         0.12.7      2016-09-05 cran (@0.12.7)                    
#>  readr      * 1.0.0.9000  2016-09-07 Github (hadley/readr@37d6eda)     
#>  rmarkdown    1.0.9014    2016-09-20 Github (rstudio/rmarkdown@81c2092)
#>  stringi      1.1.1       2016-05-27 cran (@1.1.1)                     
#>  stringr      1.1.0       2016-08-19 CRAN (R 3.3.0)                    
#>  tibble       1.2         2016-08-26 cran (@1.2)                       
#>  withr        1.0.2       2016-06-20 cran (@1.0.2)                     
#>  yaml         2.1.13      2014-06-12 CRAN (R 3.2.0)
```

