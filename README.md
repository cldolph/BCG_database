---
title: "README: BCG_database Project Files"
author: "Christy Dolph and Virginia Callison"
output:
  pdf_document: default
  extra_dependencies: ['amsmath', 'xcolor']
editor_options: 
  chunk_output_type: console
---

## Project Description

This repository contains biological condition data assembled from multiple sources (local, state, regional, federal) for ~56,000 streams across the coterminous United States between 2000-2025, as well as the RMarkdown scripts that were used in data assembly and processing. Briefly, the goal was to compile Biological Condition Gradient (BCG; USEPA 2016) data for streams across the country. While many local, state and federal agencies measure and track the biological condition of streams and rivers within their borders using some type of biological index, BCG scores have only been computed for sites where states have either explicitly developed BCG scoring criteria according to EPA’s framework (Connecticut, Illinois, Indiana, Minnesota) or mapped other biological indices onto the concepts of the BCG (California, Florida, Ohio, Oregon, Texas). Other states use state-specific biotic indices to measure the biological integrity of streams. For these states, we developed a ‘BCG proxy’ that translated state-specific biotic index scores into the 6 BCG levels, as described in Vossler et al., (2023) and Bonacquist-Currin et al. (in prep). 


This repository includes the raw biological index data we compiled, as well as the scripts we used to translate biological indices to 'BCG proxies'. The scripts include steps to:
1. assemble all data and translate biological index scores into BCG proxies
2. clean and process data resulting in an BCG score for each stream site in each year it was sampled  
3. create maps of BCG data 

Sources of biological condition data for the following states are described in Vossler et al. (2023): IA, IL, IN, KY, MN, MO, NC, OH, TN, VA, WI, WV.  Sources of biological data for the following state, regional and US-scale datasets are described in Bonacquist-Currin et al. (in prep): AZ, CA, CT, FL, GA, ID, NE, NY, NC, OR, TX, WY, the USA EPA National Aquatic Resource Survey (NARS) dataset and the Chesapeake Bay Dataset. All data was processed using R statistical software (R Core Team, 2025). 

\

## System Requirements

This project was developed on R Studio version 4.2.1. 

\

## Filing Structure

The code and filing system uses relative directory paths. No changes to the file paths are necessary. 

1. `BCG_conversion_script.R` 

Contains documentation for how the output files were generated; i.e., for how BCG data was preprocessed.

2. `BCG_Output`

+ `EPA_BCG_allsamples_ALLROUNDS_HUC8ID_NHDCOMID_clean_HUC12.xlsx`

