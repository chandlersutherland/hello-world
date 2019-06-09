Lumpsuckers: Ping-pong Balls of the Sea
================
Chandler Sutherland and Clara Park

Data Set
========

This analysis is based on a recent paper in Nature Ecology and Evolution, [**Mapping the global potential for marine aquaculture**](https://www.nature.com/articles/s41559-017-0257-9). The authors used multiple constraints including ship traffic, dissolved oxygen, bottom depth and more, to limit and map areas suitable for aquaculture.

![](./images/aquaculture_map.png)

We are going to use a similar, but much more simplified approach in this analysis, beginning with mapping potential areas of marine aquaculture for the super cute Pacific spiny lumpsucker (*Eumicrotremus orbis*)

![They have adhesive pelvic disks! How cute!](./images/lumpsucker.png)

To do this we are going to use the following spatial data:

**1. Sea Surface Temperature** (raster data)
**2. Net Primary Productivity** (raster data)
**3. Marine Protected Areas** (vector data)

Mapping Marine Protected Areas
------------------------------

First, let's examine some background information on the lumpsucker.

From [NOAA's species description](http://www.nmfs.noaa.gov/speciesid/fish_page/fish68a.html):

> A lot of people who see Pacific spiny lumpsuckers for the first time describe them as a ping-pong ball with fins. They are tiny and very inefficient swimmers, found most often in kelp or eelgrass beds attached to a rock or a log no deeper than 500 feet. They are quite common, ranging from the waters off the Washington coast, up around the arc of the Aleutian Islands, to the Asian mainland and the northern islands of Japan, and in the Bering Sea. A giant Pacific spiny lumpsucker is five inches long, but most are closer to an inch. Scuba divers are their biggest fans because the little fellows will eat right out of their hands.

Key information for optimal growth:

-   Sea surface temperatures between **12 and 18 degrees Celsius**
-   Net Primary Productivity between **2.6 and 3 mgC/m2/day**

### Load and Visualize Marine Protected Areas data

We'll start with a data file of Marine Protected Areas monitored by the US Federal government on the west coast: `mpas_westcoast.shp`.

``` r
mpas_westcoast <- st_read('shapefiles')
```

    ## Reading layer `mpas_westcoast' from data source `/Users/clarapark/UC Berkeley/Fall 2018/ESPM 157/2018-spatial-spatial_clara_chandler/spatial/shapefiles' using driver `ESRI Shapefile'
    ## Simple feature collection with 348 features and 25 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: -2707616 ymin: -457193.8 xmax: -1950642 ymax: 1553906
    ## epsg (SRID):    NA
    ## proj4string:    +proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +datum=NAD83 +units=m +no_defs

``` r
mpas_westcoast
```

    ## Simple feature collection with 348 features and 25 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: -2707616 ymin: -457193.8 xmax: -1950642 ymax: 1553906
    ## epsg (SRID):    NA
    ## proj4string:    +proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +datum=NAD83 +units=m +no_defs
    ## First 10 features:
    ##    Site_ID Area_KM_To Date_GIS_U Shape_Leng   Shape_Area
    ## 1      CA1 253.722000       <NA> 1.19425223 2.732805e-02
    ## 2     CA10   4.244570       <NA> 0.17709063 4.346111e-04
    ## 3     CA11   0.362449       <NA> 0.02441133 3.708939e-05
    ## 4    CA113  46.220300       <NA> 0.51142581 4.716015e-03
    ## 5    CA116   0.357710       <NA> 0.02575429 3.436752e-05
    ## 6     CA12   3.549420       <NA> 0.13153472 3.630051e-04
    ## 7     CA13  46.220300       <NA> 0.51142561 4.716016e-03
    ## 8    CA136 106.430000       <NA> 0.39291855 1.037097e-02
    ## 9    CA137  11.806200       <NA> 0.15762589 1.149228e-03
    ## 10   CA138  65.324700       <NA> 0.43039476 6.365586e-03
    ##                                                          Site_Name
    ## 1  Redwoods National Park ASBS State Water Quality Protection Area
    ## 2   Point Reyes Headlands ASBS State Water Quality Protection Area
    ## 3            Double Point ASBS State Water Quality Protection Area
    ## 4                                     Farallon Islands Game Refuge
    ## 5                                          Scripps Coastal Reserve
    ## 6            Duxbury Reef ASBS State Water Quality Protection Area
    ## 7        Farallon Islands ASBS State Water Quality Protection Area
    ## 8         Richardson Rock (San Miguel Island) State Marine Reserve
    ## 9             Judith Rock (San Miguel Island) State Marine Reserve
    ## 10           Harris Point (San Miguel Island) State Marine Reserve
    ##                    Site_Label Gov_Level      State      NS_Full
    ## 1             Redwood NP ASBS     State California       Member
    ## 2  Point Reyes Headlands ASBS     State California       Member
    ## 3           Double Point ASBS     State California       Member
    ## 4         Farallon Islands GL     State California Not Eligible
    ## 5                  Scripps CR     State California     Eligible
    ## 6           Duxbury Reef ASBS     State California       Member
    ## 7        Farallon Island ASBS     State California       Member
    ## 8         Richardson Rock SMR     State California       Member
    ## 9             Judith Rock SMR     State California       Member
    ## 10           Harris Point SMR     State California       Member
    ##                Prot_Lvl                        Mgmt_Plan
    ## 1  Uniform Multiple Use MPA Programmatic Management Plan
    ## 2  Uniform Multiple Use MPA Programmatic Management Plan
    ## 3  Uniform Multiple Use MPA Programmatic Management Plan
    ## 4  Uniform Multiple Use               No Management Plan
    ## 5  Uniform Multiple Use    Site-Specific Management Plan
    ## 6  Uniform Multiple Use MPA Programmatic Management Plan
    ## 7  Uniform Multiple Use MPA Programmatic Management Plan
    ## 8               No Take MPA Programmatic Management Plan
    ## 9               No Take MPA Programmatic Management Plan
    ## 10              No Take MPA Programmatic Management Plan
    ##                                                                               Mgmt_Agen
    ## 1                                        California State Water Resources Control Board
    ## 2                                        California State Water Resources Control Board
    ## 3                                        California State Water Resources Control Board
    ## 4                                            California Department of Fish and Wildlife
    ## 5  University of California Natural Reserve Manager, University of California San Diego
    ## 6                                        California State Water Resources Control Board
    ## 7                                        California State Water Resources Control Board
    ## 8                                            California Department of Fish and Wildlife
    ## 9                                            California Department of Fish and Wildlife
    ## 10                                           California Department of Fish and Wildlife
    ##                                         Fish_Rstr       Pri_Con_Fo
    ## 1                            No Site Restrictions Natural Heritage
    ## 2                            No Site Restrictions Natural Heritage
    ## 3                            No Site Restrictions Natural Heritage
    ## 4                            No Site Restrictions Natural Heritage
    ## 5                            No Site Restrictions Natural Heritage
    ## 6                            No Site Restrictions Natural Heritage
    ## 7                            No Site Restrictions Natural Heritage
    ## 8  Commercial and Recreational Fishing Prohibited Natural Heritage
    ## 9  Commercial and Recreational Fishing Prohibited Natural Heritage
    ## 10 Commercial and Recreational Fishing Prohibited Natural Heritage
    ##          Cons_Focus Prot_Focus Permanence  Constancy Estab_Yr
    ## 1  Natural Heritage  Ecosystem  Permanent Year-round     1974
    ## 2  Natural Heritage  Ecosystem  Permanent Year-round     1974
    ## 3  Natural Heritage  Ecosystem  Permanent Year-round     1974
    ## 4  Natural Heritage  Ecosystem  Permanent Year-round     1971
    ## 5  Natural Heritage  Ecosystem  Permanent Year-round     1965
    ## 6  Natural Heritage  Ecosystem  Permanent Year-round     1974
    ## 7  Natural Heritage  Ecosystem  Permanent Year-round     1974
    ## 8  Natural Heritage  Ecosystem  Permanent Year-round     2003
    ## 9  Natural Heritage  Ecosystem  Permanent Year-round     2003
    ## 10 Natural Heritage  Ecosystem  Permanent Year-round     2003
    ##                                                                           URL
    ## 1  http://www.waterboards.ca.gov/water_issues/programs/ocean/asbs_areas.shtml
    ## 2  http://www.waterboards.ca.gov/water_issues/programs/ocean/asbs_areas.shtml
    ## 3  http://www.waterboards.ca.gov/water_issues/programs/ocean/asbs_areas.shtml
    ## 4                                 http://www.dfg.ca.gov/wildlife/gamerefuges/
    ## 5                                                                        <NA>
    ## 6  http://www.waterboards.ca.gov/water_issues/programs/ocean/asbs_areas.shtml
    ## 7  http://www.waterboards.ca.gov/water_issues/programs/ocean/asbs_areas.shtml
    ## 8                                                 http://www.dfg.ca.gov/mlpa/
    ## 9                                                 http://www.dfg.ca.gov/mlpa/
    ## 10                                                http://www.dfg.ca.gov/mlpa/
    ##          Vessel       Anchor Area_KM_Ma FID                       geometry
    ## 1          <NA>         <NA> 249.424000   0 MULTIPOLYGON (((-2291279 81...
    ## 2          <NA>         <NA>   4.237840   0 MULTIPOLYGON (((-2319199 38...
    ## 3          <NA>         <NA>   0.343696   0 MULTIPOLYGON (((-2300090 37...
    ## 4    Restricted Unrestricted  46.063300   0 MULTIPOLYGON (((-2326351 35...
    ## 5          <NA>         <NA>   0.347766   0 MULTIPOLYGON (((-1960544 -2...
    ## 6          <NA>         <NA>   3.485040   0 MULTIPOLYGON (((-2297075 37...
    ## 7          <NA>         <NA>  46.063300   0 MULTIPOLYGON (((-2326351 35...
    ## 8  Unrestricted Unrestricted 106.421000   0 MULTIPOLYGON (((-2217966 -1...
    ## 9  Unrestricted Unrestricted  11.791300   0 MULTIPOLYGON (((-2215331 -1...
    ## 10 Unrestricted Unrestricted  65.296800   0 MULTIPOLYGON (((-2201709 -9...

### Visualize These Protected Areas

``` r
plot(mpas_westcoast["State"])
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-2-1.png)

### Protected Marine Habitat by Agency/State

Here we breakdown the protected marine habitat by the agency or state in charge of it's maintenance, so later on we can determine who is charge of protected the adorable lumpsucker.

``` r
most_protective <- mpas_westcoast %>% 
  mutate(protected_area = st_area(mpas_westcoast)) %>%
  group_by(State) %>%
  summarise(sum_area = sum(protected_area)) %>%
  arrange(desc(sum_area))
most_protective
```

    ## Simple feature collection with 10 features and 2 fields
    ## geometry type:  GEOMETRY
    ## dimension:      XY
    ## bbox:           xmin: -2707616 ymin: -457193.8 xmax: -1950642 ymax: 1553906
    ## epsg (SRID):    NA
    ## proj4string:    +proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +datum=NAD83 +units=m +no_defs
    ## # A tibble: 10 x 3
    ##    State             sum_area                                     geometry
    ##    <fct>             <S3: units>                            <GEOMETRY [m]>
    ##  1 National Marine … 38655297133… MULTIPOLYGON (((-2022333 -317282.9, -20…
    ##  2 National Marine … " 325463601… MULTIPOLYGON (((-2093543 -191307.6, -20…
    ##  3 National Park Se… "  58492144… MULTIPOLYGON (((-2103158 -191604.2, -21…
    ##  4 California        "  46104803… MULTIPOLYGON (((-1957131 -333326.9, -19…
    ##  5 Washington        "  22200976… MULTIPOLYGON (((-2114207 1331625, -2114…
    ##  6 National Wildlif… "   6185260… MULTIPOLYGON (((-1956125 -332742.8, -19…
    ##  7 Bureau of Ocean … "   2232733… POLYGON ((-2131762 -84901.34, -2131745 …
    ##  8 National Estuari… "   1033506… MULTIPOLYGON (((-1956380 -332373.5, -19…
    ##  9 Oregon            "    276911… MULTIPOLYGON (((-2291710 852562.4, -229…
    ## 10 Marine National … "     77084… MULTIPOLYGON (((-1964285 -319058.9, -19…

Sea Surface Temperature Data
----------------------------

**Sea Surface Temperature**

Our raw data contains 5 files with the annual average sea surface temperature for our region, and we will combine them into one raster file with the sea surface temperature (sst) for the entire time period 2008-2012.

### Reading in raster data

Here we create a single sst raster layer.

``` r
rasters <- list.files("rasters", pattern = "average", full.names = TRUE)
sst <- map(rasters, raster) 
sst
```

    ## [[1]]
    ## class       : RasterLayer 
    ## dimensions  : 480, 408, 195840  (nrow, ncol, ncell)
    ## resolution  : 0.04166185, 0.04165702  (x, y)
    ## extent      : -131.9848, -114.9867, 29.99305, 49.98842  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +ellps=WGS84 +no_defs 
    ## data source : /Users/clarapark/UC Berkeley/Fall 2018/ESPM 157/2018-spatial-spatial_clara_chandler/spatial/rasters/average_annual_sst_2008.tif 
    ## names       : average_annual_sst_2008 
    ## values      : 278.7, 301.445  (min, max)
    ## 
    ## 
    ## [[2]]
    ## class       : RasterLayer 
    ## dimensions  : 480, 408, 195840  (nrow, ncol, ncell)
    ## resolution  : 0.04166185, 0.04165702  (x, y)
    ## extent      : -131.9848, -114.9867, 29.99305, 49.98842  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +ellps=WGS84 +no_defs 
    ## data source : /Users/clarapark/UC Berkeley/Fall 2018/ESPM 157/2018-spatial-spatial_clara_chandler/spatial/rasters/average_annual_sst_2009.tif 
    ## names       : average_annual_sst_2009 
    ## values      : 278.08, 301.5  (min, max)
    ## 
    ## 
    ## [[3]]
    ## class       : RasterLayer 
    ## dimensions  : 480, 408, 195840  (nrow, ncol, ncell)
    ## resolution  : 0.04166185, 0.04165702  (x, y)
    ## extent      : -131.9848, -114.9867, 29.99305, 49.98842  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +ellps=WGS84 +no_defs 
    ## data source : /Users/clarapark/UC Berkeley/Fall 2018/ESPM 157/2018-spatial-spatial_clara_chandler/spatial/rasters/average_annual_sst_2010.tif 
    ## names       : average_annual_sst_2010 
    ## values      : 279.92, 300.96  (min, max)
    ## 
    ## 
    ## [[4]]
    ## class       : RasterLayer 
    ## dimensions  : 480, 408, 195840  (nrow, ncol, ncell)
    ## resolution  : 0.04166185, 0.04165702  (x, y)
    ## extent      : -131.9848, -114.9867, 29.99305, 49.98842  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +ellps=WGS84 +no_defs 
    ## data source : /Users/clarapark/UC Berkeley/Fall 2018/ESPM 157/2018-spatial-spatial_clara_chandler/spatial/rasters/average_annual_sst_2011.tif 
    ## names       : average_annual_sst_2011 
    ## values      : 278.86, 307.2733  (min, max)
    ## 
    ## 
    ## [[5]]
    ## class       : RasterLayer 
    ## dimensions  : 480, 408, 195840  (nrow, ncol, ncell)
    ## resolution  : 0.04166185, 0.04165702  (x, y)
    ## extent      : -131.9848, -114.9867, 29.99305, 49.98842  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +ellps=WGS84 +no_defs 
    ## data source : /Users/clarapark/UC Berkeley/Fall 2018/ESPM 157/2018-spatial-spatial_clara_chandler/spatial/rasters/average_annual_sst_2012.tif 
    ## names       : average_annual_sst_2012 
    ## values      : 278.13, 310.2  (min, max)

### Visualization & Exploration

``` r
plot(sst[[1]])
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-4-1.png)

``` r
plot(sst[[2]])
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-4-2.png)

``` r
plot(sst[[3]])
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-4-3.png)

