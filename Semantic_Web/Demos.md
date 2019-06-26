Semantic Web Demos
================
Mohammed Ali
6/9/2019

## RDF Document Building

<details>

<summary>Building RDF Document </summary>

``` r
rdf <- rdf()
base <- "http://animalshop.com/animals#"
rdf %>% 
  rdf_add(subject = paste0(base, "animal1"),
          predicate = paste0(base, "type"),
          object = "Dog") %>%
  rdf_add(subject = paste0(base, "animal1"),
          predicate = paste0(base, "name"),
          object = paste0(base, "Bengie")) %>% 
  rdf_add(subject = paste0(base, "animal1"),
          predicate = paste0(base, "friend"),
          object = paste0(base,"Bonnie")) %>%
  rdf_add(subject = paste0(base, "animal2"),
          predicate = paste0(base, "type"),
          object = "Cat") %>% 
  rdf_add(subject = paste0(base, "animal2"),
          predicate = paste0(base, "name"),
          object = paste0(base,"Bonnie")) %>% 
  rdf_add(subject = paste0(base, "animal2"),
          predicate = paste0(base, "friend"),
          object = paste0(base,"Bengie"))
```

</details>

<details>

<summary>Turtle Format </summary>

``` r
options(rdf_print_format = "turtle", rdf_max_print = 30)
rdf
```

    ## Total of 6 triples, stored in hashes
    ## -------------------------------
    ## @base <localhost://> .
    ## @prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    ## 
    ## <http://animalshop.com/animals#animal1>
    ##     <http://animalshop.com/animals#friend> <http://animalshop.com/animals#Bonnie> ;
    ##     <http://animalshop.com/animals#name> <http://animalshop.com/animals#Bengie> ;
    ##     <http://animalshop.com/animals#type> "Dog" .
    ## 
    ## <http://animalshop.com/animals#animal2>
    ##     <http://animalshop.com/animals#friend> <http://animalshop.com/animals#Bengie> ;
    ##     <http://animalshop.com/animals#name> <http://animalshop.com/animals#Bonnie> ;
    ##     <http://animalshop.com/animals#type> "Cat" .

</details>

<details>

<summary>RDF/XML Format</summary>

``` r
options(rdf_print_format = "rdfxml", rdf_max_print = 30)
rdf
```

    ## Total of 6 triples, stored in hashes
    ## -------------------------------
    ## <?xml version="1.0" encoding="utf-8"?>
    ## <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#" xml:base="localhost://">
    ##   <rdf:Description rdf:about="http://animalshop.com/animals#animal2">
    ##     <ns0:name xmlns:ns0="http://animalshop.com/animals#" rdf:resource="http://animalshop.com/animals#Bonnie"/>
    ##   </rdf:Description>
    ##   <rdf:Description rdf:about="http://animalshop.com/animals#animal1">
    ##     <ns0:type xmlns:ns0="http://animalshop.com/animals#">Dog</ns0:type>
    ##   </rdf:Description>
    ##   <rdf:Description rdf:about="http://animalshop.com/animals#animal2">
    ##     <ns0:friend xmlns:ns0="http://animalshop.com/animals#" rdf:resource="http://animalshop.com/animals#Bengie"/>
    ##   </rdf:Description>
    ##   <rdf:Description rdf:about="http://animalshop.com/animals#animal1">
    ##     <ns0:friend xmlns:ns0="http://animalshop.com/animals#" rdf:resource="http://animalshop.com/animals#Bonnie"/>
    ##   </rdf:Description>
    ##   <rdf:Description rdf:about="http://animalshop.com/animals#animal1">
    ##     <ns0:name xmlns:ns0="http://animalshop.com/animals#" rdf:resource="http://animalshop.com/animals#Bengie"/>
    ##   </rdf:Description>
    ##   <rdf:Description rdf:about="http://animalshop.com/animals#animal2">
    ##     <ns0:type xmlns:ns0="http://animalshop.com/animals#">Cat</ns0:type>
    ##   </rdf:Description>
    ## </rdf:RDF>

![Caption for the picture.](servlet_3600628766471272067.png)

</details>

## OWL

<details>

<summary>Basic Example</summary>

