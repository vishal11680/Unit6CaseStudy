# Analyzing/Matching GDP data with OECD status for selected countries
Vishal Ahir  
June 12, 2016  

### Purpose
The purpose of this analysis is to read/prep GDP ranking data set of 190 countries for year 2012, merge it with a separate data set that contains details about OECD (Organization for Economic Co-operation and Development) status of countries and provide some answers related to OECD and non-OECD country groups GDP rankings. We will also divide countries based on GDP Rank into 5 groups and see how these groups match to their respective Income Groups.  

OECD - The Organization for Economic Co-operation and Development  
The mission of the Organisation for Economic Co-operation and Development is to promote policies that will improve the economic and social well-being of people around the world. - (**www.oecd.org**)  

Following are the URLs of 2 data sets that will be utilized for this analysis:  
1)  GDP Data : https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv  
2)  Edu Data : https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FEDSTATS_Country.csv  

### Dowloading GDP data set  
  

```r
# As a first step, we will store URL for GDP data in a R variable called URL1
URL1 <- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FGDP.csv"

#Since URL points to a CSV file that has variables separated by comma, we will try and read this data
# into a R data frame using read.csv command

GDPData <- read.csv(URL1, header = FALSE, sep = ",", quote = "\"", fill=TRUE, stringsAsFactors = FALSE)

#As you can see, we are storing the data from dowloaded CSV file into a data frame - GDPData.
#The header = False argument to this function indicates header from CSV file will not be used.
# The header from input file was skipped as there was no meaningful data present.

# Below we are using str() function to check out observations/variables for the downloaded dataset.
str(GDPData)
```

```
## 'data.frame':	331 obs. of  10 variables:
##  $ V1 : chr  "" "" "" "" ...
##  $ V2 : chr  "Gross domestic product 2012" "" "" "Ranking" ...
##  $ V3 : logi  NA NA NA NA NA NA ...
##  $ V4 : chr  "" "" "" "Economy" ...
##  $ V5 : chr  "" "" "(millions of" "US dollars)" ...
##  $ V6 : chr  "" "" "" "" ...
##  $ V7 : logi  NA NA NA NA NA NA ...
##  $ V8 : logi  NA NA NA NA NA NA ...
##  $ V9 : logi  NA NA NA NA NA NA ...
##  $ V10: logi  NA NA NA NA NA NA ...
```
### Dowloading EDU data set


```r
# 2nd data set, loaded using same command as first one.
# In this case, header argument is kept true as the file contains meaningful variable header names.

URL2 <- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FEDSTATS_Country.csv" 
edu_df <- read.csv(URL2, header=TRUE, sep = ",", quote="\"", fill=TRUE, stringsAsFactors = FALSE)

# Below we are using str() function to check out observations/variables for the downloaded dataset.
str(edu_df)
```

