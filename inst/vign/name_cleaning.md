<!--
%\VignetteEngine{knitr::knitr}
%\VignetteIndexEntry{Strategies for programmatic name cleaning}
%\VignetteEncoding{UTF-8}
-->



Strategies for programmatic name cleaning
=========================================

`taxize` offers interactive prompts when using `get_*()` functions (e.g., `get_tsn()`).
These prompts make it easy in interactive use to select choices when there are more
than one match found.

However, to make your code reproducible you don't want interactive prompts.

This vignette covers some options for programmatic name cleaning.


```r
library("taxize")
```

## get_* functions

When using `get_*()` functions programatically, you have a few options.

### rows parameter

Normally, if you get more than one result, you get a prompt asking you
to select which taxon you want.


```r
get_tsn(searchterm = "Quercus b")
#>       tsn                       target             commonnames    nameusage
#> 1   19298             Quercus beebiana                         not accepted
#> 2  507263       Quercus berberidifolia               scrub oak     accepted
#> 3   19300              Quercus bicolor         swamp white oak     accepted
#> 4   19303             Quercus borealis                         not accepted
#> 5  195131 Quercus borealis var. maxima                         not accepted
#> 6  195166            Quercus boyntonii Boynton's sand post oak     accepted
#> 7  506533              Quercus brantii             Brant's oak     accepted
#> 8  195150            Quercus breviloba                         not accepted
#> 9  195099              Quercus breweri                         not accepted
#> 10 195168             Quercus buckleyi               Texas oak     accepted
#>
#> More than one TSN found for taxon 'Quercus b'!
#>
#>             Enter rownumber of taxon (other inputs will return 'NA'):
#>
#> 1:
```

Instead, we can use the rows parameter to specify which records we want
by number only (not by a name itself). Here, we want the first 3 records:


```r
get_tsn(searchterm = 'Quercus b', rows = 1:3)
#>     tsn           target     commonnames    nameusage
#> 1 19298 Quercus beebiana                 not accepted
#> 2 19300  Quercus bicolor swamp white oak     accepted
#> 3 19303 Quercus borealis                 not accepted
#>
#> More than one TSN found for taxon 'Quercus b'!
#>
#>             Enter rownumber of taxon (other inputs will return 'NA'):
#>
#> 1:
```

However, you still get a prompt as there is more than one result.

Thus, for full programmatic usage, you can specify a single row, if you happen
to know which one you want:


```r
get_tsn(searchterm = 'Quercus b', rows = 3)
#> [1] "19303"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] TRUE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=19303"
#> attr(,"class")
#> [1] "tsn"
```

In reality it is unlikely you'll know which row you want, unless perhaps you
just want one result from each query, regardless of what it is.

### underscore methods

A better fit for programmatic use are underscore methods. Each `get_*()` function
has a sister method with and trailing underscore, e.g., `get_tsn()` and `get_tsn_()`.


```r
get_tsn_(searchterm = "Quercus b")
#> $`Quercus b`
#> # A tibble: 5 x 4
#>   tsn    scientificName        commonNames                        nameUsage
#>   <chr>  <chr>                 <chr>                              <chr>    
#> 1 19300  Quercus bicolor       swamp white oak,chêne bicolore     accepted 
#> 2 195166 Quercus boyntonii     Boynton's sand post oak,Boynton's… accepted 
#> 3 195168 Quercus buckleyi      Texas oak,Buckley's oak            accepted 
#> 4 506533 Quercus brantii       Brant's oak                        accepted 
#> 5 507263 Quercus berberidifol… scrub oak                          accepted
```

The result is a single data.frame for each taxon queried, which can be
processed downstream with whatever logic is required in your workflow.

You can also combine `rows` parameter with underscore functions, as a single
number of a range of numbers:


```r
get_tsn_(searchterm = "Quercus b", rows = 1)
#> $`Quercus b`
#> # A tibble: 1 x 4
#>   tsn   scientificName  commonNames                    nameUsage
#>   <chr> <chr>           <chr>                          <chr>    
#> 1 19300 Quercus bicolor swamp white oak,chêne bicolore accepted
```


```r
get_tsn_(searchterm = "Quercus b", rows = 1:2)
#> $`Quercus b`
#> # A tibble: 2 x 4
#>   tsn    scientificName    commonNames                           nameUsage
#>   <chr>  <chr>             <chr>                                 <chr>    
#> 1 19300  Quercus bicolor   swamp white oak,chêne bicolore        accepted 
#> 2 195166 Quercus boyntonii Boynton's sand post oak,Boynton's oak accepted
```