``` r
plot(sst[[4]])
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-4-4.png)

``` r
plot(sst[[5]])
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-4-5.png)

``` r
map(sst, hist)
```

    ## Warning in .hist1(x, maxpixels = maxpixels, main = main, plot = plot, ...):
    ## 51% of the raster cells were used. 100000 values used.

    ## Warning in .hist1(x, maxpixels = maxpixels, main = main, plot = plot, ...):
    ## 51% of the raster cells were used. 100000 values used.

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-5-1.png)

    ## Warning in .hist1(x, maxpixels = maxpixels, main = main, plot = plot, ...):
    ## 51% of the raster cells were used. 100000 values used.

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-5-2.png)

    ## Warning in .hist1(x, maxpixels = maxpixels, main = main, plot = plot, ...):
    ## 51% of the raster cells were used. 100000 values used.

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-5-3.png)

    ## Warning in .hist1(x, maxpixels = maxpixels, main = main, plot = plot, ...):
    ## 51% of the raster cells were used. 100000 values used.

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-5-4.png)![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-5-5.png)

    ## [[1]]
    ## $breaks
    ##  [1] 278 279 280 281 282 283 284 285 286 287 288 289 290 291 292 293 294
    ## [18] 295 296 297 298 299 300 301 302
    ## 
    ## $counts
    ##  [1]     1     1     9   134  2425  8418 10642 14885 12126 11972 12521
    ## [12] 14166 10116  2394   151    11     4     3     5    10     1     0
    ## [23]     2     3
    ## 
    ## $density
    ##  [1] 0.00001 0.00001 0.00009 0.00134 0.02425 0.08418 0.10642 0.14885
    ##  [9] 0.12126 0.11972 0.12521 0.14166 0.10116 0.02394 0.00151 0.00011
    ## [17] 0.00004 0.00003 0.00005 0.00010 0.00001 0.00000 0.00002 0.00003
    ## 
    ## $mids
    ##  [1] 278.5 279.5 280.5 281.5 282.5 283.5 284.5 285.5 286.5 287.5 288.5
    ## [12] 289.5 290.5 291.5 292.5 293.5 294.5 295.5 296.5 297.5 298.5 299.5
    ## [23] 300.5 301.5
    ## 
    ## $xname
    ## [1] "v"
    ## 
    ## $equidist
    ## [1] TRUE
    ## 
    ## attr(,"class")
    ## [1] "histogram"
    ## 
    ## [[2]]
    ## $breaks
    ##  [1] 278 279 280 281 282 283 284 285 286 287 288 289 290 291 292 293 294
    ## [18] 295 296 297 298 299 300 301 302
    ## 
    ## $counts
    ##  [1]     1    13    32   109   521  4746 10163 13551 12778 11096 11906
    ## [12] 14439 14767  5677   143    17     9    10    11     4     1     4
    ## [23]     0     2
    ## 
    ## $density
    ##  [1] 0.00001 0.00013 0.00032 0.00109 0.00521 0.04746 0.10163 0.13551
    ##  [9] 0.12778 0.11096 0.11906 0.14439 0.14767 0.05677 0.00143 0.00017
    ## [17] 0.00009 0.00010 0.00011 0.00004 0.00001 0.00004 0.00000 0.00002
    ## 
    ## $mids
    ##  [1] 278.5 279.5 280.5 281.5 282.5 283.5 284.5 285.5 286.5 287.5 288.5
    ## [12] 289.5 290.5 291.5 292.5 293.5 294.5 295.5 296.5 297.5 298.5 299.5
    ## [23] 300.5 301.5
    ## 
    ## $xname
    ## [1] "v"
    ## 
    ## $equidist
    ## [1] TRUE
    ## 
    ## attr(,"class")
    ## [1] "histogram"
    ## 
    ## [[3]]
    ## $breaks
    ##  [1] 279 280 281 282 283 284 285 286 287 288 289 290 291 292 293 294 295
    ## [18] 296 297 298 299 300 301
    ## 
    ## $counts
    ##  [1]     2     9    28   259  3389 10588 17590 12847 14031 11653 14660
    ## [12]  9768  4703   440     9     6     3     7     0     0     6     2
    ## 
    ## $density
    ##  [1] 0.00002 0.00009 0.00028 0.00259 0.03389 0.10588 0.17590 0.12847
    ##  [9] 0.14031 0.11653 0.14660 0.09768 0.04703 0.00440 0.00009 0.00006
    ## [17] 0.00003 0.00007 0.00000 0.00000 0.00006 0.00002
    ## 
    ## $mids
    ##  [1] 279.5 280.5 281.5 282.5 283.5 284.5 285.5 286.5 287.5 288.5 289.5
    ## [12] 290.5 291.5 292.5 293.5 294.5 295.5 296.5 297.5 298.5 299.5 300.5
    ## 
    ## $xname
    ## [1] "v"
    ## 
    ## $equidist
    ## [1] TRUE
    ## 
    ## attr(,"class")
    ## [1] "histogram"
    ## 
    ## [[4]]
    ## $breaks
    ##  [1] 278 280 282 284 286 288 290 292 294 296 298 300 302 304 306 308
    ## 
    ## $counts
    ##  [1]    12    73  4911 30928 28887 27913  7223    19    10    10     0
    ## [12]     9     3     0     2
    ## 
    ## $density
    ##  [1] 0.000060 0.000365 0.024555 0.154640 0.144435 0.139565 0.036115
    ##  [8] 0.000095 0.000050 0.000050 0.000000 0.000045 0.000015 0.000000
    ## [15] 0.000010
    ## 
    ## $mids
    ##  [1] 279 281 283 285 287 289 291 293 295 297 299 301 303 305 307
    ## 
    ## $xname
    ## [1] "v"
    ## 
    ## $equidist
    ## [1] TRUE
    ## 
    ## attr(,"class")
    ## [1] "histogram"
    ## 
    ## [[5]]
    ## $breaks
    ##  [1] 278 280 282 284 286 288 290 292 294 296 298 300 302 304 306 308 310
    ## [18] 312
    ## 
    ## $counts
    ##  [1]     5    48  4838 29793 27009 26682 11404   169    17    17     0
    ## [12]    12     0     3     2     0     1
    ## 
    ## $density
    ##  [1] 0.000025 0.000240 0.024190 0.148965 0.135045 0.133410 0.057020
    ##  [8] 0.000845 0.000085 0.000085 0.000000 0.000060 0.000000 0.000015
    ## [15] 0.000010 0.000000 0.000005
    ## 
    ## $mids
    ##  [1] 279 281 283 285 287 289 291 293 295 297 299 301 303 305 307 309 311
    ## 
    ## $xname
    ## [1] "v"
    ## 
    ## $equidist
    ## [1] TRUE
    ## 
    ## attr(,"class")
    ## [1] "histogram"