```
## 'data.frame':	234 obs. of  31 variables:
##  $ CountryCode                                      : chr  "ABW" "ADO" "AFG" "AGO" ...
##  $ Long.Name                                        : chr  "Aruba" "Principality of Andorra" "Islamic State of Afghanistan" "People's Republic of Angola" ...
##  $ Income.Group                                     : chr  "High income: nonOECD" "High income: nonOECD" "Low income" "Lower middle income" ...
##  $ Region                                           : chr  "Latin America & Caribbean" "Europe & Central Asia" "South Asia" "Sub-Saharan Africa" ...
##  $ Lending.category                                 : chr  "" "" "IDA" "IDA" ...
##  $ Other.groups                                     : chr  "" "" "HIPC" "" ...
##  $ Currency.Unit                                    : chr  "Aruban florin" "Euro" "Afghan afghani" "Angolan kwanza" ...
##  $ Latest.population.census                         : chr  "2000" "Register based" "1979" "1970" ...
##  $ Latest.household.survey                          : chr  "" "" "MICS, 2003" "MICS, 2001, MIS, 2006/07" ...
##  $ Special.Notes                                    : chr  "" "" "Fiscal year end: March 20; reporting period for national accounts data: FY." "" ...
##  $ National.accounts.base.year                      : chr  "1995" "" "2002/2003" "1997" ...
##  $ National.accounts.reference.year                 : int  NA NA NA NA 1996 NA NA 1996 NA NA ...
##  $ System.of.National.Accounts                      : int  NA NA NA NA 1993 NA 1993 1993 NA NA ...
##  $ SNA.price.valuation                              : chr  "" "" "VAB" "VAP" ...
##  $ Alternative.conversion.factor                    : chr  "" "" "" "1991-96" ...
##  $ PPP.survey.year                                  : int  NA NA NA 2005 2005 NA 2005 2005 NA NA ...
##  $ Balance.of.Payments.Manual.in.use                : chr  "" "" "" "BPM5" ...
##  $ External.debt.Reporting.status                   : chr  "" "" "Actual" "Actual" ...
##  $ System.of.trade                                  : chr  "Special" "General" "General" "Special" ...
##  $ Government.Accounting.concept                    : chr  "" "" "Consolidated" "" ...
##  $ IMF.data.dissemination.standard                  : chr  "" "" "GDDS" "GDDS" ...
##  $ Source.of.most.recent.Income.and.expenditure.data: chr  "" "" "" "IHS, 2000" ...
##  $ Vital.registration.complete                      : chr  "" "Yes" "" "" ...
##  $ Latest.agricultural.census                       : chr  "" "" "" "1964-65" ...
##  $ Latest.industrial.data                           : int  NA NA NA NA 2005 NA 2001 NA NA NA ...
##  $ Latest.trade.data                                : int  2008 2006 2008 1991 2008 2008 2008 2008 NA 2007 ...
##  $ Latest.water.withdrawal.data                     : int  NA NA 2000 2000 2000 2005 2000 2000 NA 1990 ...
##  $ X2.alpha.code                                    : chr  "AW" "AD" "AF" "AO" ...
##  $ WB.2.code                                        : chr  "AW" "AD" "AF" "AO" ...
##  $ Table.Name                                       : chr  "Aruba" "Andorra" "Afghanistan" "Angola" ...
##  $ Short.Name                                       : chr  "Aruba" "Andorra" "Afghanistan" "Angola" ...
```

### Prepping GDP data for analysis  
In this block we will prep the GDP data - remove records with missing values, converting data types, sorting, etc.


```r
# 
library(plyr)

# From visually inspecting the GDP data set, it is obvious the first 5 rows are of no use as they do not contain
# valid header fields and a lot of extra commas.
# So we will remove first 5 rows of the GDP data set.
GDPData <-GDPData [-(1:5),]

# The second column of GDP Data set contains GDP Rank so we will convert that column to an integer field.
GDPData$V2 <- as.integer(GDPData$V2)
```

```
## Warning: NAs introduced by coercion
```

```r
# Now we strip the GDP data set down to only first 5 columns as only these columns are relevant to our analysis.
GDPData2 <- GDPData[,1:5]

#Assigning colume names
names(GDPData2) <- c("CountryCode", "Rank", "Blank", "Economy", "USDollars")

#Below we are removing rows that are missing Rank values for any countries.
NAcount <- sum(is.na(GDPData2$Rank))
GDPData2 <- subset(x = GDPData2, !is.na(GDPData2$Rank))

# The fifth column of GDP data set contains actual GDP in terms of millions of dollars for respective country.
# However the format of that field is XXX,XXX,XXX. So in order to convert this field to numeric we need
# remove those commas and ensure only numbers remain.
GDPData2$USDollars <- gsub(",", "", GDPData2$USDollars)

# Now converting USDollars to numeric field.
GDPData2$USDollars <- as.numeric(GDPData2$USDollars)

# Sorting GDPData2 DF using USDollars.
GDPData2 <- arrange(GDPData2, USDollars)

str(GDPData2)
```

```
## 'data.frame':	190 obs. of  5 variables:
##  $ CountryCode: chr  "TUV" "KIR" "MHL" "PLW" ...
##  $ Rank       : int  190 189 188 187 186 185 184 183 182 181 ...
##  $ Blank      : logi  NA NA NA NA NA NA ...
##  $ Economy    : chr  "Tuvalu" "Kiribati" "Marshall Islands" "Palau" ...
##  $ USDollars  : num  40 175 182 228 263 326 472 480 596 684 ...
```

