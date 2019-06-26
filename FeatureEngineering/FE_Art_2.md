FE\_Art\_2
================
Mohammed Ali
April 10, 2019

Textual columns are those that contain *free text*. What differentiates
these from the categorical columns is that the number of unique values
for the textual columns would be too big. Such columns are typically
messy, and we will have to deal with them on a column-by-column basis as
no single preprocessing procedure would suit them all.

``` r
datatable(head(startups[,textual_col_names], 20))
```

![](FE_Art_2_files/figure-gfm/data_head-1.png)<!-- -->

``` r
# set contents of these columns to lowercase
startups[,textual_col_names] <- map_df(startups[,textual_col_names], tolower)
```

#### Industry of the Company

And now, a quick look at the *industry\_of\_company* column – which
indicates the particular domain that a company is working in – shows us…

ator.

Now, we wish to create a [*document-term
matrix*](https://en.wikipedia.org/wiki/Document-term_matrix) (DTM) from
this column. The DTM is a matrix where the documents (i.e. records) and
terms (e.g. words) are the rows and columns, respectively, and the cells
contain the frequencies of the terms occurring in the documents
(e.g. the cell *DTM(i,j)* would tell how many times term *j* occurred
in document *i*). To generate the DTM, we shall use the
[quanteda](https://cran.r-project.org/web/packages/quanteda/) package.

``` r
# create the corpus object from the industry_of_company column
mycorpus <- corpus(startups$industry_of_company)

# generate the DTM using the formed corpus
dfm_industry <- dfm(mycorpus,               # the corpus to generate the DTM from
                    tolower = FALSE)        # data is already lowercase
```

At this point, let’s just have a look at the DTM before moving on.

``` r
# observe the 'terms' of the DTM
colnames(dfm_industry)
```

    ##  [1] "market"             "research"           "|"                 
    ##  [4] "marketing"          "crowdfunding"       "analytics"         
    ##  [7] "cloud"              "computing"          "software"          
    ## [10] "development"        "mobile"             "enterprise"        
    ## [13] "food"               "&"                  "beverages"         
    ## [16] "hospitality"        "network"            "/"                 
    ## [19] "hosting"            "infrastructure"     "healthcare"        
    ## [22] "pharmaceuticals"    "media"              "finance"           
    ## [25] "music"              "e-commerce"         "gaming"            
    ## [28] "advertising"        "retail"             "security"          
    ## [31] "email"              "human"              "resources"         
    ## [34] "("                  "hr"                 ")"                 
    ## [37] "career"             "job"                "search"            
    ## [40] "publishing"         "education"          "energy"            
    ## [43] "deals"              "entertainment"      "transportation"    
    ## [46] "social"             "networking"         "real"              
    ## [49] "estate"             "telecommunications" "insurance"         
    ## [52] "cleantech"          "space"              "travel"            
    ## [55] "classifieds"        "government"

Unfortunately, the output is unsatisfactory. Some of the terms that are
parsed by the
[quanteda](https://cran.r-project.org/web/packages/quanteda/) package
include `|`, `/`, `&`, `(` and `)` (improper terms, obviously). We also
observe that industries such as `cloud computing` and `software
development` have been broken down into their constituent words, which
is also incorrect.

With some investigation of the
[quanteda](https://cran.r-project.org/web/packages/quanteda/) package
(specifically the `quanteda::tokens()` function), it seems that it is
only able to split the terms over the *spaces* in the text. In other
words, we cannot specify the `|` character as the separator for
splitting the text into terms.

As such, we will deal with this issue by performing the following
actions:

  - **Replace spaces with underscores:** merges each of the multi-word
    industries into a single term.
  - **Replace occurrences of the `|` character with spaces:** helps the
    `quanteda::dfm()` function split the terms over the new spaces.
  - **Remove the special characters, `/`, `&`, `(` and `)`:** makes life
    easier for the `quanteda::dfm()` function.

It may be a good idea to look at the cases on which we intend to perform
the above actions. Let’s have a look.

``` r
# retrieve and view those industries that have '/' in them
startups$industry_of_company[ grepl(x = startups$industry_of_company, pattern = '/', fixed = TRUE) ]
```

    ##  [1] "cloud computing|network / hosting / infrastructure"                                      
    ##  [2] "analytics|security|network / hosting / infrastructure"                                   
    ##  [3] "human resources (hr)|marketing|career / job search"                                      
    ##  [4] "analytics|network / hosting / infrastructure"                                            
    ##  [5] "human resources (hr)|enterprise software|career / job search|social networking|analytics"
    ##  [6] "network / hosting / infrastructure|food & beverages|analytics"                           
    ##  [7] "network / hosting / infrastructure"                                                      
    ##  [8] "media|entertainment|analytics|network / hosting / infrastructure|publishing"             
    ##  [9] "network / hosting / infrastructure|enterprise software|software development|analytics"   
    ## [10] "network / hosting / infrastructure"                                                      
    ## [11] "network / hosting / infrastructure|analytics"                                            
    ## [12] "network / hosting / infrastructure|enterprise software"                                  
    ## [13] "career / job search"                                                                     
    ## [14] "network / hosting / infrastructure|publishing"                                           
    ## [15] "classifieds|network / hosting / infrastructure"                                          
    ## [16] "e-commerce|analytics|network / hosting / infrastructure"                                 
    ## [17] "analytics|social networking|network / hosting / infrastructure"                          
    ## [18] "human resources (hr)|career / job search"                                                
    ## [19] "human resources (hr)|enterprise software|career / job search|social networking|analytics"
    ## [20] "network / hosting / infrastructure|publishing"                                           
    ## [21] "analytics|social networking|network / hosting / infrastructure"                          
    ## [22] "human resources (hr)|analytics|marketing|career / job search"                            
    ## [23] "network / hosting / infrastructure|telecommunications|enterprise software"               
    ## [24] "network / hosting / infrastructure|marketing"                                            
    ## [25] "human resources (hr)|career / job search"                                                
    ## [26] "e-commerce|network / hosting / infrastructure"

``` r
# retrieve and view those industries that have '&' in them
startups$industry_of_company[ grepl(x = startups$industry_of_company, pattern = '&', fixed = TRUE) ]
```

    ## [1] "food & beverages|hospitality"                                 
    ## [2] "analytics|food & beverages|social networking|mobile"          
    ## [3] "network / hosting / infrastructure|food & beverages|analytics"
    ## [4] "e-commerce|food & beverages|mobile"                           
    ## [5] "food & beverages"                                             
    ## [6] "e-commerce|food & beverages|mobile"                           
    ## [7] "healthcare|analytics|mobile|food & beverages"                 
    ## [8] "e-commerce|food & beverages"

``` r
# retrieve and view those industries that have '(' in them
startups$industry_of_company[ grepl(x = startups$industry_of_company, pattern = '(', fixed = TRUE) ]
```

    ## [1] "human resources (hr)|marketing|career / job search"                                      
    ## [2] "human resources (hr)|enterprise software|career / job search|social networking|analytics"
    ## [3] "human resources (hr)|career / job search"                                                
    ## [4] "human resources (hr)|enterprise software|career / job search|social networking|analytics"
    ## [5] "human resources (hr)|analytics|marketing|career / job search"                            
    ## [6] "human resources (hr)|career / job search"

From the above inspections, it seems that all occurrences of `/` are in
the terms `network / hosting / infrastructure` and `career / job
search`. As for `&` and `(`, their occurrences are in the terms `food &
beverages` and `human resources (hr)`, respectively. Accordingly, we
perform the following modifications to the `industry_of_company`
variable.

``` r
# remove all occurrences of ' (hr)', ' /', ' &'
startups$industry_of_company <- gsub(startups$industry_of_company, pattern=' (hr)', replacement='', fixed=TRUE)
startups$industry_of_company <- gsub(startups$industry_of_company, pattern=' /',    replacement='', fixed=TRUE)
startups$industry_of_company <- gsub(startups$industry_of_company, pattern=' &',    replacement='', fixed=TRUE)

# replace spaces with underscores to merge multi-word terms
startups$industry_of_company <- gsub(startups$industry_of_company, pattern=' ',     replacement='_', fixed=TRUE)

# replace all occurrences of '|' with spaces to separate between terms
startups$industry_of_company <- gsub(startups$industry_of_company, pattern='|',     replacement=' ', fixed=TRUE)

head(startups$industry_of_company, 10)
```

    ##  [1] NA                                              
    ##  [2] "market_research marketing crowdfunding"        
    ##  [3] "analytics cloud_computing software_development"
    ##  [4] "mobile analytics"                              
    ##  [5] "analytics marketing enterprise_software"       
    ##  [6] "food_beverages hospitality"                    
    ##  [7] "analytics"                                     
    ##  [8] "cloud_computing network_hosting_infrastructure"
    ##  [9] "analytics mobile marketing"                    
    ## [10] "healthcare pharmaceuticals analytics"

Modifications have been applied successfully and as intended;
i.e. multi-word terms are merged with underscores, and the terms are
separated by spaces. Now, let’s generate the DTM and observe the
obtained terms.

``` r
# create the corpus object from the industry_of_company column
mycorpus <- corpus(startups$industry_of_company)

# generate the DTM using the formed corpus
dfm_industry <- dfm(mycorpus,               # the corpus to generate the DTM from
                    tolower = FALSE)        # data is already lowercase

# observe the 'terms' of the DTM
textplot_wordcloud(dfm_industry, color = c('red', 'pink', 'green', 'purple', 'orange', 'blue'))
```

![](FE_Art_2_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
# convert `dfm_industry` to data.frame and bind with response variable
startups_industry <- cbind(startups[,2], as.data.frame(dfm_industry))
```

    ## Warning: 'as.data.frame.dfm' is deprecated.
    ## Use 'convert(x, to "data.frame")' instead.
    ## See help("Deprecated")

``` r
glimpse(startups_industry)
```

    ## Observations: 472
    ## Variables: 42
    ## $ `dependent-company_status`     <chr> "Success", "Success", "Success"...
    ## $ document                       <chr> "text1", "text2", "text3", "tex...
    ## $ market_research                <dbl> 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ marketing                      <dbl> 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0...
    ## $ crowdfunding                   <dbl> 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ analytics                      <dbl> 0, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1...
    ## $ cloud_computing                <dbl> 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0...
    ## $ software_development           <dbl> 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ mobile                         <dbl> 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0...
    ## $ enterprise_software            <dbl> 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1...
    ## $ food_beverages                 <dbl> 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0...
    ## $ hospitality                    <dbl> 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0...
    ## $ network_hosting_infrastructure <dbl> 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0...
    ## $ healthcare                     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0...
    ## $ pharmaceuticals                <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0...
    ## $ media                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ finance                        <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ music                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ `e-commerce`                   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ gaming                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ advertising                    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ retail                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ security                       <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ email                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ human_resources                <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ career_job_search              <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ publishing                     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ education                      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ energy                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ deals                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ entertainment                  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ transportation                 <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ social_networking              <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ real_estate                    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ search                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ telecommunications             <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ insurance                      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ cleantech                      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ space_travel                   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ classifieds                    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ travel                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ government                     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...

The bind appears to be successful. Now, let’s view the frequency of each
of the industries among the list of companies in the dataset. Let’s
remove the `NA` industry as it does nothing for us.

``` r
# remove the NA industry (column)

glimpse(startups_industry)
```

    ## Observations: 472
    ## Variables: 42
    ## $ `dependent-company_status`     <chr> "Success", "Success", "Success"...
    ## $ document                       <chr> "text1", "text2", "text3", "tex...
    ## $ market_research                <dbl> 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ marketing                      <dbl> 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0...
    ## $ crowdfunding                   <dbl> 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ analytics                      <dbl> 0, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1...
    ## $ cloud_computing                <dbl> 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0...
    ## $ software_development           <dbl> 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ mobile                         <dbl> 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0...
    ## $ enterprise_software            <dbl> 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1...
    ## $ food_beverages                 <dbl> 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0...
    ## $ hospitality                    <dbl> 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0...
    ## $ network_hosting_infrastructure <dbl> 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0...
    ## $ healthcare                     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0...
    ## $ pharmaceuticals                <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0...
    ## $ media                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ finance                        <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ music                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ `e-commerce`                   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ gaming                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ advertising                    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ retail                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ security                       <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ email                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ human_resources                <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ career_job_search              <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ publishing                     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ education                      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ energy                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ deals                          <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ entertainment                  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ transportation                 <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ social_networking              <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ real_estate                    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ search                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ telecommunications             <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ insurance                      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ cleantech                      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ space_travel                   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ classifieds                    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ travel                         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ government                     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...

Alright. Let’s now view the successes and failures under each industry.
Specifically, we want (1) to have the rows be the industries and (2) to
have two columns, *success* and *fail*, showing us the numbers of
successful and failed companies, respectively, in each of the
industries.

``` r
# (1) have the rows be the industries
industry_frequencies <-
  startups_industry %>% 
  # "industry" column gathers the industries (i.e. keys) from the former columns
  # "binary" column holds the values from the former columns
  gather("industry", "binary", 2:41) %>% 
  #summarise the binary columns from the DTM into two values per industry (one for success/fail each)
  group_by(`dependent-company_status`, industry)

industry_frequencies
```

    ## # A tibble: 18,880 x 4
    ## # Groups:   dependent-company_status, industry [80]
    ##    `dependent-company_status` government industry binary
    ##    <chr>                           <dbl> <chr>    <chr> 
    ##  1 Success                             0 document text1 
    ##  2 Success                             0 document text2 
    ##  3 Success                             0 document text3 
    ##  4 Success                             0 document text4 
    ##  5 Success                             0 document text5 
    ##  6 Success                             0 document text6 
    ##  7 Success                             0 document text7 
    ##  8 Success                             0 document text8 
    ##  9 Success                             0 document text9 
    ## 10 Success                             0 document text10
    ## # ... with 18,870 more rows