NOTE: This is the key data file to be used for data analysis! This file contains BCG scores for all sites compiled from state and other agencies. Note that this file includes sites that were assessed by more than once agency - so there may be multiple scores for a site based on the same sampling event, but different agency scoring criteria. This data also includes repeat visits over time to some sites where available. This file includes HUC8 watershed ID information for sample sites, as well as NHD IDs (called "COMID"") for each site. The NHD is the 'National Hydrography Dataset'. It shows the location of all stream reaches in the US. Each stream reach (ie, each section of stream) has a unique ID (COMID). These IDs can be used to join to geospatial attributes to the EPA StreamCat dataset (later on down the road if we get to that point)

+ `EPA_BCG_data_allsamples_ALLROUNDS.xlsx` same as first data file described above, but without HUC8 or NHD ID info

+ `EPA_Round1_data" and "EPA_Round2_data` contain input data gathered from states and other agencies that were used to generate compiled BCG scores.


3. `HUC_spatial_data`

Spatial data folder for mapping HUC 2 and HUC 8 watershed regions.

4. `Prelminary_analysis_scripts`

Contains script files, in numerical workflow order. 

5. `Prelminary_analysis_output` 

Contains output from script files (descriptive graphics, tables, intermediate data sets)

\

## Set Up & Installation

1. Download `~/BCG-analysis.zip` to local file system.

2. Unzip the files to a folder labeled `~/BCG-analysis`.

3. Still in the local file system, navigate to `~/BCG-analysis` double click the `BCG-analysisRproj` file. This will open the project in R Studio and set the working directory to `~/BCG-analysis`.

4. Read this README document **closely and completely** as it contains important instructions on how to run the project scripts.

5. Open R Studio. Navigate to the `~/Preliminary_analysis_scripts` folder and run the scripts in numerical order, beginning with `0.0_workspace_setup`. Follow the notes in each script file for detailed instructions.

6. To access script output, navigate to `~/Preliminary_analysis_output`


To run a script file, highlight the code in the file and click "Run" (upper left corner of the script file) or equivalently type `Ctrl Enter`. 

\

## Script & Output Details


### `0.0_workspace_setup.R`

This file sets up the work space with a relative path system, as well as all the necessary packages to run the subsequent scripts. There are two functions defined in this script: 

+ `notin`: returns the values that are NOT present in a list or vector
+ `deg_dist`: returns the degree distance between two lat/long coordinates based on simple geometry. This function is a useful check for overlap and intersections between sites. 

### `0.1_qc.R`

This file creates several clean intermediate data sets for subsequent workflow. We identified the following issues: 

+ Sites that were sampled multiple times in the same year (sometimes by the same agency) 
+ Sites with wide disparities in scores generated through multi-sampling

Site were generally sampled non-randomly by state/regional agencies with disparate programs and policy goals. This presents an analysis problem among sites with multiple, same-year samples taken by the same agency and resulting in different BCG proxy values. Without understanding program goals at a deeper level (we do not have access to this information), it is impossible choosing a representative annual sample for each of these sites. 

There are two graphics under `~/Preliminary_analysis_output/graphics` visualizing this issue. 

+ `multi_sample_sites_obs_year.png` shows the number of observations with at 2 or more samples a year
+ `multi_sample_sites_yr_score_range.png` shows the portion of multi-sampled sites with a difference in BCG proxy value

Our solution currently (5/22/24) is to remove same-year multi-sampled site observations taken by the same agency with a non-zero difference in BCG proxy. We also remove duplicates  for same-year multi-sampled site observations by the same agency that resulted in the same BCG proxy. This represents a small reduction in our overall data volume. 

The data loss is reported in `~/Preliminary_analysis_output/tables/multi-sample_summary_tbl.tex` We currently have a single representative BCG proxy at each site in each year for each agency. Conceptually, think of this as removing unidentifiable agency-specific data abnormalities at the annual level.

There are two clean data sets generated by `0.1_qc.R`: 

+ `/bcg_data_general.xlsx`: includes multi-sample sites data, with flags for multi-sampling by date
+ `/bcg_data_filtered.xlsx`: removed multi-sample sites with BCG proxy difference. Note that this dta set will not have the multi-sampling flags, but retains all other values.


**Data Dictionary**

*Please keep this section updated*

+ **Row_ID**, a unique observation identifier
+ **State**, the sampling agency 
+ **epa_site**, dummy variable = 1 if the site was sampled by the EPA in that year
+ **cp_site**, dummy variable = 1 if the site was sampled by Chesapeake Bay Program 
+ **SiteID**, unique identifier for the site as assigned by the sampling agency
+ **State_Site_ID**, unique identifier for the site by sampling agency
+ **Date**
+ **year** 
+ **month** 
+ **day**
+ **Lat**, latitude in degrees 
+ **Long**, longitude in degrees
+ **LatLong_ID**, unique site identifier by geolocation
+ **HUC8**, HUC 8 watershed id
+ **huc2**, HUC 2 Watershed id
+ **huc2name**, HUC2 regional name
+ **StreamLeve** 
+ **StreamOrde**
+ **COMID**, stream segment id
+ **NAME**
+ **GNIS_ID**, Geographic Names Information System id
+ **GNIS_NAME** 
+ **AREAACRES**, Area in acres 
+ **AREASQKM**, Area in sq km
+ **LENGTHKM**, length
+ **REACHCODE** 
+ **FTYPE** 
+ **FCODE** 
+ **count_sample**, number of times this location is sampled based on LatLong_ID
+ **count_agency**, number of agencies sampling this location based on LatLong_ID
+ **year_count**, number of times this site was sampled in this year
+ **mnth_count**, number of times this site was in this month of this year
+ **date_count**, number of times this site was sampled on this exact date
+ **multi_sample_yr**, dummy = 1 if this site was sampled more than once in this year
+ **multi_sample_mnth**, dummy = 1 if this site was sampled more than once in this month of this year
+ **multi_sample_date**, dummy = 1 if this site was sampled more than once on this exact date
+ **same_agency_multi_sample_yr**, dummy =1 if the same-year multi-sampling of this site was done by one agency
+ **same_agency_multi_sample_mnth**, dummy =1 if the same month-year multi-sampling of this site was done by one agency
+ **same_agency_multi_sample_date**, dummy =1 if the same date multi-sampling of this site was done by one agency
+ **Taxa** 
+ **BCG_proxy**, score generated through the conversion script


### `1.0_tables.R`

This file creates descriptive tables and intermeidate data sets from the cleaned data set `/bcg_data_filtered.xlsx`.  Resulting intermediate data sets are found under `~/Preliminary_analysis_output` and resulting latex tables are found under `~/Preliminary_analysis_output/tables`

1. Distribution/Quantile Descriptions: `/state_tbl.tex`, `/huc2_tble.tex`, and `/huc8_tbl.tex`

Measures of min, max, median, average, std for the variables below by state agency, HUC2 region, HUC8 region over years sampled

+ Count of unique sites
+ BCG proxy values

2. Site Intersections: `/BCG_avg_deviations.tex`, `/overlap_req.tex`, and `/site_intersections.xlsx`

Site cross over & proximity, measuring the sites that were evaluated by multiple state agencies in the same year and differences in their BCG proxies. There are 4 categories of overlap: 50 meters, 100 meters, 200 meters, isolated (the closest site is over 200 meters away).  A general note that this section of code requires a strong background in spatial packages. It is strongly recommended to run this code secton line by line or chunk by chunk. 

3. Site Volume: `/state_ste_vol_dist_tbl_tbl.tex`, `/ste_freq.xlsx`, and `/state_ste_vol.xlsx`

Annual sampling volume, min, 1st quartile, median, 3rd quartile, by state agency for the following

+ number of sites sampled
+ number of times each site was sampled

4. Average Annual BCG score for State, HUC2 & HUC8 regions: `/states_avg.xlsx`, `/huc2_avg.xlsx`, and `/huc8_avg.xlsx`

Note: this code takes a significant time to run on lower powered devices. For HUC8 regions, the averages were only calculated for year 2000 onward to save time and computing capacity. The resulting tables are in wide format and take values between [0,6], continuously. 


### `1.2_graphics.R`

This file creates three visual representations of in response to the questions below. All  output is housed under `~/Preliminary_analysis_output/graphics` or `~/Preliminary_analysis_output/videos`

1. `/annual_avg_bcg_frequntly_sampled.png`

Do most frequently sampled sites (by agency) have a temporal pattern related to ecological quality? Are only "poor" or "high value" sites sampled?

2. `/annual_bcg_by_huc8.mp4`

Where/when are the highest and lowest quality BCG Scores concentrated? Does this track with our intuition around the actual ecological conditions in those areas? 

Note: There is a possible code issue here. The images in this video were generated "manually" (i.e. the code for each was run separately) because the loop to generate them all concurrently was not functional. This is likely a memory issue as the same code works outside of the loop. Do not be surprised by an error code or blank images resulting from running the loop as it is on a low  powered device. 

3. `/annual_site_vol.mp4`

Is site volume concentrated among a specific agency or HUC2? Does this help us decide on (a) presentation of data in the survey or (b) provide confidence in our ability to project BCG proxies outside of sampled areas?




## Questions, Comments,  Bugs? 

Please contact the code developer, Virginia Callison at vwc28@cornell.edu for assistance. Read Me File - TBD