```r
summary(GDPData2$USDollars)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
##       40     7005    27640   377700   205300 16240000
```

*Out of 331 observations in GDP Dataset, 5 rows from header were removed as they were invalid and from remaining 326 observatiaons, count of observations missing GDP rank is : 136*


### Prepping EDU data set for analysis  
In this block we will prep the EDU data - remove records with missing values, converting data types, sorting, etc.


```r
#Taking only the first five columns from the edu dataset as only those are relevant.
edu_df_5 <- edu_df[ , 1:5]

#Checking all unique values and their counts using table func.
table(edu_df_5$Income.Group)
```

```
## 
##                      High income: nonOECD    High income: OECD 
##                   24                   37                   30 
##           Low income  Lower middle income  Upper middle income 
##                   40                   56                   47
```

```r
blank_ig <- subset(edu_df_5, Income.Group == "")
blank_ig_cnt <- nrow(blank_ig)
```

*We have 24 missing values for Income Group variable in the EDU dataset.*  


### Merging 2 data sets


```r
#Now that we have both data sets ready for analysis, we can merge these datasets using common variable "CountryCode".
join_gdp_edu <- merge(GDPData2, edu_df_5, by="CountryCode", all = T)

# Analyzing how many values are missing for Rank field in combined dataset. We can see we have 45 NAs
summary(join_gdp_edu$Rank)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    1.00   48.25   95.50   95.49  142.80  190.00      45
```

```r
# Analyzing how many values are missing for USDollars (GDP) field in combined dataset. We can see we have 45 NAs as well.
summary(join_gdp_edu$USDollars)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
##       40     7005    27640   377700   205300 16240000       45
```

```r
# Removing records with missing values.
na_rank <- sum(is.na(join_gdp_edu$Rank))
na_inc_grp <- sum(is.na(join_gdp_edu$Income.Group))
gdp_edu_no_na <- subset(join_gdp_edu, !is.na(Rank) & !is.na(Income.Group))

# Sorting combined data set on USDollars (GDP) ascending.
gdp_edu_no_na <- arrange(gdp_edu_no_na, USDollars)
```

*In the merged data frame we see Rank variable has 45 missing values and "Income Group" has 1 missing values.*


### Questions related to analysis

* Question 1 : Match the data based on the country shortcode. How many of the IDs match?  


```r
# We will match function between GDPData2 and edu_df_5 data frames to see how many rows match
# Rows that dont match will be marked as 0 else we will get the record number for the record that matches.

match_count <- match(GDPData2$CountryCode, edu_df_5$CountryCode, nomatch=0)

# Summing up all the records that match.
cnt <- sum(match_count != 0)
```
**Answer**  In the combined data set, total match count based on IDs is : 189 


* Question 2 : Sort the data frame in ascending order by GDP rank (so United States is last). What is the 13th country in the resulting data frame?  


```r
# Sorting combined data set on USDollars (GDP) ascending.
gdp_edu_no_na <- arrange(gdp_edu_no_na, USDollars)
thirteen_row <- gdp_edu_no_na[13,]
cntry <- thirteen_row$Long.Name
```
**Answer** Thirteen country is : St. Kitts and Nevis

* Question 3 : What are the average GDP rankings for the "High income: OECD" and "High income: nonOECD" groups?  


```r
# Take subset out of gdp_edu_no_na DF Rank Variable for only those records where Income Group is High Income OECD.
# This subset or Rank is then passed to mean() function to get average rank of such countries.
avg_rank_oecd <- mean(subset(gdp_edu_no_na$Rank, gdp_edu_no_na$Income.Group == "High income: OECD" ))

# Take subset out of gdp_edu_no_na DF Rank Variable for only those records where Income Group is High Income nonOECD.
# This subset or Rank is then passed to mean() function to get average rank of such countries.
avg_rank_non_oecd <- mean(subset(gdp_edu_no_na$Rank, gdp_edu_no_na$Income.Group == "High income: nonOECD" ))
```

