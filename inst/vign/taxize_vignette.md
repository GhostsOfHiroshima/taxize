<!--
%\VignetteEngine{knitr::knitr}
%\VignetteIndexEntry{taxize vignette}
%\VignetteEncoding{UTF-8}
-->



taxize vignette - a taxonomic toolbelt for R
======

`taxize` is a taxonomic toolbelt for R. `taxize` wraps APIs for a large suite of taxonomic databases availab on the web.

## Installation

First, install and load `taxize` into the R session.


```r
install.packages("taxize")
```


```r
library("taxize")
```

Advanced users can also download and install the latest development copy from [GitHub](https://github.com/ropensci/taxize_).

## Resolve taxonomic name

This is a common task in biology. We often have a list of species names and we want to know a) if we have the most up to date names, b) if our names are spelled correctly, and c) the scientific name for a common name. One way to resolve names is via the Global Names Resolver (GNR) service provided by the [Encyclopedia of Life][eol]. Here, we are searching for two misspelled names:


```r
temp <- gnr_resolve(names = c("Helianthos annus", "Homo saapiens"))
head(temp)
```

```
#> # A tibble: 6 x 5
#>   user_supplied_na… submitted_name  matched_name     data_source_tit… score
#>   <chr>             <chr>           <chr>            <chr>            <dbl>
#> 1 Helianthos annus  Helianthos ann… Helianthus annus EOL               0.75
#> 2 Helianthos annus  Helianthos ann… Helianthus annu… EOL               0.75
#> 3 Helianthos annus  Helianthos ann… Helianthus annus uBio NameBank     0.75
#> 4 Helianthos annus  Helianthos ann… Helianthus annu… Catalogue of Li…  0.75
#> 5 Helianthos annus  Helianthos ann… Helianthus annu… ITIS              0.75
#> 6 Helianthos annus  Helianthos ann… Helianthus annu… NCBI              0.75
```

The correct spellings are *Helianthus annuus* and *Homo sapiens*. Another approach uses the Taxonomic Name Resolution Service via the Taxosaurus API developed by iPLant and the Phylotastic organization. In this example, we provide a list of species names, some of which are misspelled, and we'll call the API with the *tnrs* function.


```r
mynames <- c("Helianthus annuus", "Pinus contort", "Poa anua", "Abis magnifica",
  	"Rosa california", "Festuca arundinace", "Sorbus occidentalos","Madia sateva")
tnrs(query = mynames, source = "iPlant_TNRS")[ , -c(5:7)]
```

```
#>         submittedname        acceptedname    sourceid score
#> 1 Sorbus occidentalos Sorbus occidentalis iPlant_TNRS  0.99
#> 2  Festuca arundinace Festuca arundinacea iPlant_TNRS  0.99
#> 3      Abis magnifica     Abies magnifica iPlant_TNRS  0.96
#> 4       Pinus contort      Pinus contorta iPlant_TNRS  0.98
#> 5            Poa anua           Poa annua iPlant_TNRS  0.96
#> 6        Madia sateva        Madia sativa iPlant_TNRS  0.97
#> 7   Helianthus annuus   Helianthus annuus iPlant_TNRS     1
#> 8     Rosa california    Rosa californica iPlant_TNRS  0.99
```

It turns out there are a few corrections: e.g., *Madia sateva* should be *Madia sativa*, and *Rosa california* should be *Rosa californica*. Note that this search worked because fuzzy matching was employed to retrieve names that were close, but not exact matches. Fuzzy matching is only available for plants in the TNRS service, so we advise using EOL's Global Names Resolver if you need to resolve animal names.

taxize takes the approach that the user should be able to make decisions about what resource to trust, rather than making the decision. Both the EOL GNR and the TNRS services provide data from a variety of data sources. The user may trust a specific data source, thus may want to use the names from that data source. In the future, we may provide the ability for taxize to suggest the best match from a variety of sources.

Another common use case is when there are many synonyms for a species. In this example, we have three synonyms of the currently accepted name for a species.


```r
mynames <- c("Helianthus annuus ssp. jaegeri", "Helianthus annuus ssp. lenticularis", "Helianthus annuus ssp. texanus")
(tsn <- get_tsn(mynames, accepted = FALSE))
```

```
[1] "525928" "525929" "525930"
attr(,"match")
[1] "found" "found" "found"
attr(,"multiple_matches")
[1] FALSE FALSE FALSE
attr(,"pattern_match")
[1] FALSE FALSE FALSE
attr(,"uri")
[1] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=525928"
[2] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=525929"
[3] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=525930"
attr(,"class")
[1] "tsn"
```

