![Sentinel-2 RGB Image of Islands in South Greenland](images/SGreenland_RGB.png)

Sentinel 2 workflow using CDSE OData Python 

Updated 23 March 2026

1. Find features by tile ID and date
2. Iterate through features to check for L1C processing level and < 30% cloud cover. 
3. Identify features that meet above criteria and download S2 L1C
4. Mask out land, fjords, skerries, and shallow, near-shore waters