``` r
map(sst, summary)
```

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## [[1]]
    ##         average_annual_sst_2008
    ## Min.                   278.7000
    ## 1st Qu.                285.2590
    ## Median                 287.1251
    ## 3rd Qu.                289.1211
    ## Max.                   301.4450
    ## NA's                     0.0000
    ## 
    ## [[2]]
    ##         average_annual_sst_2009
    ## Min.                   278.0800
    ## 1st Qu.                285.7000
    ## Median                 287.7087
    ## 3rd Qu.                289.7131
    ## Max.                   301.5000
    ## NA's                     0.0000
    ## 
    ## [[3]]
    ##         average_annual_sst_2010
    ## Min.                   279.9200
    ## 1st Qu.                285.6547
    ## Median                 287.3980
    ## 3rd Qu.                289.3688
    ## Max.                   300.9600
    ## NA's                     0.0000
    ## 
    ## [[4]]
    ##         average_annual_sst_2011
    ## Min.                   278.8600
    ## 1st Qu.                285.4931
    ## Median                 286.9885
    ## 3rd Qu.                288.7878
    ## Max.                   307.2733
    ## NA's                     0.0000
    ## 
    ## [[5]]
    ##         average_annual_sst_2012
    ## Min.                   278.1300
    ## 1st Qu.                285.5439
    ## Median                 286.9989
    ## 3rd Qu.                289.0252
    ## Max.                   310.2000
    ## NA's                     0.0000

