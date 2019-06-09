
# Fisheries Collapse Analysis

Chandler Sutherland and Kaila Souza

## The Database

We will use data from the [RAM Legacy Stock Assessment
Database](http://ramlegacy.marinebiodiversity.ca/ram-legacy-stock-assessment-database)

First, we load in the necessary libraries, using the ‘readxl’ package to
load .xls files.

``` r
library("tidyverse")
library("readxl")
```

## Reading in the Tables

``` r
## old link not working today:
#download.file("https://depts.washington.edu/ramlegac/wordpress/databaseVersions/RLSADB_v3.0_(assessment_data_only)_excel.zip", 

# backup copy for class:

if(file.exists('ramlegacy.zip')){
    download.file("https://github.com/espm-157/fish-template/releases/download/data/ramlegacy.zip", 
             "ramlegacy.zip")
    }
path <- unzip("ramlegacy.zip")  #unzip the .xls files
sheets <- readxl::excel_sheets(path) #use the readxl package to identify sheet names 
ram <- lapply(sheets, readxl::read_excel, path = path)  #read the data from all 3 sheets into a list
names(ram) <- sheets # give the list of datatables their assigned sheet names

## check your names
names(ram)
```

    ##  [1] "area"                    "assessment"             
    ##  [3] "assessmethod"            "assessor"               
    ##  [5] "biometrics"              "bioparams"              
    ##  [7] "bioparams_ids_views"     "bioparams_units_views"  
    ##  [9] "bioparams_values_views"  "management"             
    ## [11] "stock"                   "taxonomy"               
    ## [13] "timeseries"              "timeseries_ids_views"   
    ## [15] "timeseries_units_views"  "timeseries_values_views"
    ## [17] "tsmetrics"

# Investigating the North-Atlantic Cod

First, we seek to replicate the following figure from the Millennium
Ecosystem Assessment Project using the RAM data.

![](https://github.com/espm-157/website/raw/master/static/img/cod.jpg)

## Plotting Total Cod Catch in Canada

Here we calculate and plot the catch in metric tons (MT) of Atlantic Cod
from Canada using the RAM data.

``` r
cod_tsn <- ram$taxonomy %>%
  filter(genus == 'Gadus', species == "morhua") %>%
  select(tsn) %>%
  left_join(ram$stock, by = 'tsn') %>%
  left_join(ram$area, by = 'areaid') %>%
  left_join(ram$timeseries, by = 'stockid') %>%
  left_join(ram$tsmetrics, by = c('tsid' = 'tsunique')) %>%
  filter(tscategory == 'CATCH or LANDINGS') %>%
  filter(country == 'Canada')
cod_tsn
```

    ## # A tibble: 894 x 24
    ##       tsn stockid scientificname commonname areaid stocklong.x region
    ##     <dbl> <chr>   <chr>          <chr>      <chr>  <chr>       <chr> 
    ##  1 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  2 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  3 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  4 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  5 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  6 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  7 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  8 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ##  9 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ## 10 164712 COD2J3~ Gadus morhua   Atlantic ~ Canad~ Atlantic c~ Canad~
    ## # ... with 884 more rows, and 17 more variables: inmyersdb <dbl>,
    ## #   myersstockid <chr>, country <chr>, areatype <chr>, areacode <chr>,
    ## #   areaname <chr>, alternateareaname <chr>, assessid <chr>,
    ## #   stocklong.y <chr>, tsid <chr>, tsyear <dbl>, tsvalue <dbl>,
    ## #   tscategory <chr>, tsshort <chr>, tslong <chr>, tsunitsshort <chr>,
    ## #   tsunitslong <chr>

``` r
cod_tsn %>% select(areaname, areacode, areaid, region, country) %>%
  distinct()
```

    ## # A tibble: 6 x 5
    ##   areaname                     areacode areaid        region       country
    ##   <chr>                        <chr>    <chr>         <chr>        <chr>  
    ## 1 Southern Labrador-Eastern N~ 2J3KL    Canada-DFO-2~ Canada East~ Canada 
    ## 2 Northern Gulf of St. Lawren~ 3Pn4RS   Canada-DFO-3~ Canada East~ Canada 
    ## 3 St. Pierre Bank              3Ps      Canada-DFO-3~ Canada East~ Canada 
    ## 4 Southern Gulf of St. Lawren~ 4T       Canada-DFO-4T Canada East~ Canada 
    ## 5 Eastern Scotian Shelf        4VsW     Canada-DFO-4~ Canada East~ Canada 
    ## 6 Western Scotian Shelf        4X       Canada-DFO-4X Canada East~ Canada

``` r
cod_tsn %>%
  group_by(tsyear) %>%
  summarize(catch_tons = sum(tsvalue, na.rm = TRUE)) %>%
ggplot(aes(tsyear, catch_tons)) + geom_line() + labs(title = 'Fish Landings by Year', x = 'Year', y= 'Catch in Metric Tons')
```

![](fish-assignment_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

Compared to the graph presented by the RAM data scientists, our plot
shows a similar trend in the exploitation and collapse of cod
populations, but has different y units. We are unsure as to why.

-----

## Stock Collapses

We seek to replicate the temporal trend in stock declines shown in [Worm
et al 2006](http://doi.org/10.1126/science.1132294):

![](https://espm-157.carlboettiger.info/img/worm2006.jpg)

The Worm et al. plot includes years 1950 to 2005 and is plotting the
trajectory of collapsed fish and invertebrate taxa as calculated by year
and cumulative collapse, represented by diamonds and triangles
respectively. Black represents all species, blue represents
species-poor, and red represents species rich. The regression lines are
calculated via best-fit models, and are corrected for temporal auto
correlation.

## Plotting Total Taxa Caught Worldwide 1950-2006

Here we create a new table, general\_tsn, as a broader version of
cod\_tsn to plot the number of total taxa caught each year from 1950 to
2006, using geom\_point(), group\_by(), and tally().

``` r
general_tsn <- ram$taxonomy %>%
  left_join(ram$stock, by = 'tsn') %>%
  left_join(ram$area, by = 'areaid') %>%
  left_join(ram$timeseries, by = 'stockid') %>%
  left_join(ram$tsmetrics, by = c('tsid' = 'tsunique')) %>%
  filter(tscategory == 'CATCH or LANDINGS') 

worm_imitation <- general_tsn %>% 
  filter(!is.na(tsvalue)) %>%
  select(stockid, tsyear) %>%
  filter(tsyear >= 1950, tsyear <= 2006) %>%
  group_by(tsyear) %>%
  tally()


ggplot(worm_imitation, aes(x = tsyear, y = n)) + 
  geom_point() + labs(x = 'Year', y = 'Number of Total Taxa caught', 
                      title = 'Number of Total Taxa Caught vs Year')
```

![](fish-assignment_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## Removing Incomplete Datasets

Species can either have missing data (within a series) or a time range
that just doesn’t span the full interval. Here, we group by stockid
instead of year, to build a character vector containing only those
stockids that have data for the full range (1950-2006).

``` r
#here we modify general_tsn to be more useful for this problem
fish_3 <- general_tsn %>%
  select(stockid, tsyear, tsvalue, tsshort)%>%
  filter(tsshort == 'TC') %>%
  select(stockid, year = tsyear, TC = tsvalue)
```

We filter by ‘TC’ instead of ‘Catch or Landings’ to avoid double
counting data that was entered as catch and total catch, which includes
landings/discards.

``` r
#By filtering only those stockids that had 57 distinct entries, we can create a table of just those that have the full range 
stock_id_grouping <- fish_3 %>% 
  select(year, stockid, TC) %>%
  filter(!is.na(TC)) %>%
  distinct() %>%
  filter(year >= 1950, year <= 2006) %>%
  group_by(stockid)%>%
  tally() %>%
  filter( n == 57) %>%
  select(stockid)

#pulling out the column stockid to print a vector
stock_id_full_range <- stock_id_grouping %>%
  pull(stockid)

stock_id_full_range
```

    ##  [1] "ACADREDGOMGB"      "ALBAIO"            "ALBANATL"         
    ##  [4] "ARFLOUNDPCOAST"    "ATBTUNAEATL"       "ATBTUNAWATL"      
    ##  [7] "ATHAL5YZ"          "BGROCKPCOAST"      "BHEADSHARATL"     
    ## [10] "BIGEYEATL"         "BIGEYEIO"          "BKCDLFENI"        
    ## [13] "BLACKROCKNPCOAST"  "BLACKROCKSPCOAST"  "BLUEROCKCAL"      
    ## [16] "BNSNZ"             "BOCACCSPCOAST"     "CABEZSCAL"        
    ## [19] "CHAKESA"           "CHILISPCOAST"      "CMACKPCOAST"      
    ## [22] "COD2J3KL"          "COWCODSCAL"        "CROCKPCOAST"      
    ## [25] "CROCKWCVANISOGQCI" "CTRACSA"           "DEEPCHAKESA"      
    ## [28] "DSOLEPCOAST"       "ESOLEPCOAST"       "GRNSTROCKPCOAST"  
    ## [31] "GRSPROCKNCAL"      "GRSPROCKSCAL"      "LNOSESKAPCOAST"   
    ## [34] "OROUGHYNZMEC"      "PHALNPAC"          "POPERCHPCOAST"    
    ## [37] "PSOLEPCOAST"       "RROCKLOBSTERCRA3"  "SABLEFPCOAST"     
    ## [40] "SBT"               "SKJCIO"            "SKJCWPAC"         
    ## [43] "SKJEATL"           "SNROCKPCOAST"      "SPSDOGPCOAST"     
    ## [46] "SWORDEPAC"         "SWORDIO"           "SWORDMED"         
    ## [49] "SWORDNATL"         "SWORDSATL"         "TARAKNZ"          
    ## [52] "WROCKPCOAST"       "YELLCCODGOM"       "YELLGB"           
    ## [55] "YELLSNEMATL"       "YEYEROCKPCOAST"    "YFINATL"

56 taxa have data for the full range.

``` r
length(stock_id_full_range)
```

    ## [1] 57

## Examining Which Fisheries Have Collapsed

A fishery may be considered *collapsed* when total catch (TC) falls
below 10% of its peak. For the 88 stocks with complete data sets, we
created a new tidy table including columns: `stockid`, `TC`, `year`,
`collapsed`, and `cumulative`, where `collapsed` is a logical (True or
False) for whether or not that fishery could be considered collapsed in
that year, and `cumulative` is the count of total years the fishery has
been collapsed at that point in time.

``` r
#Add columns collapse and cumulative
collapse_analysis <- fish_3 %>% 
  filter(year >= 1950, year <= 2006) %>%
  right_join(stock_id_grouping) %>%
  group_by(stockid) %>%
  mutate(collapse = (TC < 0.1 * cummax(TC))) %>%
  mutate(cumulative = cumsum(collapse))  

collapse_analysis
```

    ## # A tibble: 3,249 x 5
    ## # Groups:   stockid [57]
    ##    stockid       year    TC collapse cumulative
    ##    <chr>        <dbl> <dbl> <lgl>         <int>
    ##  1 ACADREDGOMGB  1950 34307 FALSE             0
    ##  2 ACADREDGOMGB  1951 30077 FALSE             0
    ##  3 ACADREDGOMGB  1952 21377 FALSE             0
    ##  4 ACADREDGOMGB  1953 16791 FALSE             0
    ##  5 ACADREDGOMGB  1954 12988 FALSE             0
    ##  6 ACADREDGOMGB  1955 13914 FALSE             0
    ##  7 ACADREDGOMGB  1956 14388 FALSE             0
    ##  8 ACADREDGOMGB  1957 18490 FALSE             0
    ##  9 ACADREDGOMGB  1958 16047 FALSE             0
    ## 10 ACADREDGOMGB  1959 15521 FALSE             0
    ## # ... with 3,239 more rows

## Plotting Total Catch

Using `geom_area()` plot the TC per stockid across all years.

``` r
ggplot(collapse_analysis, aes(x = year, y = TC, fill = stockid)) + 
  geom_area() + 
  guides(fill = FALSE) +
  labs(title = 'Total Catch per Year from 1950 to 2006', y = 'Total Catch (MT)')
```

![](fish-assignment_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

## Calculating Percent Collapsed

To replicate the original plot, we must calculate the percent of taxa
collapsed over time. Using the `summarise()` function, and only the core
stocks that have data across the full interval, we build a new tidy
table that gives the fraction of all stocks that are collapsed in each
year and include a cumulative column that gives the fraction of all
years (between 1950 and each year) that has experience at least one
collapse.

``` r
percent_collapse_analysis <- collapse_analysis %>% 
  ungroup(stockid) %>%
  group_by(year) %>%
  summarise(percent_collapse = 100*sum(collapse)/length(stock_id_full_range), 
            cumulative = 100*sum(cumulative != 0)/57)

percent_collapse_analysis
```

    ## # A tibble: 57 x 3
    ##     year percent_collapse cumulative
    ##    <dbl>            <dbl>      <dbl>
    ##  1  1950             0          0   
    ##  2  1951             0          0   
    ##  3  1952             0          0   
    ##  4  1953             1.75       1.75
    ##  5  1954             1.75       3.51
    ##  6  1955             0          3.51
    ##  7  1956             1.75       5.26
    ##  8  1957             0          5.26
    ##  9  1958             0          5.26
    ## 10  1959             0          5.26
    ## # ... with 47 more rows

## Plotting Proportion Collapsed Over Time

``` r
ggplot(percent_collapse_analysis, aes(x = year)) + 
  geom_line(aes(y = percent_collapse, color = 'green')) + 
  geom_line(aes(y= cumulative, color = 'blue')) +
  scale_y_reverse() + 
  labs(y = 'Percent of Taxa Collapsed', x = 'Year', 
       title = 'Percent of Taxa Collapsed vs Time') + 
  scale_color_discrete(name = 'Legend', 
                       breaks = c('blue', 'green'), 
                       labels = c('Percent Collapsed Each Year', 'Cumulative Collapsed'))
```

![](fish-assignment_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

This graph is showing us the percent of collapsed taxa, with collapse
defined as less than 10% of the Total Catch. Using this metric is
problematic because total catch is not equivalent to true population
numbers of fish in the ocean. Fishing technology vastly improved in the
1970s, as you can see in our first graph, and so total catch might have
been low in the 1950s and 1960s while the population of fish was
healthy. Using the cummax function we avoid any false collapses.

Even using cummax, there are still some years that show collapse using
our analysis but aren’t necessarily collapsed. For example, stockid
ARFLOUNDPCOAST shows collapse in 1954, but this was most likely a bad
year for fishing as the following years show rebounded catch.
