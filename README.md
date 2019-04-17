# electricity_entity_boundaries
Nested boundaries for electricity entities (e.g., counties, utilities, balancing authority areas, NERC regions and subregions) including workflow processing for automated mapping.

## Contact
Casey Burleyson, PNNL

## Purpose
Historical data from utilities and other entities in the electric sector are a key resource for model formulation and calibration in the multisector dynamics community. For example, observed hourly electricity loads can be used to evaluate models of simulated demand in response to heat waves and other environmental stressors. However, despite its utility (pun intended), there are several key challenges to maximizing the usefulness of these data. The first of which is a spatial mismatch between the data reported by individual utilities and the output produced by models. Utilities serve regions that are heterogenous in both size and shape. Mapping the data that they produce to compare with output from models that are either on regular grids or are nodal in nature is nontrivial. The second challenge is that utilities are constantly being bought and sold or changing names, making it hard to track the evolution of their reported loads across years.

This workflow goes through the steps of mapping utilities, balancing authorities (BAs), and NERC regions to counties in the U.S. Counties serve as a useful base spatial scale because they can be aggregated or disaggregated easily using populations as weights in the scaling and because other datasets that are useful in multisector dynamics (e.g., population) are often available at the county scale. Other teams, including the team responsible for this workflow, have completed this mapping in the past, but their efforts were not documented and their process was not repeatable. Previous versions of this mappings relied on assumptions and other subjective decisions that were not communicated or well understood. The purpose of this workflow is to document step-by-step the process used to create a utility-to-BA-to-NERC region-to-county mapping.

The base resource for this mapping is the EIA-861 annual report on the electric power industry. Within this dataset are a series of files in which all utilities in the U.S. report the BA which they operate under, their NERC region, and counties and states where they operate. The technical challenge is that these mappings are reported in individual data files. The bulk of the workflow involves merging these individual files to create a complete mapping. As the EIA-861 data is reported annually, this workflow should be repeatable on subsequent years of data assuming that the same base information is included in all future versions.

## Input Data
1. Annual Electric Power Industry Report: Form EIA-861 Detailed Data Files
    * Source: [https://www.eia.gov/electricity/data/eia861/](https://www.eia.gov/electricity/data/eia861/)
    * Purpose: Contains all of the detailed files described below
Accessed: 9-April 2019

2. Sales to Ultimate Customers
    * Source: Sales_Ult_Cust_2017.xlsx from the EIA-861 zip file
    * Purpose: Maps utilities to balancing authorities (BAs) and provides the total sales for each utility that are used as a tie-breaker when more than there is more than one BA or NERC region per county.

3. Utility Data
    * Source: Utility_Data_2017.xlsx from the EIA-861 zip file
    * Purpose: Maps utilities to NERC regions.

4. County Metadata
    * Source: This spreadsheet (County_Metadata.xlsx) was made in house and gives the FIPS code, county name, state information, population-weighted latitude, population-weighted longitude, area, and total population estimated by the census bureau in 2017 for all counties in the United States.
    * Purpose: Gives basic county information needed for mapping and scaling.

5. Service Territory
    * Source: Service_Territory_2017.xlsx from the EIA-861 zip file
    * Purpose: Maps utilities to the states and counties that they operate in.

## Processing Steps
1.	Download and uncompress the EIA-861 zip file.

2.	Convert the Sales to Ultimate Customers spreadsheet into .mat file.
Scripts:
  -	Read_Sales_Ult_Cust_2017_Spreadsheet.m
Required Functions:
  -	BA_Information_From_BA_Short_Name.m
  -	State_FIPS_From_State_Abbreviations.m

3.	Convert the Utility Data spreadsheet into a .mat file.
Scripts:
  -	Read_Utility_Data_2017_Spreadsheet.m
Required Functions:
  -	NERC_Region_Information_From_NERC_Region_Short_Name.m

4.	Convert the County Metadata spreadsheet into a .mat file.
Scripts:
  -	Read_County_Metadata_Spreadsheet.m
Required Functions:
  -	State_Information_From_State_FIPS.m

5.	Convert the Service Territory spreadsheet into a .mat file and match the listed counties to counties from the County Metadata dataset.
Scripts:
  -	Read_Service_Territory_2017_Spreadsheet.m
Required Functions:
  -	State_FIPS_From_State_Abbreviations.m

6.	Merge all of the above datasets by mapping utilities to BAs to NERC regions and eventually to counties in the U.S. The output data file is a structure with each row being a county and then embedded in the structure for that row is all of the utilities operating in that county and their associated BA and NERC region. In counties with more than one BA or NERC region listed, which happens quite often, the county is assigned to the utility with the highest total sales of electricity in 2017. The full output is given in a Matlab file (Utility_to_BA_to_NERC_Region_to_County_Mapping.mat) and a summary table containing the largest BA and NERC region in each county is given as an Excel file (Utility_to_BA_to_NERC_Region_to_County_Mapping_Summary_File.xlsx).
Scripts:
  -	Process_Utility_to_BA_to_NERC_Region_to_County_Mapping.m

7.	Create and save maps of counties in the CONUS with their number of utilities, number of BAs, primary BA, number of NERC regions, and primary NERC region.
Scripts:
  -	Plot_Utility_to_BA_to_NERC_Region_to_County_Mapping.m
Required Functions:
  -	BA_Information_From_BA_Code.m
  -	NERC_Region_Information_From_NERC_Region_Code.m