#### Highest Annual Sea Surface Temperature

By observing the summary tables, we can conclude that 2012 had the highest annual sea surface temperature.

``` r
map(sst, summary)
```

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## Warning in .local(object, ...): summary is an estimate based on a sample of 1e+05 cells (51.06% of all cells)

    ## [[1]]
    ##         average_annual_sst_2008
    ## Min.                   278.7000
    ## 1st Qu.                285.2582
    ## Median                 287.1189
    ## 3rd Qu.                289.1225
    ## Max.                   301.4450
    ## NA's                     0.0000
    ## 
    ## [[2]]
    ##         average_annual_sst_2009
    ## Min.                   278.0800
    ## 1st Qu.                285.7005
    ## Median                 287.7048
    ## 3rd Qu.                289.7108
    ## Max.                   301.5000
    ## NA's                     0.0000
    ## 
    ## [[3]]
    ##         average_annual_sst_2010
    ## Min.                   279.9200
    ## 1st Qu.                285.6608
    ## Median                 287.4014
    ## 3rd Qu.                289.3620
    ## Max.                   300.9600
    ## NA's                     0.0000
    ## 
    ## [[4]]
    ##         average_annual_sst_2011
    ## Min.                   278.8600
    ## 1st Qu.                285.4871
    ## Median                 286.9859
    ## 3rd Qu.                288.7816
    ## Max.                   307.2733
    ## NA's                     0.0000
    ## 
    ## [[5]]
    ##         average_annual_sst_2012
    ## Min.                   278.5000
    ## 1st Qu.                285.5472
    ## Median                 287.0019
    ## 3rd Qu.                289.0225
    ## Max.                   310.2000
    ## NA's                     0.0000

