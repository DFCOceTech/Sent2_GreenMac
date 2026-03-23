Sentinel 2 workflow using CDSE OData Python 

Updated 23 March 2026

1. Find features by tile ID and date
2. Iterate through features to check for L1C processing level and < 30% cloud cover. Check for early features with N0500 to remove duplicates
3. Identify features that meet above criteria and download S2 L1C
4. Mask out land, fjords, skerries, and shallow, near-shore waters
