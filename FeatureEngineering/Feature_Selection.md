Feature\_Selection
================
Mohammed Ali
April 13, 2019

## Import Data

For illustrating the various methods, we will use the ‘Ozone’ data from
‘mlbench’ package, except for Information value method which is
applicable for binary categorical response variables.

``` r
inputData <- read.csv("ozone.csv", stringsAsFactors=F)
datatable(inputData, rownames = FALSE)
```

![](Feature_Selection_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

## Random Forest Method

Random forest can be very effective to find a set of predictors that
best explains the variance in the response
variable.

``` r
cf1 <- cforest(ozone_reading ~ . , data= inputData, control=cforest_unbiased(mtry=2,ntree=50)) # fit the random forest
varimp(cf1) # get variable importance, based on mean decrease in accuracy
```

    ##                 Month          Day_of_month           Day_of_week 
    ##            0.85488913           -0.14553609           -0.10086419 
    ##       pressure_height            Wind_speed              Humidity 
    ##            4.12324498            0.03692366            4.56508001 
    ##  Temperature_Sandburg   Temperature_ElMonte Inversion_base_height 
    ##           15.67217679           16.17518814            3.97827424 
    ##     Pressure_gradient Inversion_temperature            Visibility 
    ##            3.62376874           12.54718110            3.54525768

``` r
cforestImpPlot(cf1)
```

![](Feature_Selection_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Relative Importance

``` r
lmMod <- lm(ozone_reading ~ . , data = inputData)  # fit lm() model
relImportance <- calc.relimp(lmMod, type = "lmg", rela = TRUE)  # calculate relative importance scaled to 100
sort(relImportance$lmg, decreasing=TRUE) 
```

    ##   Temperature_ElMonte  Temperature_Sandburg Inversion_temperature 
    ##          0.2179985762          0.2006324013          0.1621225687 
    ##       pressure_height              Humidity Inversion_base_height 
    ##          0.1120464264          0.1011576835          0.0864158303 
    ##            Visibility     Pressure_gradient                 Month 
    ##          0.0614176817          0.0279614158          0.0205933505 
    ##            Wind_speed          Day_of_month           Day_of_week 
    ##          0.0070146997          0.0022105122          0.0004288536

``` r
plot(relImportance)
```

![](Feature_Selection_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## MARS

The `earth` package implements variable importance based on Generalized
cross validation (GCV), number of subset models the variable occurs
(nsubsets) and residual sum of squares (RSS).

``` r
marsModel <- earth(ozone_reading ~ ., data=inputData) # build model
ev <- evimp (marsModel)
plot(ev)
```

![](Feature_Selection_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

## Step-wise Regression

If you have large number of predictors (\> 15), split the inputData in
chunks of 10 predictors with each chunk holding the
`responseVar`.

``` r
base.mod <- lm(ozone_reading ~ 1 , data= inputData)  # base intercept only model
all.mod <- lm(ozone_reading ~ . , data= inputData) # full model with all predictors
stepMod <- step(base.mod, scope = list(lower = base.mod, upper = all.mod), direction = "both", trace = 0, steps = 1000)  # perform step-wise algorithm
shortlistedVars <- names(unlist(stepMod[[1]])) # get the shortlisted variable.
shortlistedVars <- shortlistedVars[!shortlistedVars %in% "(Intercept)"]  # remove intercept 
print(shortlistedVars)
```

    ## [1] "Temperature_Sandburg"  "Humidity"              "Temperature_ElMonte"  
    ## [4] "Month"                 "pressure_height"       "Inversion_base_height"

The output could includes levels within categorical variables, since
‘stepwise’ is a linear regression based technique, as seen above.

If you have a large number of predictor variables (100+), the above code
may need to be placed in a loop that will run stepwise on sequential
chunks of predictors. The shortlisted variables can be accumulated for
further analysis towards the end of each iteration. This can be very
effective method, if you want to:

  - be highly selective about discarding valuable predictor variables.
  - build multiple models on the response variable.

## Boruta

The `Boruta` method can be used to decide if a variable is important or
not.

``` r
# Decide if a variable is important or not using Boruta
boruta_output <- Boruta(ozone_reading ~ ., data=na.omit(inputData), doTrace=2)  # perform Boruta search
```

    ##  1. run of importance source...

    ##  2. run of importance source...

    ##  3. run of importance source...

    ##  4. run of importance source...

    ##  5. run of importance source...

    ##  6. run of importance source...

    ##  7. run of importance source...

    ##  8. run of importance source...

    ##  9. run of importance source...

    ##  10. run of importance source...

    ##  11. run of importance source...

    ## After 11 iterations, +13 secs:

    ##  confirmed 9 attributes: Humidity, Inversion_base_height, Inversion_temperature, Month, Pressure_gradient and 4 more;

    ##  rejected 1 attribute: Day_of_week;

    ##  still have 2 attributes left.

    ##  12. run of importance source...

    ##  13. run of importance source...

    ##  14. run of importance source...

    ##  15. run of importance source...

    ## After 15 iterations, +18 secs:

    ##  rejected 1 attribute: Day_of_month;

    ##  still have 1 attribute left.

    ##  16. run of importance source...

    ##  17. run of importance source...

    ##  18. run of importance source...

    ##  19. run of importance source...

    ##  20. run of importance source...

    ##  21. run of importance source...

    ##  22. run of importance source...

    ##  23. run of importance source...

    ##  24. run of importance source...

    ##  25. run of importance source...

    ##  26. run of importance source...

    ##  27. run of importance source...

    ##  28. run of importance source...

    ##  29. run of importance source...

    ##  30. run of importance source...

    ##  31. run of importance source...

    ##  32. run of importance source...

    ##  33. run of importance source...

    ## After 33 iterations, +38 secs:

    ##  rejected 1 attribute: Wind_speed;

    ##  no more attributes left.

``` r
# Confirmed 10 attributes: Humidity, Inversion_base_height, Inversion_temperature, Month, Pressure_gradient and 5 more.
# Rejected 3 attributes: Day_of_month, Day_of_week, Wind_speed.
boruta_signif <- names(boruta_output$finalDecision[boruta_output$finalDecision %in% c("Confirmed", "Tentative")])  # collect Confirmed and Tentative variables
print(boruta_signif)  # significant variables
```

    ## [1] "Month"                 "pressure_height"       "Humidity"             
    ## [4] "Temperature_Sandburg"  "Temperature_ElMonte"   "Inversion_base_height"
    ## [7] "Pressure_gradient"     "Inversion_temperature" "Visibility"

``` r
#=> [1] "Month"                 "ozone_reading"         "pressure_height"      
#=> [4] "Humidity"              "Temperature_Sandburg"  "Temperature_ElMonte"  
#=> [7] "Inversion_base_height" "Pressure_gradient"     "Inversion_temperature"
#=> [10] "Visibility"
plot(boruta_output, cex.axis=.7, las=2, xlab="", main="Variable Importance")  # plot variable importance
```

![](Feature_Selection_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## CART

A popular automatic method for feature selection provided by the caret R
package is called Recursive Feature Elimination or RFE.

The example below provides an example of the RFE method on the Pima
Indians Diabetes dataset. A Random Forest algorithm is used on each
iteration to evaluate the model. The algorithm is configured to explore
all possible subsets of the attributes. All 8 attributes are selected in
this example, although in the plot showing the accuracy of the different
attribute subset sizes, we can see that just 4 attributes gives almost
comparable results.

``` r
# ensure the results are repeatable
set.seed(7)
# load the data
data(PimaIndiansDiabetes)
# define the control using a random forest selection function
control <- rfeControl(functions=rfFuncs, method="cv", number=10)
# run the RFE algorithm
results <- rfe(PimaIndiansDiabetes[,1:8], PimaIndiansDiabetes[,9], sizes=c(1:8), rfeControl=control)
# summarize the results
print(results)
```

    ## 
    ## Recursive feature selection
    ## 
    ## Outer resampling method: Cross-Validated (10 fold) 
    ## 
    ## Resampling performance over subset size:
    ## 
    ##  Variables Accuracy  Kappa AccuracySD KappaSD Selected
    ##          1   0.6926 0.2653    0.04916 0.10925         
    ##          2   0.7343 0.3906    0.04725 0.10847         
    ##          3   0.7356 0.4058    0.05105 0.11126         
    ##          4   0.7513 0.4435    0.04222 0.09472         
    ##          5   0.7604 0.4539    0.05007 0.11691        *
    ##          6   0.7499 0.4364    0.04327 0.09967         
    ##          7   0.7603 0.4574    0.04052 0.09838         
    ##          8   0.7590 0.4549    0.04804 0.10781         
    ## 
    ## The top 5 variables (out of 5):
    ##    glucose, mass, age, pregnant, insulin

``` r
# list the chosen features
predictors(results)
```

    ## [1] "glucose"  "mass"     "age"      "pregnant" "insulin"

``` r
# plot the results
plot(results, type=c("g", "o"))
```

![](Feature_Selection_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->
