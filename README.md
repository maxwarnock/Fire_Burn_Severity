# Cameron Peak and East Troublesome Fire Burn Severity

## Overview

This project maps burn severity for the 2020 Cameron Peak and East Troublesome fires, Colorado's two largest wildfires, using HLS satellite imagery. It answers three questions:

1. Where did the fires burn most severely, by watershed?
2. How has vegetation recovered from 2021 to 2026?
3. How much of the high severity area is in or near homes in the wildland urban interface (WUI)?

A short write-up of the project and its results is in the [project blog](code/blog.ipynb).

## Study Area

- Cameron Peak Fire: started August 13, 2020, west of Fort Collins, and burned 208,913 acres
- East Troublesome Fire: started October 14, 2020, in Grand County, and burned 193,694 acres

The study area for each fire is its NIFC perimeter plus a 5km buffer. All rasters are on a 30m grid in UTM zone 13N that lines up with the HLS pixels, except the WUI map, which keeps its 250m grid.

## Data Sources

- Fire perimeters: [National Interagency Fire Center (NIFC), InterAgency Fire Perimeter History](https://data-nifc.opendata.arcgis.com/)
- Burn severity: [Monitoring Trends in Burn Severity (MTBS)](https://www.mtbs.gov/) burned area boundaries and 2020 burn severity maps
- Watersheds: [USGS Watershed Boundary Dataset](https://www.usgs.gov/national-hydrography/watershed-boundary-dataset), HUC12 subwatersheds
- Wildland urban interface: 2019 WUI map for Colorado from my [WUI Mapping Tool](https://github.com/maxwarnock/WUI_Mapping_Tool). A clip around the fires is saved in data/wui
- Satellite imagery: [NASA Harmonized Landsat and Sentinel-2 (HLS) v2.0](https://hls.gsfc.nasa.gov/), which combines Landsat 8/9 and Sentinel-2 into one 30m dataset

## Notebooks

### [01 Data Download](code/01_data_download.ipynb)
Downloads the fire perimeters, the MTBS records and burn severity maps, and the HUC12 subwatersheds, and loads the WUI map. Then it builds a summer median NBR and NDVI composite from HLS for every year from 2019 to 2026, using the clear images from June 1 to September 30 (Howe et al., 2022; Parks et al., 2018). Clouds, cloud shadows, snow, and water are masked with the HLS quality band (Ju et al., 2026).

### [02 Burn Severity](code/02_burn_severity.ipynb)
Calculates dNBR, the drop in NBR from the summer before the fires (2019) to the summer after (2021), and splits it into the six Key and Benson (2006) classes: Enhanced regrowth, Unburned, Low, Moderate-low, Moderate-high, and High. No dNBR offset is applied. The classes are mapped and the area in each class is counted inside the fire perimeters.

### [03 MTBS Comparison](code/03_mtbs_comparison.ipynb)
Compares the Key and Benson map with the official MTBS maps (Eidenshink et al., 2007) in four shared classes: Unburned, Low, Moderate, and High. It plots the HLS dNBR values in each MTBS class against the Key and Benson thresholds, maps where the two maps differ, and measures how often they agree with an error matrix, percent agreement, and kappa (Cohen, 1960; Congalton, 1991; Landis & Koch, 1977). MTBS thresholds are picked by an analyst for each fire (Kolden et al., 2015), so agreeing with MTBS isn't the same as being accurate.

### [04 Watershed Severity](code/04_watershed_severity.ipynb)
Finds the area of each burn severity class in every HUC12 subwatershed the fires burned, counting only the part inside the perimeters, and ranks the subwatersheds by the percent of their total area that burned at high severity. The interactive map of the results is saved as a web page, which the blog shows.

### [05 Vegetation Recovery](code/05_vegetation_recovery.ipynb)
Tracks the mean summer NDVI (Tucker, 1979) of each burned class from 2019 to 2026, compared with unburned land more than 1km outside the fires, and measures how much of the NDVI lost in the fires has come back. 2020 is left out, because Cameron Peak burned in the middle of that summer.

### [06 WUI Severity](code/06_wui_severity.ipynb)
Compares burn severity in and outside the WUI (Radeloff et al., 2005), and finds how much of the high severity area was in the WUI or within 2.4km of it, about how far embers can fly ahead of a fire. Burn severity measures the vegetation, not damage to buildings, so this shows where severe fire was close to homes, not which homes burned.

### [Blog](code/blog.ipynb)
A short write-up of the project, with figures from notebooks 02, 04, 05, and 06.

## Results

- Burn severity: 25% of Cameron Peak (52,930 acres) and 18% of East Troublesome (35,738 acres) burned at high severity
- MTBS comparison: the two maps agree on 68-71% of the compared area (kappa 0.54-0.61). The Key and Benson map has less high severity than MTBS, but 92-96% of what it calls high is also high in MTBS, so its high severity areas are a conservative estimate
- Watersheds: seven subwatersheds had 20% or more of their area burn at high severity. The most were Little Beaver Creek (40%) and Miller Fork (37%) at Cameron Peak, and Headwaters Willow Creek (32%) at East Troublesome
- Vegetation recovery: by 2026, the moderate-high and high severity areas had regained about 40-45% of the NDVI they lost
- WUI: less than 1% of the high severity area was in the WUI, but 28% at Cameron Peak and 14% at East Troublesome was within 2.4km of it

## Outputs

| Folder | Contents |
|---|---|
| data/perimeters, data/mtbs, data/watersheds, data/wui, data/hls | Data from 01_data_download |
| data/severity | dNBR and burn severity class rasters, and the area in each class (02) |
| data/comparison | MTBS agreement tables (03) |
| data/watershed_severity | High severity by subwatershed, as a table and GeoPackage, and the interactive map used in the blog (04) |
| data/recovery | NDVI and recovery by class and year (05) |
| data/wui_severity | Burn severity in and near the WUI (06) |
| code/blog_figures | The blog's header image, and figures copied from notebooks 02, 04, 05, and 06 |

## Reproducibility

1. Create and activate the conda environment from the repository folder:

```
conda env create -f environment.yml
conda activate fire_severity
```

2. Run the notebooks in order, 01 to 06, then the blog.

- 01_data_download needs a free [NASA Earthdata account](https://urs.earthdata.nasa.gov/) for the HLS data
- The downloaded data is saved in this repo. 01_data_download loads data that is already saved instead of downloading it again, so it can be rerun when a download server is down. To download something again, delete its file in the data folder
- HLS composites that are already saved are skipped, so the HLS step can be stopped and restarted
- The WUI map isn't downloaded. It's a clip of the WUI Mapping Tool output saved in data/wui
- All file paths are relative to the project root
- The blog's subwatershed map is the web page saved by 04_watershed_severity, so after changing that map, rerun 04 and then the blog

To export the blog as a web page without the code:

```
jupyter nbconvert --to html --no-input code/blog.ipynb
```

## Works Cited

Cohen, J. (1960). A coefficient of agreement for nominal scales. *Educational and Psychological Measurement, 20*(1), 37-46. https://doi.org/10.1177/001316446002000104

Congalton, R. G. (1991). A review of assessing the accuracy of classifications of remotely sensed data. *Remote Sensing of Environment, 37*(1), 35-46. https://doi.org/10.1016/0034-4257(91)90048-B

Eidenshink, J., Schwind, B., Brewer, K., Zhu, Z.-L., Quayle, B., & Howard, S. (2007). A project for monitoring trends in burn severity. *Fire Ecology, 3*(1), 3-21. https://doi.org/10.4996/fireecology.0301003

Howe, A. A., Parks, S. A., Harvey, B. J., Saberi, S. J., Lutz, J. A., & Yocom, L. L. (2022). Comparing Sentinel-2 and Landsat 8 for burn severity mapping in western North America. *Remote Sensing, 14*(20), 5249. https://doi.org/10.3390/rs14205249

Ju, J., Neigh, C., Sridhar, M., Claverie, M., Skakun, S., Roger, J.-C., Vermote, E., Dungan, J., & Ashokkumar, L. (2026). *Harmonized Landsat and Sentinel-2 (HLS) Product User Guide, Product Version 2.0*. NASA Land Processes Distributed Active Archive Center. https://lpdaac.usgs.gov/documents/1698/HLS_User_Guide_V2.pdf

Key, C. H., & Benson, N. C. (2006). Landscape assessment (LA): Sampling and analysis methods. In D. C. Lutes et al. (Eds.), *FIREMON: Fire effects monitoring and inventory system* (Gen. Tech. Rep. RMRS-GTR-164-CD, pp. LA-1-LA-55). USDA Forest Service, Rocky Mountain Research Station.

Kolden, C. A., Smith, A. M. S., & Abatzoglou, J. T. (2015). Limitations and utilisation of Monitoring Trends in Burn Severity products for assessing wildfire severity in the USA. *International Journal of Wildland Fire, 24*(7), 1023-1028. https://doi.org/10.1071/WF15082

Landis, J. R., & Koch, G. G. (1977). The measurement of observer agreement for categorical data. *Biometrics, 33*(1), 159-174. https://doi.org/10.2307/2529310

Parks, S. A., Holsinger, L. M., Voss, M. A., Loehman, R. A., & Robinson, N. P. (2018). Mean composite fire severity metrics computed with Google Earth Engine offer improved accuracy and expanded mapping potential. *Remote Sensing, 10*(6), 879. https://doi.org/10.3390/rs10060879

Radeloff, V. C., Hammer, R. B., Stewart, S. I., Fried, J. S., Holcomb, S. S., & McKeefry, J. F. (2005). The wildland-urban interface in the United States. *Ecological Applications, 15*(3), 799-805. https://doi.org/10.1890/04-1413

Tucker, C. J. (1979). Red and photographic infrared linear combinations for monitoring vegetation. *Remote Sensing of Environment, 8*(2), 127-150. https://doi.org/10.1016/0034-4257(79)90013-0

Warnock, M. (2026). *WUI Mapping Tool* (Version 1.0.0) [Computer software]. Zenodo. https://doi.org/10.5281/zenodo.23404697