```r
lapply(tsn, itis_acceptname)
```

```
[[1]]
  submittedtsn      acceptedname acceptedtsn author
1       525928 Helianthus annuus       36616     L.

[[2]]
  submittedtsn      acceptedname acceptedtsn author
1       525929 Helianthus annuus       36616     L.

[[3]]
  submittedtsn      acceptedname acceptedtsn author
1       525930 Helianthus annuus       36616     L.
```

## Retrieve higher taxonomic names

Another task biologists often face is getting higher taxonomic names for a taxa list. Having the higher taxonomy allows you to put into context the relationships of your species list. For example, you may find out that species A and species B are in Family C, which may lead to some interesting insight, as opposed to not knowing that Species A and B are closely related. This also makes it easy to aggregate/standardize data to a specific taxonomic level (e.g., family level) or to match data to other databases with different taxonomic resolution (e.g., trait databases).

A number of data sources in taxize provide the capability to retrieve higher taxonomic names, but we will highlight two of the more useful ones: [Integrated Taxonomic Information System (ITIS)][itis] and [National Center for Biotechnology Information (NCBI)][ncbi]. First, we'll search for two species, *Abies procera} and *Pinus contorta* within ITIS.


```r
specieslist <- c("Abies procera","Pinus contorta")
classification(specieslist, db = 'itis')
```

```
#> $`Abies procera`
#>               name          rank     id
#> 1          Plantae       kingdom 202422
#> 2    Viridiplantae    subkingdom 954898
#> 3     Streptophyta  infrakingdom 846494
#> 4      Embryophyta superdivision 954900
#> 5     Tracheophyta      division 846496
#> 6  Spermatophytina   subdivision 846504
#> 7        Pinopsida         class 500009
#> 8          Pinidae      subclass 954916
#> 9          Pinales         order 500028
#> 10        Pinaceae        family  18030
#> 11           Abies         genus  18031
#> 12   Abies procera       species 181835
#> 
#> $`Pinus contorta`
#>               name          rank     id
#> 1          Plantae       kingdom 202422
#> 2    Viridiplantae    subkingdom 954898
#> 3     Streptophyta  infrakingdom 846494
#> 4      Embryophyta superdivision 954900
#> 5     Tracheophyta      division 846496
#> 6  Spermatophytina   subdivision 846504
#> 7        Pinopsida         class 500009
#> 8          Pinidae      subclass 954916
#> 9          Pinales         order 500028
#> 10        Pinaceae        family  18030
#> 11           Pinus         genus  18035
#> 12  Pinus contorta       species 183327
#> 
#> attr(,"class")
#> [1] "classification"
#> attr(,"db")
#> [1] "itis"
```

It turns out both species are in the family Pinaceae. You can also get this type of information from the NCBI by doing `classification(specieslist, db = 'ncbi')`.

Instead of a full classification, you may only want a single name, say a family name for your species of interest. The function *tax_name} is built just for this purpose. As with the `classification` function you can specify the data source with the `db` argument, either ITIS or NCBI.


```r
tax_name(query = "Helianthus annuus", get = "family", db = "ncbi")
```

```
#>     db             query     family
#> 1 ncbi Helianthus annuus Asteraceae
```

I may happen that a data source does not provide information on the queried species, than one could take the result from another source and union the results from the different sources.

## Interactive name selection

As mentioned most databases use a numeric code to reference a species. A general workflow in taxize is: Retrieve Code for the queried species and then use this code to query more data/information.

Below are a few examples. When you run these examples in R, you are presented with a command prompt asking for the row that contains the name you would like back; that output is not printed below for brevity. In this example, the search term has many matches. The function returns a data frame of the matches, and asks for the user to input what row number to accept.


```r
get_uid(sciname = "Pinus")
```

```
#>   status     rank    division scientificname commonname    uid genus
#> 1 active subgenus seed plants          Pinus hard pines 139271      
#> 2 active    genus seed plants          Pinus              3337      
#>   species subsp modificationdate
#> 1               2017/06/14 00:00
#> 2               2016/03/25 00:00
```

```
#> [1] "139271"
#> attr(,"class")
#> [1] "uid"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] TRUE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "https://www.ncbi.nlm.nih.gov/taxonomy/139271"
```

In another example, you can pass in a long character vector of taxonomic names (although this one is rather short for demo purposes):


```r
splist <- c("annona cherimola", 'annona muricata', "quercus robur")
get_tsn(searchterm = splist, searchtype = "scientific")
```

