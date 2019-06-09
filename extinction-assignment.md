The Sixth Mass Extinction
================
Gi-Gi Lu, Chandler Sutherland

## Mass Extinction: Are we experiencing the sixth great extinction?

## Background

Species origination and extinction is a part of the natural process of
the earth. In order to determine if the current rate of extinction on
earth is abnormal, indicating a sixth mass extinction, we must first
establish a background rate of extinction, and compare it to the current
rate. We will be attempting to recreate a figure from [Ceballos et al
(2015)](http://doi.org/10.1126/sciadv.1400253), using the IUCN Rest API.

# Calculating Extinction Rates

First, we download a list of extinct species names and then build a
function to extract the date they went extinct from the IUCN Rest
API.

``` r
extinct = read_csv("https://espm-157.github.io/extinction-module/extinct.csv")
extinct
```

    # A tibble: 834 x 23
       `Species ID` Kingdom Phylum Class Order Family Genus Species Authority
              <int> <chr>   <chr>  <chr> <chr> <chr>  <chr> <chr>   <chr>    
     1        44072 PLANTAE TRACH~ MAGN~ ROSA~ ROSAC~ Acae~ exigua  A.Gray   
     2       195373 PLANTAE TRACH~ MAGN~ EUPH~ EUPHO~ Acal~ dikulu~ P.A. Duv~
     3        37854 PLANTAE TRACH~ MAGN~ EUPH~ EUPHO~ Acal~ rubrin~ Cronk    
     4       199821 PLANTAE TRACH~ MAGN~ EUPH~ EUPHO~ Acal~ wilderi Merr.    
     5           82 ANIMAL~ ARTHR~ INSE~ EPHE~ ACANT~ Acan~ pecato~ (Burks, ~
     6          167 ANIMAL~ MOLLU~ GAST~ STYL~ ACHAT~ Acha~ abbrev~ Reeve, 1~
     7          170 ANIMAL~ MOLLU~ GAST~ STYL~ ACHAT~ Acha~ buddii  Newcomb,~
     8          173 ANIMAL~ MOLLU~ GAST~ STYL~ ACHAT~ Acha~ caesia  Gulick, ~
     9          174 ANIMAL~ MOLLU~ GAST~ STYL~ ACHAT~ Acha~ casta   Newcomb,~
    10          179 ANIMAL~ MOLLU~ GAST~ STYL~ ACHAT~ Acha~ decora  "(F\xe9r~
    # ... with 824 more rows, and 14 more variables: `Infraspecific
    #   rank` <chr>, `Infraspecific name` <chr>, `Infraspecific
    #   authority` <chr>, `Stock/subpopulation` <chr>, Synonyms <chr>, `Common
    #   names (Eng)` <chr>, `Common names (Fre)` <chr>, `Common names
    #   (Spa)` <chr>, `Red List status` <chr>, `Red List criteria` <chr>, `Red
    #   List criteria version` <dbl>, `Year assessed` <int>, `Population
    #   trend` <chr>, Petitioned <chr>

``` r
#Creates a list of apis to call 
api_call <- map2_chr(extinct$Genus, 
                     extinct$Species, 
                     function(genus, species)
                       paste0("http://api.iucnredlist.org/index/species/", genus, "-", species, ".json"))

#Get data from IUCN Rest API
resp <- map(api_call, 
            function(url){
              Sys.sleep(0.5)
              GET(url)
            })

status <- resp %>% map_int(httr::status_code)
good_resp <- resp[status == 200]
df <- map(good_resp, content, as = 'text') %>%
  map_dfr(fromJSON)

#Create dataframe from good response calls 
df_year <- df %>% select(genus, species, class, phylum, kingdom, rationale) %>% 
  mutate(year = as.numeric(str_extract(rationale, '\\d{4}'))) %>%
  filter(!is.na(year), !is.na(rationale)) %>%
  filter(year >= 1500, year <= 2015)
```

``` r
hist(df_year$year, breaks = 20, xlab = 'Year', main = "Extinction Frequency per Year")
```

![](extinction-assignment_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

Now that we have a useable dataset, we can begin to investigate
extinction rates.

# Comparing Extinctions by Taxa

First, we computed the number of extinctions from 1500 - 1900 and from
1900 to present for a number of taxonomic groups.

``` r
k <- df_year %>% group_by(kingdom) %>% 
  filter(kingdom == "PLANTAE") %>% 
  summarize(early = sum(year <= 1900), late = sum(year > 1900)) %>% 
  rename("taxa" = kingdom) 

p <- df_year %>% group_by(phylum) %>% 
  filter(phylum == "CHORDATA") %>% 
  summarize(early = sum(year <= 1900), late = sum(year > 1900)) %>% 
  rename("taxa" = phylum) 

c <- df_year %>% group_by(class) %>% 
  filter(class == "MAMMALIA" | class == 'AVES' | class == 'AMPHIBIA' | class == 'REPTILIA' | class == 'INSECTA') %>%
  summarize(early = sum(year <= 1900), late = sum(year > 1900)) %>% 
  rename("taxa" = class) 

early_late <- bind_rows(k, p, c)
early_late
```

    # A tibble: 7 x 3
      taxa     early  late
      <chr>    <int> <int>
    1 PLANTAE      8    32
    2 CHORDATA   121   120
    3 AMPHIBIA     0    10
    4 AVES        85    44
    5 INSECTA      1     9
    6 MAMMALIA    25    29
    7 REPTILIA     8     4

> Relative to Table 1 of [Ceballos et al
> (2015)](http://doi.org/10.1126/sciadv.1400253), we found fewer
> extinctions per taxa. This is probably due to our method of extracting
> the year, since we only look for some matching year in “rationale” and
> there is no documentation specifying that this is the year they went
> extinct. However, our numbers are similar proportionally.

## Weighing by number of species

In order to have a meaningful comparison of extinction rates, we must
weight our number of species going extinct at each time point by the
total number of species in each taxonomic group. That is, we will
compute the number of extinctions per million species per year (MSY;
equivalently, the number extinctions per 10,000 species per 100 years).

First, we compute how many species are present in each of the taxonomic
groups by creating a table that includes all assessed species by quering
the IUCN API again.

``` r
#Gets the count of all the species
getCount <- function() {
  resp <- GET("http://apiv3.iucnredlist.org/api/v3/speciescount?token=9bb4facb6d23f48efbf424bb05c0c1ef1cf6f468393bc745d42179ac4aca5fee")
  status <- http_error(resp)
  if (!status) {
     out <- content(resp, as = "text")
    df <- jsonlite::fromJSON(out)
    return(as.numeric(df$count))
  }
 
}
speciesCount <- getCount()
 
#Creates a list of API calls to species information 
api_species <- map(seq(0, ceiling(speciesCount/10000)-1), function(i)
  GET(paste0( "http://apiv3.iucnredlist.org/api/v3/species/page/", as.character(i),"?token=9bb4facb6d23f48efbf424bb05c0c1ef1cf6f468393bc745d42179ac4aca5fee"))
  )
spec <- map(api_species, content, as = 'text') %>%
  map_dfr(function(resp) {
    jsonlite::fromJSON(resp)[["result"]]
    })

#create a summary table by taxa, counting the number of species to get a species_count
k2 <- spec %>%
  filter(kingdom_name == "PLANTAE") %>% 
  group_by(kingdom_name) %>%
  summarize(species_count = n()) %>%
  rename("taxa" = kingdom_name) 

p2 <- spec %>% 
  filter(phylum_name == "CHORDATA") %>% 
  group_by(phylum_name) %>% 
  summarize(species_count = n()) %>%
  rename("taxa" = phylum_name) 

c2 <- spec %>%
  filter(class_name == "MAMMALIA" | class_name == 'AVES' | 
           class_name == 'AMPHIBIA' | class_name == 'REPTILIA' | 
           class_name == 'INSECTA') %>%
  group_by(class_name) %>%
  summarize(species_count = n()) %>%
  rename("taxa" = class_name) 


all <- bind_rows(k2, p2, c2)

#calculate million species-year for each taxa 
msy <- all %>% right_join(early_late) %>%
  mutate(extinct = early + late) %>% 
  mutate(MSY = extinct/species_count*100) %>%
  select(taxa, MSY)

msy
```

    # A tibble: 7 x 2
      taxa       MSY
      <chr>    <dbl>
    1 PLANTAE  0.149
    2 CHORDATA 0.510
    3 AMPHIBIA 0.150
    4 AVES     1.16 
    5 INSECTA  0.126
    6 MAMMALIA 0.884
    7 REPTILIA 0.178

Our MSY is lower than the proposed baseline of 2 MSY. Howver, this is
most likely due to issues with our algorithm in how we extract dates
from the rationale. We are undercounting the number of extinct species
severely using this method.

## Improving our algorithm

Our method of extracting date from rationale is flawed because we just
pull the first four digit number in the rationale. However, without the
context of the sentence this could be the incorrect year. Oftentimes the
first year listed is the last time it was seen in the wild, not the date
declared extinct. We also encountered missing values because the data
was not meant to be parsed in this way, ie to extract extinction year
from a text block.