### Stacking rasters

To get a single layer of average SST in degrees Celsius we need to first `stack` all layers.

![](images/singletomulti.png)

Here we produce a raster stack across the 5 years, and then visualize using plot.

``` r
sst_stack <- stack(sst)
sst_stack
```

    ## class       : RasterStack 
    ## dimensions  : 480, 408, 195840, 5  (nrow, ncol, ncell, nlayers)
    ## resolution  : 0.04166185, 0.04165702  (x, y)
    ## extent      : -131.9848, -114.9867, 29.99305, 49.98842  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +ellps=WGS84 +no_defs 
    ## names       : average_annual_sst_2008, average_annual_sst_2009, average_annual_sst_2010, average_annual_sst_2011, average_annual_sst_2012 
    ## min values  :                  278.70,                  278.08,                  279.92,                  278.86,                  278.13 
    ## max values  :                301.4450,                301.5000,                300.9600,                307.2733,                310.2000

``` r
plot(sst_stack)
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-7-1.png)

### Convert to Celsius

To make this data more accessible for later calculations, we will convert this raster stack into one, and convert our data from Kelvin to Celsius using a custom R function to perform the conversion:

*C* = *K* − 273.15

``` r
conversion <- function (K) {
  mean(K, na.rm = TRUE) - 273.15
}