```
#> [1] "506198" "18098"  "19405" 
#> attr(,"match")
#> [1] "found" "found" "found"
#> attr(,"multiple_matches")
#> [1] FALSE FALSE  TRUE
#> attr(,"pattern_match")
#> [1] FALSE FALSE  TRUE
#> attr(,"uri")
#> [1] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=506198"
#> [2] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=18098" 
#> [3] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=19405" 
#> attr(,"class")
#> [1] "tsn"
```

There are functions for many other sources

* `get_boldid()`
* `get_colid()`
* `get_eolid()`
* `get_gbifid()`
* `get_nbnid()`
* `get_tpsid()`

Sometimes with these functions you get a lot of data back. In these cases you may want to limit your choices. Soon we will incorporate the ability to filter using `regex` to limit matches, but for now, we have a new parameter, `rows`, which lets you select certain rows. For example, you can select the first row of each given name, which means there is no interactive component:


```r
get_nbnid(c("Zootoca vivipara","Pinus contorta"), rows = 1)
```

```
#> [1] "NHMSYS0001706186" "NBNSYS0000004786"
#> attr(,"class")
#> [1] "nbnid"
#> attr(,"match")
#> [1] "found" "found"
#> attr(,"multiple_matches")
#> [1] TRUE TRUE
#> attr(,"pattern_match")
#> [1] FALSE FALSE
#> attr(,"uri")
#> [1] "https://species.nbnatlas.org/species/NHMSYS0001706186"
#> [2] "https://species.nbnatlas.org/species/NBNSYS0000004786"
```

Or you can select a range of rows


```r
get_nbnid(c("Zootoca vivipara","Pinus contorta"), rows = 1:3)
```

```
#> [1] "NHMSYS0001706186" "NBNSYS0000004786"
#> attr(,"class")
#> [1] "nbnid"
#> attr(,"match")
#> [1] "found" "found"
#> attr(,"multiple_matches")
#> [1] TRUE TRUE
#> attr(,"pattern_match")
#> [1] TRUE TRUE
#> attr(,"uri")
#> [1] "https://species.nbnatlas.org/species/NHMSYS0001706186"
#> [2] "https://species.nbnatlas.org/species/NBNSYS0000004786"
```

In addition, in case you don't want to do interactive name selection in the case where there are a lot of names, you can get all data back with functions of the form, e.g., `get_tsn_()`, and likewise for other data sources. For example:


```r
out <- get_nbnid_("Poa annua")
NROW(out$`Poa annua`)
```

```
#> [1] 25
```

That's a lot of data, so we can get only certain rows back


```r
get_nbnid_("Poa annua", rows = 1:10)
```

```
#> $`Poa annua`
#>                guid     scientificName    rank taxonomicStatus
#> 1  NBNSYS0000002544          Poa annua species        accepted
#> 2  NBNSYS0200001901       Bellis annua species        accepted
#> 3  NBNSYS0200003392   Triumfetta annua species        accepted
#> 4  NBNSYS0200002555        Lonas annua species        accepted
#> 5  NHMSYS0000456951  Carrichtera annua species        accepted
#> 6  NHMSYS0000461807 Poa labillardierei species        accepted
#> 7  NHMSYS0000461808      Poa ligularis species        accepted
#> 8  NHMSYS0000461817     Poa sieberiana species        accepted
#> 9  NHMSYS0000461805         Poa gunnii species        accepted
#> 10 NHMSYS0000461801     Poa costiniana species        accepted
```

## Coerce numerics/alphanumerics to taxon IDs

We've also introduced in `v0.5` the ability to coerce numerics and alphanumerics to taxonomic ID classes that are usually only retrieved via `get_*()` functions.

For example, adfafd


```r
as.gbifid(get_gbifid("Poa annua")) # already a uid, returns the same
```

```
#>    gbifid             scientificname    rank   status matchtype
#> 1 2704179               Poa annua L. species ACCEPTED     EXACT
#> 2 8422205 Poa annua Cham. & Schltdl. species  SYNONYM     EXACT
#> 3 7730008           Poa annua Steud. species DOUBTFUL     EXACT
```

```
#> [1] "2704179"
#> attr(,"class")
#> [1] "gbifid"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] TRUE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "http://www.gbif.org/species/2704179"
```

```r
as.gbifid(2704179) # numeric
```

```
#> [1] "2704179"
#> attr(,"class")
#> [1] "gbifid"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] FALSE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "http://www.gbif.org/species/2704179"
```

