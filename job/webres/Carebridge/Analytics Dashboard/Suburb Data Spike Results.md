## Findings

SA2s are not suitable as the primary suburb-style data type because they are statistical areas rather than suburb/locality records. While they can be similar to suburbs in some cases, they do not align closely enough with the way users are expected to enter preferred care locations.

SSCs are a better fit for suburb-style preferred locations and should be used for structured suburb records.

Postcodes and suburbs do not map cleanly to each other. A postcode can cover multiple suburbs, and a suburb can relate to multiple postcodes. Because of this, attempting to migrate postcodes into suburbs would add complexity and risk without producing reliable results.

The existing system already uses postcode-based location data, so the preferred location model should support both postcodes and SSCs rather than replacing postcodes entirely.

Both postcodes and SSCs can be linked to Aged Care Planning Regions independently using ABS correspondence data. This provides a cleaner model than trying to convert one location type into another.

The available ACPR correspondence data is slightly out of date, but it appears to be the best currently available source for mapping locations to Aged Care Planning Regions. More recent datasets were considered for the postcode and SSC data, but they do not map cleanly back to the available ACPR correspondence data. Attempting to combine newer postcode or SSC datasets with the older ACPR dataset proved impractical and would increase the risk of incorrect regional mappings.

Once more up-to-date ACPR data becomes available, the postcode and SSC reference data should be reviewed and updated together so that all location datasets remain aligned. Note that, from 2021 onwards, the ABS moved from using SSCs to Suburbs and Localities (SALs). This appears to be largely a change in terminology and should not materially alter the findings of this spike.

## Data Sources

- ACPR: [correspondence-of-2018-aged-care-planning-regions-and-2016-sa2s_0.xlsx](https://www.health.gov.au/resources/publications/correspondence-of-2018-aged-care-planning-regions-and-2016-sa2s?language=en)
- SCCs: [CG_SA2_2016_SSC_2016.xls](https://data.gov.au/data/dataset/asgs-geographic-correspondences-2016/resource/951e18c7-f187-4c86-a73f-fcabcd19af16)
- Postcodes: [ACPR-to-postcode-correspondence.xlsx](https://www.gen-agedcaredata.gov.au/getmedia/8e305cbe-dd3e-4a11-ab65-4e1a0d905495/ACPR-to-postcode-correspondence.xlsx)

## Recommendation

Use a structured location model that supports both postcode-based and SSC-based preferred locations.

Aged Care Planning Region should be resolved from the linked postcode or SSC reference record when needed for reporting.

The data model should not attempt to derive suburbs from postcodes, and the Application preferred location record should not store duplicated or derived region data.

Because the available ACPR correspondence data is tied to older reference datasets, the implementation should use matching source data for postcodes and SSCs rather than mixing in newer datasets that do not align cleanly.