conversion
```

    ## function (K) {
    ##   mean(K, na.rm = TRUE) - 273.15
    ## }

We now perform this operation on the raster stack.

``` r
sst_mean_C <- calc(sst_stack, conversion)
sst_mean_C
```

    ## class       : RasterLayer 
    ## dimensions  : 480, 408, 195840  (nrow, ncol, ncell)
    ## resolution  : 0.04166185, 0.04165702  (x, y)
    ## extent      : -131.9848, -114.9867, 29.99305, 49.98842  (xmin, xmax, ymin, ymax)
    ## coord. ref. : +proj=longlat +ellps=WGS84 +no_defs 
    ## data source : in memory
    ## names       : layer 
    ## values      : 4.98, 32.895  (min, max)

Net Primary Production Data
---------------------------

Since Lumpsuckers may be influenced by more than just sea surface temperature, we want to include **Net Primary Production (NPP)** in our analysis. So we need to read that in too and create a rasterstack of ur new `sst_avg` raster and the NPP layer.

#### NPP raster data

Here we read in the NPP data, which is measured in (mgC/m2/day), and plot it to visualize.

``` r
annual_npp <- raster('rasters/annual_npp.tif')
plot(annual_npp)
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-10-1.png)

### Reproject and combination

Before we stack these two layers, we must convert them to the same projection/coordinate reference system.