```r
as.gbifid("2704179") # character
```

```
#> [1] "2704179"
#> attr(,"class")
#> [1] "gbifid"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] FALSE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "http://www.gbif.org/species/2704179"
```

```r
as.gbifid(list("2704179","2435099","3171445")) # list, either numeric or character
```

```
#> [1] "2704179" "2435099" "3171445"
#> attr(,"class")
#> [1] "gbifid"
#> attr(,"match")
#> [1] "found" "found" "found"
#> attr(,"multiple_matches")
#> [1] FALSE FALSE FALSE
#> attr(,"pattern_match")
#> [1] FALSE FALSE FALSE
#> attr(,"uri")
#> [1] "http://www.gbif.org/species/2704179"
#> [2] "http://www.gbif.org/species/2435099"
#> [3] "http://www.gbif.org/species/3171445"
```

These `as.*()` functions do a quick check of the web resource to make sure it's a real ID. However, you can turn this check off, making this coercion much faster:


```r
system.time( replicate(3, as.gbifid(c("2704179","2435099","3171445"), check=TRUE)) )
```

```
#>    user  system elapsed 
#>   0.072   0.003   1.760
```

```r
system.time( replicate(3, as.gbifid(c("2704179","2435099","3171445"), check=FALSE)) )
```

```
#>    user  system elapsed 
#>   0.001   0.000   0.002
```

## What taxa are downstream of my taxon of interest?

If someone is not a taxonomic specialist on a particular taxon he likely does not know what children taxa are within a family, or within a genus. This task becomes especially unwieldy when there are a large number of taxa downstream. You can of course go to a website like [Wikispecies][wikispecies] or [Encyclopedia of Life][eol] to get downstream names. However, taxize provides an easy way to programatically search for downstream taxa, both for the [Catalogue of Life (CoL)][col] and the [Integrated Taxonomic Information System][itis]. Here is a short example using the CoL in which we want to find all the species within the genus *Apis* (honey bees).


```r
apis_col_id <- "015be25f6b061ba517f495394b80f108" # id for Apis, fetched beforehand to save time here
downstream(apis_col_id, downto = "species", db = "col")
```

```
#> $`015be25f6b061ba517f495394b80f108`
#>                       childtaxa_id     childtaxa_name childtaxa_rank
#> 1 7a4a38c5095963949d6d6ec917d471de Apis andreniformis        species
#> 2 39610a4ceff7e5244e334a3fbc5e47e5        Apis cerana        species
#> 3 e1d4cbf3872c6c310b7a1c17ddd00ebc       Apis dorsata        species
#> 4 92dca82a063fedd1da94b3f3972d7b22        Apis florea        species
#> 5 4bbc06b9dfbde0b72c619810b564c6e6 Apis koschevnikovi        species
#> 6 67cbbcf92cd60748759e58e802d98518     Apis mellifera        species
#> 7 213668a26ba6d2aad9575218f10d422f   Apis nigrocincta        species
#>   childtaxa_extinct
#> 1             FALSE
#> 2             FALSE
#> 3             FALSE
#> 4             FALSE
#> 5             FALSE
#> 6             FALSE
#> 7             FALSE
#> 
#> attr(,"class")
#> [1] "downstream"
#> attr(,"db")
#> [1] "col"
```

We can also request data from ITIS


```r
apis_itis_id <- 154395 # id for Apis, fetched beforehand to save time here
downstream(apis_itis_id, downto = "species", db = "itis")
```

```
#> $`154395`
#>      tsn parentname parenttsn          taxonname rankid rankname
#> 1 154396       Apis    154395     Apis mellifera    220  species
#> 2 763550       Apis    154395 Apis andreniformis    220  species
#> 3 763551       Apis    154395        Apis cerana    220  species
#> 4 763552       Apis    154395       Apis dorsata    220  species
#> 5 763553       Apis    154395        Apis florea    220  species
#> 6 763554       Apis    154395 Apis koschevnikovi    220  species
#> 7 763555       Apis    154395   Apis nigrocincta    220  species
#> 
#> attr(,"class")
#> [1] "downstream"
#> attr(,"db")
#> [1] "itis"
```

## Direct children

You may sometimes only want the direct children. We got you covered on that front, with methods for ITIS, NCBI, and Catalogue of Life. For example, let's get direct children (species in this case) of the bee genus _Apis_ using COL data:


```r
# using the id for Apis we defined above for COL
children(apis_col_id, db = "col")
```