## as.* methods

All `get_*()` functions have associated `as.*()` functions (e.g., `get_tsn()` and `as.tsn()`).

Many `taxize` functions use taxonomic identifier classes (S3 objects) that are the output
of `get_*()` functions. `as.*()` methods make it easy to make the required S3 taxonomic
identifier classes if you already know the identifier. For example:

Already a tsn, returns the same


```r
as.tsn(get_tsn("Quercus douglasii"))
#> [1] "19322"
#> attr(,"match")
#> [1] "found"
#> attr(,"multiple_matches")
#> [1] FALSE
#> attr(,"pattern_match")
#> [1] FALSE
#> attr(,"uri")
#> [1] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=19322"
#> attr(,"class")
#> [1] "tsn"
```

numeric


```r
as.tsn(c(19322, 129313, 506198))
#> [1] "19322"  "129313" "506198"
#> attr(,"class")
#> [1] "tsn"
#> attr(,"match")
#> [1] "found" "found" "found"
#> attr(,"multiple_matches")
#> [1] FALSE FALSE FALSE
#> attr(,"pattern_match")
#> [1] FALSE FALSE FALSE
#> attr(,"uri")
#> [1] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=19322" 
#> [2] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=129313"
#> [3] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=506198"
```

And you can do the same for character, or list inputs - depending on the data source.

The above `as.tsn()` examples have the parameter `check = TRUE`, meaning we ping the 
data source web service to make sure the identifier exists. You can skip that check 
if you like by setting `check = FALSE`, and the result is returned much faster:


```r
as.tsn(c("19322","129313","506198"), check = FALSE)
#> [1] "19322"  "129313" "506198"
#> attr(,"class")
#> [1] "tsn"
#> attr(,"match")
#> [1] "found" "found" "found"
#> attr(,"multiple_matches")
#> [1] FALSE FALSE FALSE
#> attr(,"pattern_match")
#> [1] FALSE FALSE FALSE
#> attr(,"uri")
#> [1] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=19322" 
#> [2] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=129313"
#> [3] "https://www.itis.gov/servlet/SingleRpt/SingleRpt?search_topic=TSN&search_value=506198"
```

With the output of `as.*()` methods, you can then proceed with other `taxize` functions.

## gnr_resolve

Some functions in `taxize` are meant specifically for name cleaning. One of those
is `gnr_resolve()`.

`gnr_resolve()` doesn't provide prompts as do `get_*()` functions, but instead
return data.frame's. So we don't face the same problem, and can use `gnr_resolve()`
in a programmatic workflow straight away.


```r
spp <- names_list(rank = "species", size = 10)
gnr_resolve(names = spp, preferred_data_sources = 11)
#> # A tibble: 12 x 5
#>    user_supplied_na… submitted_name  matched_name    data_source_tit… score
#>  * <chr>             <chr>           <chr>           <chr>            <dbl>
#>  1 Cassine transvaa… Cassine transv… Cassine transv… GBIF Backbone T… 0.988
#>  2 Plantago annua    Plantago annua  Plantago annua… GBIF Backbone T… 0.988
#>  3 Ribes tularensis  Ribes tularens… Ribes tularens… GBIF Backbone T… 0.988
#>  4 Piper verruclifo… Piper verrucli… Piper verrucli… GBIF Backbone T… 0.988
#>  5 Inula stricta     Inula stricta   Inula stricta … GBIF Backbone T… 0.988
#>  6 Carex austromexi… Carex austrome… Carex austrome… GBIF Backbone T… 0.988
#>  7 Schmidelia macro… Schmidelia mac… Schmidelia mac… GBIF Backbone T… 0.988
#>  8 Schmidelia macro… Schmidelia mac… Schmidelia mac… GBIF Backbone T… 0.988
#>  9 Schmidelia macro… Schmidelia mac… Schmidelia mac… GBIF Backbone T… 0.988
#> 10 Anila sessilifol… Anila sessilif… Anila sessilif… GBIF Backbone T… 0.988
#> 11 Glochidion muell… Glochidion mue… Glochidion mue… GBIF Backbone T… 0.988
#> 12 Viguiera viridis  Viguiera virid… Viguiera virid… GBIF Backbone T… 0.988
```

## Other functions

Some other functions in `taxize` use `get_*()` functions internally (e.g., `classification()`),
but you can can generally pass on parameters to the `get_*()` functions internally.


## Feedback?

Let us know if you have ideas for better ways to do programmatic name cleaning at 
<https://github.com/ropensci/taxize/issues> or <https://discuss.ropensci.org/> !
