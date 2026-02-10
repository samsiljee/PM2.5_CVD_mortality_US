# Sourcing the data for replication

## PM2.5 data
THe PM2.5 data can be sourced from the github repository of the original paper in a .zip file.
Download and extract the .zip file, found here: [Data_2000-2016_county_monthly_combined.7z](https://github.com/CHENlab-Yale/PM2.5_CVD_mortality_US/blob/main/Data_2000-2016_county_monthly_combined.7z)

## .shp file
The .shp file can be found in the github repository of the original paper in a .zip file.
Download and extract the .zip file, found here: [US_continental_counties_3103.zip](https://github.com/CHENlab-Yale/PM2.5_CVD_mortality_US/blob/main/US_continental_counties_3103.zip)

## Mortality data
The mortality data can be sourced from the [CDC WONDER portal](https://wonder.cdc.gov/controller/datarequest/D140)
Follow these steps to get the subset of data used in the replication project:
1. Click on the link "http://wonder.cdc.gov/cmf-icd10.html"
2. Agree to the terms of data use
3. In the "Organize table layout" section group results by County and Year, and include Age Adjusted Rates alongside the default measures
4. in the "Select location, State/County or CBSA" section, select all states except for Alaska and Hawaii
5. Leave section 2.a at the default values
6. In the "Select years and demographics" section, under "race" select "Black or African American" and "White", under "Hispanic Origin" select "Hispanic or Latino" and "Not Hispanic or Latino", and under "Year" select 2001 to 2008 initially. Subsequently do a second data request for the years 2009 to 2016. The resulting data for all the years together is too large to be allowed by the portal
7. In the "Select cause of death" section, use the ICD-10 codes, and select "I00-I99"
8. In the "Other options" section check the "Export Results" box, change "Export Type" to CSV, check the "Show Zero Values" and "Show Suppressed Values" boxes, change Precision to three decimal places, and click "send"
9. Rename the downloaded file appropriately

## Missing column names
Column names to be added with blanks for missing data can be found in the replication repository at [missing_columns.txt](https://github.com/samsiljee/PM2.5_CVD_mortality_US/blob/main/missing_columns.txt)