``` r
# <rdf:RDF
#   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
#   xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#"
#   xmlns:owl="http://www.w3.org/2002/07/owl#"
#   xmlns:dc="http://purl.org/dc/elements/1.1/">
# 
#   <!-- OWL Header Example -->
#   <owl:Ontology rdf:about="http://www.linkeddatatools.com/plants">
#       <dc:title>The LinkedDataTools.com Example Plant Ontology</dc:title>
#       <dc:description>An example ontology written for the LinkedDataTools.com RDFS & OWL introduction tutorial</dc:description>
#   </owl:Ontology>
# 
#   <!-- OWL Class Definition Example -->
#   <owl:Class rdf:about="http://www.linkeddatatools.com/plants#planttype">
#       <rdfs:label>The plant type</rdfs:label>
#       <rdfs:comment>The class of plant types.</rdfs:comment>
#   </owl:Class>
# 
# </rdf:RDF>
```

</details>

<details>

<summary>OWL Classes, Subclasses & Instance</summary>

``` r
# <rdf:RDF
#   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
#   xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#"
#   xmlns:owl="http://www.w3.org/2002/07/owl#"
#   xmlns:dc="http://purl.org/dc/elements/1.1/"
#   xmlns:plants="http://www.linkeddatatools.com/plants#">
# 
#   <!-- OWL Header Omitted For Brevity -->
# 
#   <!-- OWL Class Definition - Plant Type -->
#   <owl:Class rdf:about="http://www.linkeddatatools.com/plants#planttype">
# 
#       <rdfs:label>The plant type</rdfs:label>
#       <rdfs:comment>The class of all plant types.</rdfs:comment>
# 
#   </owl:Class>
# 
#   <!-- OWL Subclass Definition - Flower -->
#   <owl:Class rdf:about="http://www.linkeddatatools.com/plants#flowers">
# 
#       <!-- Flowers is a subclassification of planttype -->
#       <rdfs:subClassOf rdf:resource="http://www.linkeddatatools.com/plants#planttype"/>
# 
#       <rdfs:label>Flowering plants</rdfs:label>
#       <rdfs:comment>Flowering plants, also known as angiosperms.</rdfs:comment>
# 
#   </owl:Class>
# 
#   <!-- OWL Subclass Definition - Shrub -->
#   <owl:Class rdf:about="http://www.linkeddatatools.com/plants#shrubs">
# 
#       <!-- Shrubs is a subclassification of planttype -->
#       <rdfs:subClassOf rdf:resource="http://www.linkeddatatools.com/plants#planttype"/>
# 
#       <rdfs:label>Shrubbery</rdfs:label>
#       <rdfs:comment>Shrubs, a type of plant which branches from the base.</rdfs:comment>
# 
#   </owl:Class>
# 
#   <!-- Individual (Instance) Example RDF Statement -->
#   <rdf:Description rdf:about="http://www.linkeddatatools.com/plants#magnolia">
# 
#       <!-- Magnolia is a type (instance) of the flowers classification -->
#       <rdf:type rdf:resource="http://www.linkeddatatools.com/plants#flowers"/>
# 
#   </rdf:Description>
# 
# </rdf:RDF>
```

</deails>

## SPARQL

<details>

<summary>Select Everything</summary>

``` r
query <-   'SELECT  *
WHERE {
 ?sub ?pred ?obj
}'
rdf_query(rdf, query)
## # A tibble: 6 x 3
##   sub                       pred                    obj                    
##   <chr>                     <chr>                   <chr>                  
## 1 http://animalshop.com/an~ http://animalshop.com/~ http://animalshop.com/~
## 2 http://animalshop.com/an~ http://animalshop.com/~ Dog                    
## 3 http://animalshop.com/an~ http://animalshop.com/~ http://animalshop.com/~
## 4 http://animalshop.com/an~ http://animalshop.com/~ http://animalshop.com/~
## 5 http://animalshop.com/an~ http://animalshop.com/~ http://animalshop.com/~
## 6 http://animalshop.com/an~ http://animalshop.com/~ Cat
```

</details>

<details>

<summary>Select Bonnie Data</summary>

``` r
query <-   'SELECT  *
WHERE {
 ?sub ?pred <http://animalshop.com/animals#Bonnie>
}'
rdf_query(rdf, query)
## # A tibble: 2 x 2
##   sub                                   pred                               
##   <chr>                                 <chr>                              
## 1 http://animalshop.com/animals#animal2 http://animalshop.com/animals#name 
## 2 http://animalshop.com/animals#animal1 http://animalshop.com/animals#frie~
```

</details>

<details>

<summary>Select Animal Types</summary>

``` r
query <- 'SELECT  *
WHERE {
 ?sub <http://animalshop.com/animals#type> ?obj
}'
rdf_query(rdf, query)
## # A tibble: 2 x 2
##   sub                                   obj  
##   <chr>                                 <chr>
## 1 http://animalshop.com/animals#animal1 Dog  
## 2 http://animalshop.com/animals#animal2 Cat
```

</details>
