## Datasets
### Types:
  -  Image/ImageCollection (raster): 
  ```
    var image = ee.Image('users/yourfolder/yourimage');
  ```
  -  Table (feature, vector): 
  ```
    var fc = ee.FeatureCollection('TIGER/2016/Roads');
  ```
### User own assets:
  -  upload: Assets --> NEW to upload raster or vector dataset
  -  use: same to system data

### Several important datasets
#### Landsat
```
ee.ImageCollection('LANDSAT/LC08/C01/T1')     # Landsat 8, Collection 1, Tier 1 only (meet quality requirement), raw DN values
ee.ImageCollection('LANDSAT/LC08/C01/T1_RT')  # Landsat 8, Collection 1, Tier 1 + real time
ee.ImageCollection('LANDSAT/LC08/C01/T2')     # Landsat 8, Collection 1, Tier 2 only (doesn't meet quality requirement)
ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA') # TOA data collection
ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')  # Surface reflectance data collection (need to scale by 0.0001)
ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20140318')  # single image
```
#### MODIS
```
ee.Image('MODIS/051/MCD12Q1/2012_01_01').select('Land_Cover_Type_1')
ee.Image('MODIS/006/MCD64A1/2015_09_01').select('BurnDate')
ee.ImageCollection('FIRMS').filter(ee.Filter.date('2019-08-01', '2019-08-31')).select('T21').max()
```
#### Global Forest Change (Hansen et al)
```
ee.Image('UMD/hansen/global_forest_change_2015')
```
#### Sentinel 2
```
ee.ImageCollection("COPERNICUS/S2")
```
#### Others  
VIIRS, DMSP-OLS, GRACE, ...