Let's investigate the coordinate reference systems we are dealing with.

``` r
st_crs(mpas_westcoast)
```

    ## Coordinate Reference System:
    ##   No EPSG code
    ##   proj4string: "+proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +datum=NAD83 +units=m +no_defs"

``` r
crs_npp <- crs(annual_npp)
crs(sst_mean_C)
```

    ## CRS arguments: +proj=longlat +ellps=WGS84 +no_defs

`annual_npp` is equal to our `mpas_westcoast`, but our mean SST layer is different.

We will need to define what the new projection should be by setting a coordinate reference system.

Here, we project our average SST layer into npp's coordinate reference system and prove to yourself they are now equal.

``` r
projected_sst_mean <- projectRaster(sst_mean_C, crs = crs(annual_npp))
identicalCRS(projected_sst_mean, annual_npp)
```

    ## [1] TRUE

``` r
extent(projected_sst_mean)
```

    ## class       : Extent 
    ## xmin        : -3409966 
    ## xmax        : -1351546 
    ## ymin        : -676006.1 
    ## ymax        : 1903184

``` r
extent(annual_npp)
```

    ## class       : Extent 
    ## xmin        : -3409966 
    ## xmax        : -1351546 
    ## ymin        : -676006.1 
    ## ymax        : 1903184

