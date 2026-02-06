I4R replicationGames (Wellington, NZ)
================
Steph, Sam, Iggy
2025-12-04

## *Replication attempt of (Ma et al., 2023) Racial/ethnic disparities in PM<sub>2.5</sub>-attributable cardiovascular mortality burden in the United States*

<https://www.nature.com/articles/s41562-023-01694-7>

# Details

## Replication attempts

- Figure 1a ➝ `fig_1a_rep` ✓

- Figure 1b ➝ `fig_1b_rep` ✓

- ~~Figure 1c~~ *➝ see attempted section*

- ~~Supplementary Figure 2~~ *➝ see attempted section*

## Repository

**All the data and code from our replication attempt are in the
following GitHub repository:
<a href="https://github.com/samsiljee/PM2.5_CVD_mortality_US"
class="uri"><em>https://github.com/samsiljee/PM2.5_CVD_mortality_US</em></a>**

<span class="smallcaps">*Our replication repository was branched from
the github repository of the original article
(<https://github.com/CHENlab-Yale/PM2.5_CVD_mortality_US>).*</span>

### Code

**Final version** (this present file):
***I4R_replication_code_Ma_et_al_2023_NHB_PM2.5_CVD_mort_US.Rmd***

*Previous version - replicationReportWorking.Rmd*

### Data

All data is contained within the replication repository, including the
data provided by the authors.

- *US_continental_counties_3103.zip*

- *Data_2000-2016_county_monthly_combined.7z*

In addition to the data retrieved from
<https://wonder.cdc.gov/cmf-icd10.html> and generated as a part of the
replication attempt.

- *CompMort_01_08.csv*

- *CompMort_09_16.csv*

- *missing_columns.txt*

------------------------------------------------------------------------

# Setup

### Load Packages

``` r
# Load Packages
library(readr) # read csv file in
library(dplyr) # wrangling
library(tidyr) # Remove NAs in merged data
library(lubridate) # dates 
library(sf) # for reading .shp data | fig1a & 1b
library(tsModel) # for runMean() - e.g. fig 1c
library(ggplot2) # graphing | fig1a & 1b
```

## Prepare Data

``` r
# Set seed for randomisation purposes
set.seed(2025)

# Setup - Load in data file
# set working directory as source file location
# Read in upzipped .csv data from "input" folder within

# Read in .shp US countys geospatial data us sf package
US.county <- st_read("input/US_continental_counties_3103/US_continental_counties_3103.shp") 

# Read in PM2.5 & evironmental data 
df1 <- read_csv("input/Data_2000-2016_county_monthly_combined.csv")



# ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ 
# The data below was not provided with the original article
# retrieved from CDC WONDER in a further replication attempt
# Read in overall mortality data 
df2 <- rbind(
  read_csv("input/CompMort_01_08.csv"),
  read_csv("input/CompMort_09_16.csv")
)

# Merge data 
df3 <- merge(
  mutate(filter(df2, !is.na("Year")), GEOID = `County Code`, year = Year),
  df1,
  by = c("GEOID", "year")
)

# Clean data
df3 <- df3 %>%
  mutate(
    CVD.adj = as.numeric(`Age Adjusted Rate`, na.rm = TRUE),
    GEOID = as.factor(GEOID)
  ) %>% # change column classes
  dplyr::select(
    c(
      GEOID,
      year_month,
      year,
      month,
      PM25,
      NO2,
      O3,
      Tmean,
      dewT,
      pop.total,
      pop.Male,
      pop.Female,
      pop.White,
      pop.Black,
      pop.Hispanic,
      CVD.adj
    )
  ) # Select required columns

# Add in dummy data for columns missing from mortality data
missing_columns <- readLines("input/missing_columns.txt")

# Make dummy data with only positive numbers
dummy_data <- matrix(
  data = abs(rnorm(length(missing_columns) * nrow(df3))),
  nrow = nrow(df3),
  ncol = length(missing_columns)
)

colnames(dummy_data) <- missing_columns

df4 <- bind_cols(
  df3,
  dummy_data
)
```

# Successfully Replicated

This section details the successful replication of Figures 1a & 1b from
the primary article. It uses the `df1` and `US.county` data prepared
above.

## Figure 1ab

#### Figure 1ab (Ma et al’s Code)

``` r
# from 'Figure 1ab.R'
# using packages dplyr & sf

# re assign df1 "Data_2000-2016_county_monthly_combined.csv" to data
data <- df1 # added 

## average PM2.5 (Figure 1a)
PM25.ave <- data %>%
  group_by(GEOID) %>%
  summarise(PM25_mean = mean(PM25))

## PM2.5 change (Figure 1b)
PM25.2000 <- data %>%
  filter(year == 2000) %>%
  group_by(GEOID) %>%
  summarise(PM25_2000 = mean(PM25))

PM25.2016 <- data %>%
  filter(year == 2016) %>%
  group_by(GEOID) %>%
  summarise(PM25_2016 = mean(PM25))

PM25.change <- PM25.2000 %>%
  left_join(PM25.2016, by = c("GEOID")) %>%
  mutate(PM25_chg = (PM25_2016 - PM25_2000) / PM25_2000 * 100) %>%
  dplyr::select(GEOID, PM25_chg)


## join with shapefile
PM25.ave$GEOID <- as.character(PM25.ave$GEOID)
PM25.change$GEOID <- as.character(PM25.change$GEOID)

US.county.F1 <- US.county %>%
  left_join(PM25.ave, by = "GEOID") %>%
  left_join(PM25.change, by = "GEOID")
```

## Figure 1ab (Our code)

This takes the data prepared above & uses a combination of functions
from `dplyr`, `sf`, & `ggplot2` to replicate the figures as an
alternative to the **ArcGIS Pro** software used in the original article.

- *Note: “ArcGIS Pro was used to make the final maps.” (Github \>
  Analysis Description.pdf \> 2.2 Descriptive Maps)*
  - *Attempted to use the sf package to map information instead.*

    - <a href="https://r-spatial.github.io/sf/articles/sf5.html"
      class="uri"><em>https://r-spatial.github.io/sf/articles/sf5.html</em></a>

    - <a href="https://r-spatial.github.io/sf/articles/sf5.html#ggplot2"
      class="uri"><em>https://r-spatial.github.io/sf/articles/sf5.html#ggplot2</em></a>

##### Colors from figures

``` r
# colors were mapped from the original paper using the eyedropper tool 
# Colors used in figure 1a
fig_1a_colors <- c("#006837", "#86CB67", "#FFFFBF", "#F98E52", "#A50026")

# Colors used in figure 1b
fig_1b_colors <- c("#313695", "#74ADD1", "#E0F3F8", "#F98E52", "#A50026")
```

### Figure 1a Replication

``` r
fig_1a_rep <- US.county.F1 %>%
  mutate( # make new column split by thresholds shown in fig1a
    PM25_mean_rank = case_when(
      PM25_mean > 12 ~ "12.00-13.74", 
      PM25_mean > 9 ~ "9.00-12.00",
      PM25_mean > 6 ~ "6.00-9.00",
      PM25_mean > 3 ~ "3.00-6.00",
      PM25_mean <= 3 ~ "2.15-3.00",
      TRUE ~ "F")) %>% # Default case: if none of the above are true
  ggplot() + # plot
  geom_sf(aes(fill = PM25_mean_rank)) +
  scale_fill_manual( # mapping color to rank/thresholds
    name = "Average PM2.5 \nconcentration (\u00B5g/m\u00b3)",
    breaks = c("2.15-3.00",
               "3.00-6.00",
               "6.00-9.00", 
               "9.00-12.00",
               "12.00-13.74"),
    values = c("#006837", 
               "#86CB67",
               "#FFFFBF",
               "#F98E52",
               "#A50026")) +
  theme_bw() +
  theme(
    legend.position = "inside",
    legend.position.inside = c(.92, .17),
    legend.title = element_text(size = 10, hjust = 1),
  )

fig_1a_rep
```

![](I4R_replication_code_Ma_et_al_2023_NHB_PM2.5_CVD_mort_US_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

### Figure 1b Replication

``` r
fig_1b_rep <- US.county.F1 %>%
  mutate( # make new column split by thresholds shown in fig1b
    PM25_chg_rank = case_when(
      PM25_chg > 20 ~ "20.00-52.99",
      PM25_chg > 0 ~ "0.00-20.00",
      PM25_chg > -20 ~ "-20.00-0.00",
      PM25_chg > -40 ~ "-40.00--20.00", 
      PM25_chg <= -40 ~ "-60.43--40.00",
      TRUE ~ "F")) %>% # Default case: if none of the above are true
  ggplot() + # plot
  geom_sf(aes(fill = PM25_chg_rank)) + 
  scale_fill_manual( # mapping color to rank/thresholds
    name = "Change in PM2.5 \nconcentration (%)",
    breaks = c(
      "-60.43--40.00",
      "-40.00--20.00",
      "-20.00-0.00",
      "0.00-20.00",
      "20.00-52.99"),
    values = c(
      "#313695",
      "#74ADD1",
      "#E0F3F8",
      "#F98E52",
      "#A50026")) +
  theme_bw() +
  theme(
    legend.position = "inside",
    legend.position.inside = c(.93, .17),
    legend.title = element_text(size = 10, hjust = 1),
  )

fig_1b_rep
```

![](I4R_replication_code_Ma_et_al_2023_NHB_PM2.5_CVD_mort_US_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

------------------------------------------------------------------------

# Attempted

### Figure 1c (**Ma et al’s Code**)

``` r
# library(dplyr)
# library(ggplot2)
# library(cowplot)
# library(MetBrewer)
# 
# ## filter for cardiovascular mortality data
# mortality.CVD <- mortality %>%
#   filter(ICD.category == "I" & ICD.number >= 0 & ICD.number <= 99.99) %>%
#   dplyr::select(year_month, res_FIPS, race, hispanic) %>%
#   rename(GEOID = res_FIPS) %>%
#   mutate(
#     race_cat = case_when(
#       race == 1 & hispanic == 0 ~ "non-Hispanic White",
#       race == 2 & hispanic == 0 ~ "non-Hispanic Black",
#       hispanic == 1 ~ "Hispanic"
#     ),
#     race_cat = factor(race_cat, levels = c("non-Hispanic White", "non-Hispanic Black", "Hispanic"))
#   ) %>%
#   dplyr::select(GEOID, year_month, race_cat) %>%
#   filter(is.na(race_cat) == F)
# 
# rm(mortality)
# 
# 
# ## select PM2.5 data
# data.AP <- data %>%
#   dplyr::select(GEOID, year_month, year, PM25)
# 
# ## 12-month moving average of PM2.5 concentration
# data.list <- split(data.AP, as.factor(data$GEOID))
# for (j in 1:length(data.list)) {
#   data.list[[j]][, paste0("PM25_lag0", 11)] <- tsModel::runMean(data.list[[j]]$PM25, c(0:11))
# }
# 
# data.AP.01to16 <- bind_rows(data.list) %>%
#   filter(year >= 2001) %>%
#   dplyr::select(-PM25, -year)
# 
# 
# ## individual-level long-term PM2.5 exposure
# AP.individual <- mortality.CVD %>%
#   left_join(data.AP, by = c("GEOID", "year_month")) %>%
#   left_join(data.AP.01to16, by = c("GEOID", "year_month"))
# 
# ## make the violin plot
# theme_set(theme_cowplot())
# 
# plot.individual.12M <- ggplot(AP.individual, aes(x = race_cat, y = PM25_lag011, fill = race_cat)) +
#   geom_violin() +
#   scale_fill_met_d("Kandinsky", direction = 1) +
#   geom_boxplot(width = 0.1, color = "grey", alpha = 0.2) +
#   xlab("") +
#   ylab("12-month moving average of PM2.5") +
#   theme(legend.position = "none")
```

### Figure 1c ()

``` r
# mortality.CVD_df4 <- df4 %>%


# Taken directly from their code
## select PM2.5 data
data.AP <- data %>%
  dplyr::select(GEOID, year_month, year, PM25)

## 12-month moving average of PM2.5 concentration
data.list <- split(data.AP, as.factor(data$GEOID))
for (j in 1:length(data.list)) {
  data.list[[j]][, paste0("PM25_lag0", 11)] <- tsModel::runMean(data.list[[j]]$PM25, c(0:11))
}

data.AP.01to16 <- bind_rows(data.list) %>%
  filter(year >= 2001) %>%
  dplyr::select(-PM25, -year)
```

## Supplementary Figure 2 (our code)

The reproduced figures are generally similar to the original paper;
however, the data retrieved from CDC WONDER was a simplified set, so the
figures appear less granular.

``` r
# Cut off at <5000
sfig_2 <- df4 %>%
  filter(pop.total < 5000) %>%
  mutate(year_month = lubridate::ym(year_month)) %>% 
  group_by(year_month) %>%
  mutate(monthly_mean_CVD.adj = mean(CVD.adj, na.rm = TRUE)) %>%
  ggplot(aes(x = year_month, y = monthly_mean_CVD.adj)) +
  geom_line() +
  scale_x_date(date_labels = "%Y-%m", date_breaks = "6 month",
               labels = ) +
  labs(title = "Supplementary Figure 2 (pop <5000)") +
  #geom_smooth(method=lm , color="red", fill="#69b3a2", se=TRUE) 
  theme_bw() +
  theme(
    axis.text.x = element_text(angle = 90, vjust = 0.5), # vertical x labels
        )
  
sfig_2
```

![](I4R_replication_code_Ma_et_al_2023_NHB_PM2.5_CVD_mort_US_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
# Cut off at <50000
sfig_2.v2 <- df4 %>%
  filter(pop.total < 50000) %>%
  mutate(year_month = lubridate::ym(year_month)) %>% 
  group_by(year_month) %>%
  mutate(monthly_mean_CVD.adj = mean(CVD.adj, na.rm = TRUE)) %>%
  ggplot(aes(x = year_month, y = monthly_mean_CVD.adj)) +
  geom_line() +
  scale_x_date(date_labels = "%Y-%m", date_breaks = "6 month",
               labels = ) +
  labs(title = "Supplementary Figure 2 v2 (pop <50000)") +
  #geom_smooth(method=lm , color="red", fill="#69b3a2", se=TRUE) 
  theme_bw() +
  theme(
    axis.text.x = element_text(angle = 90, vjust = 0.5), # vertical x labels
        )
  
sfig_2.v2
```

![](I4R_replication_code_Ma_et_al_2023_NHB_PM2.5_CVD_mort_US_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

------------------------------------------------------------------------

``` r
# end - write out package versions for .html or .md knitted files
sessionInfo()
```

    ## R version 4.4.1 (2024-06-14 ucrt)
    ## Platform: x86_64-w64-mingw32/x64
    ## Running under: Windows 10 x64 (build 19045)
    ## 
    ## Matrix products: default
    ## 
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_New Zealand.utf8  LC_CTYPE=English_New Zealand.utf8   
    ## [3] LC_MONETARY=English_New Zealand.utf8 LC_NUMERIC=C                        
    ## [5] LC_TIME=English_New Zealand.utf8    
    ## 
    ## time zone: Pacific/Auckland
    ## tzcode source: internal
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] ggplot2_4.0.1   tsModel_0.6-2   sf_1.0-23       lubridate_1.9.4
    ## [5] tidyr_1.3.1     dplyr_1.1.4     readr_2.1.6    
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] generics_0.1.4     class_7.3-22       KernSmooth_2.23-24 hms_1.1.4         
    ##  [5] digest_0.6.37      magrittr_2.0.3     evaluate_1.0.5     grid_4.4.1        
    ##  [9] timechange_0.3.0   RColorBrewer_1.1-3 fastmap_1.2.0      e1071_1.7-16      
    ## [13] DBI_1.2.3          purrr_1.0.2        scales_1.4.0       cli_3.6.3         
    ## [17] rlang_1.1.4        crayon_1.5.3       units_1.0-0        bit64_4.6.0-1     
    ## [21] splines_4.4.1      withr_3.0.2        yaml_2.3.10        tools_4.4.1       
    ## [25] parallel_4.4.1     tzdb_0.5.0         vctrs_0.6.5        R6_2.6.1          
    ## [29] proxy_0.4-27       lifecycle_1.0.4    classInt_0.4-11    bit_4.6.0         
    ## [33] vroom_1.6.6        pkgconfig_2.0.3    pillar_1.11.1      gtable_0.3.6      
    ## [37] glue_1.8.0         Rcpp_1.1.0         xfun_0.54          tibble_3.2.1      
    ## [41] tidyselect_1.2.1   rstudioapi_0.17.1  knitr_1.50         farver_2.1.2      
    ## [45] htmltools_0.5.8.1  labeling_0.4.3     rmarkdown_2.30     compiler_4.4.1    
    ## [49] S7_0.2.1