***Answer*** Average GDP Ranking for High Income OECD is : 32.9666667  
             Average GDP Ranking for High Income nonOECD is : 91.9130435  

* Question 4 : Plot the GDP for all of the countries. Use ggplot2 to color your plot by Income Group.  


```r
#We will use qplot to plot GDP for all countries.
library(ggplot2)

# Plotting chart with raw USDollars(GDP) values.
#qplot (CountryCode, USDollars, data = gdp_edu_no_na, color=Income.Group, xlab="Country", ylab = "GDP")
ggplot(gdp_edu_no_na, aes(CountryCode, USDollars, color = Income.Group)) + geom_point() +
  ggtitle("GDP Plot using 2012 data") + labs(x="Country", y="GDP - 2012 Data")
```

![](RubricCaseStudy_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
#Plotting same chart using log of USDollars.
#qplot (CountryCode, log(USDollars), data = gdp_edu_no_na, color=Income.Group, xlab="Country", ylab = "log(GDP)")

ggplot(gdp_edu_no_na, aes(CountryCode, log(USDollars), color = Income.Group)) + geom_point() +
  ggtitle("GDP Plotted using log(GDP) - 2012 data") + labs(x="Country", y="log(GDP) - 2012 Data")
```

![](RubricCaseStudy_files/figure-html/unnamed-chunk-9-2.png)<!-- -->

Few points on plot for GDP of all countries  
    * Without log of GDP, the plot shows 2 outliers from High Income OECD group and 1 outlier from Lower Middle Income group.  
    * Without log of GDP, the distribution appears a little right skewed when we exclude outliers.  
    * With the log of GDP, the plot shows 1 outlier from High Income OECD group and 1 from Lower Middle Income group.  
    * With the the distribution appears equal.  

* Question 5 : Cut the GDP ranking into 5 separate quantile groups. Make a table versus Income.Group. How many countries are Lower middle income but among the 38 nations with highest GDP?  


```r
# Breaking 189 ranks into 5 groups.
attach(gdp_edu_no_na)
gdp_edu_no_na$rank_groups[Rank <= 38] <- 1
gdp_edu_no_na$rank_groups[Rank > 38 & Rank <= 76] <- 2
gdp_edu_no_na$rank_groups[Rank > 76 & Rank <= 114] <- 3
gdp_edu_no_na$rank_groups[Rank > 114 & Rank <= 152] <- 4
gdp_edu_no_na$rank_groups[Rank >  152] <- 5

# Create a table of rank_groups vs Income Groups to identify Lower Middle Income in nations with 
# highest GDP.

table(gdp_edu_no_na$rank_groups, gdp_edu_no_na$Income.Group)
```

```
##    
##     High income: nonOECD High income: OECD Low income Lower middle income
##   1                    4                18          0                   5
##   2                    5                10          1                  13
##   3                    8                 1          9                  12
##   4                    4                 1         16                   8
##   5                    2                 0         11                  16
##    
##     Upper middle income
##   1                  11
##   2                   9
##   3                   8
##   4                   8
##   5                   9
```
**Answer** After looking at table function call above, we can there are *5 countries* that are part of 38 countries with highest GDP and are also part of Lower Middle Income group.  

### Conclusion  

* Analysis of GDP and EDU datasets gave below insights
  * GDP Dataset
    * 1)  GDP dataset had a total of 326 observations with 10 variables.
    * 2)  Out of these only 190 countries had their GDP listed along with respective GDP Ranks. So 136 rows were NAs.
    * 3)  USA is the ranked as country with highest GDP among all 190 countries.
  * EDU Dataset
    * 1)  EDU dataset had a total  of 234 observations of 31 variables.
    * 2)  When we matched EDU dataset with GDP dataset, we found 189 matches and 45 non-matches in total.
    * 3)  Plot on GDP with log shows equal distribution.
  * NA Counts in both datasets
    * In GDP Dataset, count of missing values for Rank variable : 136
    * In EDU dataset, count of missing values for Income Group variable is : 24
    * In merged dataset, count of missing values for Rank variable is : 45
    * In merged dataset, count of missing values for Income Group variable is : 1