```
#> $`015be25f6b061ba517f495394b80f108`
#>                       childtaxa_id     childtaxa_name childtaxa_rank
#> 1 7a4a38c5095963949d6d6ec917d471de Apis andreniformis        species
#> 2 39610a4ceff7e5244e334a3fbc5e47e5        Apis cerana        species
#> 3 e1d4cbf3872c6c310b7a1c17ddd00ebc       Apis dorsata        species
#> 4 92dca82a063fedd1da94b3f3972d7b22        Apis florea        species
#> 5 4bbc06b9dfbde0b72c619810b564c6e6 Apis koschevnikovi        species
#> 6 67cbbcf92cd60748759e58e802d98518     Apis mellifera        species
#> 7 213668a26ba6d2aad9575218f10d422f   Apis nigrocincta        species
#>   childtaxa_extinct
#> 1             FALSE
#> 2             FALSE
#> 3             FALSE
#> 4             FALSE
#> 5             FALSE
#> 6             FALSE
#> 7             FALSE
#> 
#> attr(,"class")
#> [1] "children"
#> attr(,"db")
#> [1] "col"
```

The direct children (genera in this case) of _Pinaceae_ using NCBI data:


```r
children("Pinaceae", db = "ncbi")
```

```
#> $Pinaceae
#>    childtaxa_id childtaxa_name childtaxa_rank
#> 1        123600     Nothotsuga          genus
#> 2         64685        Cathaya          genus
#> 3          3358          Tsuga          genus
#> 4          3356    Pseudotsuga          genus
#> 5          3354    Pseudolarix          genus
#> 6          3337          Pinus          genus
#> 7          3328          Picea          genus
#> 8          3325          Larix          genus
#> 9          3323     Keteleeria          genus
#> 10         3321         Cedrus          genus
#> 11         3319          Abies          genus
#> 
#> attr(,"class")
#> [1] "children"
#> attr(,"db")
#> [1] "ncbi"
```

## Get NCBI ID from GenBank Ids

With accession numbers


```r
genbank2uid(id = 'AJ748748')
```

```
#> [[1]]
#> [1] "282199"
#> attr(,"class")
#> [1] "uid"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] FALSE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "https://www.ncbi.nlm.nih.gov/taxonomy/282199"
#> attr(,"name")
#> [1] "Nereida ignava 16S rRNA gene, type strain 2SM4T"
```

With gi numbers


```r
genbank2uid(id = 62689767)
```

```
#> [[1]]
#> [1] "282199"
#> attr(,"class")
#> [1] "uid"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] FALSE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "https://www.ncbi.nlm.nih.gov/taxonomy/282199"
#> attr(,"name")
#> [1] "Nereida ignava 16S rRNA gene, type strain 2SM4T"
```

## Matching species tables with different taxonomic resolution

Biologist often need to match different sets of data tied to species. For example, trait-based approaches are a promising tool in ecology. One problem is that abundance data must be matched with trait databases. These two data tables may contain species information on different taxonomic levels and possibly data must be aggregated to a joint taxonomic level, so that the data can be merged. taxize can help in this data-cleaning step, providing a reproducible workflow:

We can use the mentioned `classification`-function to retrieve the taxonomic hierarchy and then search the hierarchies up- and downwards for matches. Here is an example to match a species with names on three different taxonomic levels.


```r
A <- "gammarus roeseli"

B1 <- "gammarus roeseli"
B2 <- "gammarus"
B3 <- "gammaridae"

A_clas <- classification(A, db = 'ncbi')
B1_clas <- classification(B1, db = 'ncbi')
B2_clas <- classification(B2, db = 'ncbi')
B3_clas <- classification(B3, db = 'ncbi')

B1[match(A, B1)]
```

```
#> [1] "gammarus roeseli"
```

```r
A_clas[[1]]$rank[tolower(A_clas[[1]]$name) %in% B2]
```

```
#> [1] "genus"
```

```r
A_clas[[1]]$rank[tolower(A_clas[[1]]$name) %in% B3]
```

```
#> [1] "family"
```

If we find a direct match (here *Gammarus roeseli*), we are lucky. But we can also match Gammaridae with *Gammarus roeseli*, but on a lower taxonomic level. A more comprehensive and realistic example (matching a trait table with an abundance table) is given in the vignette on matching.

[eol]: http://www.eol.org/
[ncbi]: http://www.ncbi.nlm.nih.gov/
[itis]: http://www.itis.gov/
[wikispecies]: http://species.wikimedia.org/wiki/Main_Page
[col]: http://www.catalogueoflife.org/