``` r
res(projected_sst_mean)
```

    ## [1] 3380 4470

``` r
res(annual_npp)
```

    ## [1] 3380 4470

The error about non-missing arguments is because in order to have our two raster layers match in extent, our SST layer covers a lot of missing values on its edges which `raster` is encountering in the projection. We can ignore this error for now.

Now we can stack the now matching rasters together using the `stack` function and plot them.

``` r
stacked <- stack(annual_npp, projected_sst_mean)
plot(stacked)
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-13-1.png)

#### Probable Lumpsucker Habitat

Lumpsucker fish grow best in waters that are **between 12 and 18 degrees Celsius.** and with an NPP between **2.6 and 3 mgC/m2/day**, so we predict they will survive not too close to the shore, off of the Oregon and washington coast where both of these conditions are met.

Analysis
--------

Now that our data is prepared, we can move onto **analysis**. For this specific analysis, we need to use the SST and NPP data to find areas along the US West Coast that are suitable for growing lumpsucker fish.

### Sample Points & Extract values from Rasters

Here we use `st_sample()` function to sample 1000 points from the mpas\_westcoast polygons, creating a "simple features collection," representing spatial geometry without attribute data. We then will convert to an `sf` object to be able to extract and retrieve the attribute data of npp and sst for each point.

``` r
mpas_sample <- most_protective %>%
  st_sample(1000) %>%
  st_sf() %>%
  st_join(most_protective, join = st_intersects)
mpas_sample
```

    ## Simple feature collection with 1001 features and 2 fields
    ## geometry type:  POINT
    ## dimension:      XY
    ## bbox:           xmin: -2698750 ymin: -447047.1 xmax: -1955077 ymax: 1545897
    ## epsg (SRID):    NA
    ## proj4string:    +proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +datum=NAD83 +units=m +no_defs
    ## First 10 features:
    ##                                State          sum_area
    ## 1  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 2  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 3  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 4  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 5  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 6  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 7  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 8  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 9  National Marine Fisheries Service 3.86553e+11 [m^2]
    ## 10 National Marine Fisheries Service 3.86553e+11 [m^2]
    ##                      geometry
    ## 1   POINT (-2315236 -72494.7)
    ## 2   POINT (-2650950 794186.1)
    ## 3    POINT (-2540503 1232367)
    ## 4  POINT (-2164181 -250469.8)
    ## 5   POINT (-2473954 900567.7)
    ## 6    POINT (-2570755 1112799)
    ## 7    POINT (-2348418 1319075)
    ## 8    POINT (-2316502 1352334)
    ## 9  POINT (-2144190 -366657.2)
    ## 10   POINT (-2463402 1327061)

#### R Question: Why does your new dataframe of points likely have fewer than 1000 points?

See the `st_sample()` documentation and explain.

### Extracting Raster Values

We now use our sampled points to extract information from the rasters on sea surface temperature and net primary productivity. Also, we project the new vector data into longitude and latitude coordinates for later analysis.

``` r
mpas_extracts <- raster::extract(stacked, mpas_sample) %>%
  as.data.frame()

mpas_combined <- mpas_sample %>%
  mutate(npp = mpas_extracts$annual_npp) %>%
  mutate(sst = mpas_extracts$layer) %>%
  st_transform(crs = '+proj=longlat')
```

### Where are the lumpsuckers?

For the following analyses, remember that Lumpsucker fish grow best in waters that are **between 12 and 18 degrees Celsius.** and with an NPP between **2.6 and 3 mgC/m2/day**

#### Percentage of our Sampled Points with Lumpsuckers

``` r
  lumpsucker <- mpas_combined %>%
    filter(sst >= 12, sst <= 18) %>%
    filter(npp >= 2.6, npp <= 3)
  
nrow(lumpsucker)/10
```

    ## [1] 62.4

#### Minimum Latitude of Lumpsucker Distribution

When we plot just the geometry, we can see that the y minimum, which represents minimum latitude, is 31 degrees.

``` r
geo <- lumpsucker$geometry
```

#### Plot lumpsucker points

``` r
lumpsucker %>%
  select(geometry, npp) %>%
  plot(main = "Net Primary Production in Waters Where Lumpsuckers are found", axes = TRUE)
```

![](spatial-assignment_files/figure-markdown_github/unnamed-chunk-18-1.png)
