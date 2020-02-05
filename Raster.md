## Raster data: image and image collection

### Get images

#### Get one Landsat 8 imagery
```
var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20140318');
```

#### Generate a constant image from metadata
```
var image_tstart = image.metadata('system:time_start');
```

#### Get information of this imagery
```
print(image);
print(image.bandNames());
print(image.get('ELEVATION_SOURCE'));
```

#### Get thumbnail of the image
```
// Fetch a digital elevation model.
var image = ee.Image('CGIAR/SRTM90_V4');

// Request a default thumbnail of the DEM with defined linear stretch.
// Set masked pixels (ocean) to 1000 so they map as gray.
var thumbnail1 = image.unmask(1000).getThumbURL({
  'min': 0,
  'max': 3000
});
print('Default extent and size:', thumbnail1);

// Specify region by GeoJSON, define palette, set size of the larger aspect dimension.
var thumbnail2 = image.getThumbURL({
  'min': 0,
  'max': 3000,
  'palette': ['00A600','63C600','E6E600','E9BD3A','ECB176','EFC2B3','F2F2F2'],
  'dimensions': 500,
  'region': ee.Geometry.Rectangle([-84.6, -55.9, -32.9, 15.7]),
});
print('GeoJSON region, palette, and max dimension:', thumbnail2);

// Specify region by list of points and set display CRS as Web Mercator.
var thumbnail3 = image.getThumbURL({
  'min': 0,
  'max': 3000,
  'palette': ['00A600','63C600','E6E600','E9BD3A','ECB176','EFC2B3','F2F2F2'],
  'region': [[-84.6, 15.7], [-84.6, -55.9], [-32.9, -55.9]],
  'crs': 'EPSG:3857'
});
print('Linear ring region and specified crs', thumbnail3);
```

#### Get the red, green, and blue bands from the imagery
```
var imgred = image.select('B4');
var imggreen = image.select('B3');
var imgblue = image.select('B2');
```

### Show images on map

#### Show single band image
```
Map.addLayer(imgred,
            {palette:['000000','FF0000'], max: 0.2, min: 0},
            'red');
Map.addLayer(imggreen,
            {palette:['000000','00FF00'], max: 0.2, min: 0},
            'green');
Map.addLayer(imgblue,
            {palette:['000000','0000FF'], max: 0.2, min: 0},
            'blue');
Map.setCenter(-122.1899, 37.5010, 9);
Map.setOptions('satellite');
```

#### Show the rgb image
```
Map.addLayer(image,
            {bands: ['B4','B3','B2'], min: 0, max: 0.3, gamma: [0.95, 1.1, 1]},
            'rgb');
Map.addLayer(image,
            {bands: ['B5','B4','B3'], min: 0, max: 0.3, gamma: [0.95, 1.1, 1]},
            'veg');
```



### Image operations

#### Mathematics
```
var res=image.select('B4')
        .subtract(image.select('B3'))
        .divide(100)
        .add(image.select('B2'));
Map.addLayer(res,{min:0,max:0.2},'res');
```
#### expression
```
var evi = image.expression(
    '2.5 * ((NIR - RED) / (NIR + 6 * RED - 7.5 * BLUE + 1))', {
      'NIR': image.select('B5'),
      'RED': image.select('B4'),
      'BLUE': image.select('B2')
});
Map.addLayer(evi,{min:0,max:1},'evi');        
```
#### normalizedDifference
```
var ndvi = image.normalizedDifference(['B5', 'B4']);
Map.addLayer(ndvi,{min:0,max:1},'ndvi'); 
```
#### Reducing an imagecollection   
This function uses a conditional statement to return the image if the solar elevation > 40 degrees.  Otherwise it returns a zero image.
```
var conditional = function(image) {
  return ee.Algorithms.If(ee.Number(image.get('SUN_ELEVATION')).gt(50),
                          image,
                          ee.Image(0));
};
print('Expand this to see the result: ', imagefiltered.map(conditional));
```

#### Mapping over an imagecollection, or an image
```
var median = imagefiltered.median();
var median = imagefiltered.reduce(ee.Reducer.median());

var region = ee.Geometry.Point(-122.1899, 37.5010).buffer(20000);
Map.addLayer(region,{color:'orange'},'region');
var regmean = image.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: region,
  scale: 30
});
print(regmean);
```
#### Mask
```
print(imgred);
var highmask = imgred.gt(0.1);
var highred = highmask.updateMask(highmask);
Map.addLayer(highred, {palette: 'red'}, 'highred');
```
#### Clip
```
var regred = imgred.clip(region);
print(regred);
Map.addLayer(regred, {palette:['white','blue'],min:0,max:0.2} ,'regred');
```

#### Replace using where
```
// Load a cloudy Landsat 8 image.
var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20130603');
// Load another image to replace the cloudy pixels.
var replacement = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20130416');
// Compute a cloud score band.
var cloud = ee.Algorithms.Landsat.simpleCloudScore(image).select('cloud');
// Set cloudy pixels to the other image.
var replaced = image.where(cloud.gt(10), replacement);    
```

#### Image visualize
```
// Use the image.visualize() method to convert an image into an 8-bit RGB image for display or export. 
// Load an image.
var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20140318');

// Create an NDWI image, define visualization parameters and display.
var ndwi = image.normalizedDifference(['B3', 'B5']);
    
// Mask the non-watery parts of the image, where NDWI < 0.4.
var ndwiMasked = ndwi.updateMask(ndwi.gte(0.4));

// Create visualization layers.
var imageRGB = image.visualize({bands: ['B5', 'B4', 'B3'], max: 0.5});
var ndwiRGB = ndwiMasked.visualize({
  min: 0.5,
  max: 1,
  palette: ['00FFFF', '0000FF']
});
        
```

#### Mosaicking
```
// Mosaic the visualization layers and display (or export).
var mosaic = ee.ImageCollection([imageRGB, ndwiRGB]).mosaic();
Map.addLayer(mosaic, {}, 'mosaic');
```

#### Styled layer descriptions (SLD)  
You can use a [Styled Layer Descriptor (SLD)](https://developers.google.com/earth-engine/image_visualization#styled-layer-descriptors) to render imagery for display.
```
var cover = ee.Image('MODIS/051/MCD12Q1/2012_01_01').select('Land_Cover_Type_1');

// Define an SLD style of discrete intervals to apply to the image.
var sld_intervals =
'<RasterSymbolizer>' +
 ' <ColorMap  type="intervals" extended="false" >' +
    '<ColorMapEntry color="#aec3d4" quantity="0" label="Water"/>' +
    '<ColorMapEntry color="#152106" quantity="1" label="Evergreen Needleleaf Forest"/>' +
    '<ColorMapEntry color="#225129" quantity="2" label="Evergreen Broadleaf Forest"/>' +
    '<ColorMapEntry color="#369b47" quantity="3" label="Deciduous Needleleaf Forest"/>' +
    '<ColorMapEntry color="#30eb5b" quantity="4" label="Deciduous Broadleaf Forest"/>' +
    '<ColorMapEntry color="#387242" quantity="5" label="Mixed Deciduous Forest"/>' +
    '<ColorMapEntry color="#6a2325" quantity="6" label="Closed Shrubland"/>' +
    '<ColorMapEntry color="#c3aa69" quantity="7" label="Open Shrubland"/>' +
    '<ColorMapEntry color="#b76031" quantity="8" label="Woody Savanna"/>' +
    '<ColorMapEntry color="#d9903d" quantity="9" label="Savanna"/>' +
    '<ColorMapEntry color="#91af40" quantity="10" label="Grassland"/>' +
    '<ColorMapEntry color="#111149" quantity="11" label="Permanent Wetland"/>' +
    '<ColorMapEntry color="#cdb33b" quantity="12" label="Cropland"/>' +
    '<ColorMapEntry color="#cc0013" quantity="13" label="Urban"/>' +
    '<ColorMapEntry color="#33280d" quantity="14" label="Crop, Natural Veg. Mosaic"/>' +
    '<ColorMapEntry color="#d7cdcc" quantity="15" label="Permanent Snow, Ice"/>' +
    '<ColorMapEntry color="#f7e084" quantity="16" label="Barren, Desert"/>' +
    '<ColorMapEntry color="#6f6f6f" quantity="17" label="Tundra"/>' +
  '</ColorMap>' +
'</RasterSymbolizer>';
Map.addLayer(cover.sldStyle(sld_intervals), {}, 'IGBP classification styled');
```

#### Convolution  
Each pixel of the image output by convolve() is the linear combination of the kernel values and the input image pixels covered by the kernel. 
```
// Load and display an image.
var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20140318');
Map.setCenter(-121.9785, 37.8694, 11);
Map.addLayer(image, {bands: ['B5', 'B4', 'B3'], max: 0.5}, 'input image');

// Define a boxcar or low-pass kernel.
var boxcar = ee.Kernel.square({
  radius: 7, units: 'pixels', normalize: true
});

// Smooth the image by convolving with the boxcar kernel.
var smooth = image.convolve(boxcar);
Map.addLayer(smooth, {bands: ['B5', 'B4', 'B3'], max: 0.5}, 'smoothed');
```  
There are many kernels available. Here list some important ones:
- ee.Kernel.circle()
- ee.Kernel.square()
- ee.Kernel.fixed()
- ee.Kernel.gaussian()
- ee.Kernel.laplacian8()


#### Morphological Operations
```
// Load a Landsat 8 image, select the NIR band, threshold, display.
var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20140318')
            .select(4).gt(0.2);
Map.setCenter(-122.1899, 37.5010, 13);
Map.addLayer(image, {}, 'NIR threshold');

// Define a kernel.
var kernel = ee.Kernel.circle({radius: 1});

// Perform an erosion followed by a dilation, display.
var opened = image
             .focal_min({kernel: kernel, iterations: 2})
             .focal_max({kernel: kernel, iterations: 2});
Map.addLayer(opened, {}, 'opened');
```

#### Gradients
```
// Load a Landsat 8 image and select the panchromatic band.
var image = ee.Image('LANDSAT/LC08/C01/T1/LC08_044034_20140318').select('B8');

// Compute the image gradient in the X and Y directions.
var xyGrad = image.gradient();

// Compute the magnitude of the gradient.
var gradient = xyGrad.select('x').pow(2)
          .add(xyGrad.select('y').pow(2)).sqrt();

// Compute the direction of the gradient.
var direction = xyGrad.select('y').atan2(xyGrad.select('x'));

// Display the results.
Map.setCenter(-122.054, 37.7295, 10);
Map.addLayer(direction, {min: -2, max: 2, format: 'png'}, 'direction');
Map.addLayer(gradient, {min: -7, max: 7, format: 'png'}, 'gradient');
    
```

#### Edge detection  
Canny algorithm
```
// Load a Landsat 8 image, select the panchromatic band.
var image = ee.Image('LANDSAT/LC08/C01/T1/LC08_044034_20140318').select('B8');

// Perform Canny edge detection and display the result.
var canny = ee.Algorithms.CannyEdgeDetector({
  image: image, threshold: 10, sigma: 1
});
Map.setCenter(-122.054, 37.7295, 10);
Map.addLayer(canny, {}, 'canny');
    
```

Zero crossing algorithm
```
// Load a Landsat 8 image, select the panchromatic band.
var image = ee.Image('LANDSAT/LC08/C01/T1/LC08_044034_20140318').select('B8');
Map.addLayer(image, {max: 12000});

// Define a "fat" Gaussian kernel.
var fat = ee.Kernel.gaussian({
  radius: 3,
  sigma: 3,
  units: 'pixels',
  normalize: true,
  magnitude: -1
});

// Define a "skinny" Gaussian kernel.
var skinny = ee.Kernel.gaussian({
  radius: 3,
  sigma: 1,
  units: 'pixels',
  normalize: true,
});

// Compute a difference-of-Gaussians (DOG) kernel.
var dog = fat.add(skinny);

// Compute the zero crossings of the second derivative, display.
var zeroXings = image.convolve(dog).zeroCrossing();
Map.setCenter(-122.054, 37.7295, 10);
Map.addLayer(zeroXings.updateMask(zeroXings), {palette: 'FF0000'}, 'zero crossings');
    
```

#### Texture
```
// Compute entrapy
// Load a high-resolution NAIP image.
var image = ee.Image('USDA/NAIP/DOQQ/m_3712213_sw_10_1_20140613');
// Get the NIR band.
var nir = image.select('N');
// Define a neighborhood with a kernel.
var square = ee.Kernel.square({radius: 4});
// Compute entropy and display.
var entropy = nir.entropy(square);
```  
Other related functions related to texture include __glcmTexture__, __neighborhoodToBands__, etc.

#### Object-based methods
Image objects are sets of connected pixels having the same integer value. Categorical, binned, and boolean image data are suitable for object analysis.
```
// Set up image
var point = ee.Geometry.Point(-122.1899, 37.5010);
var aoi = point.buffer(10000);
var kelvin = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_044034_20140318')
  .select(['B10'], ['kelvin'])
  .clip(aoi);
var hotspots = kelvin.gt(303)
  .selfMask()
  .rename('hotspots');
  
// Uniquely label the hotspot image objects using connectedComponents
var objectId = hotspots.connectedComponents({
  connectedness: ee.Kernel.plus(1),
  maxSize: 128
Map.addLayer(objectId.randomVisualizer(), null, 'Objects');  

// Compute the number of pixels in each object defined by the "labels" band using connectedPixelCount
var objectSize = objectId.select('labels')
  .connectedPixelCount({
    maxSize: 128, eightConnected: false
  });
Map.addLayer(objectSize, null, 'Object n pixels');

// Multiply pixel area by the number of pixels in an object to calculate the object area.
var pixelArea = ee.Image.pixelArea();
var objectArea = objectSize.multiply(pixelArea);
Map.addLayer(objectArea, null, 'Object area m^2');

// Do zonal statistics using reduceConnectedComponents
kelvin = kelvin.addBands(objectId.select('labels'));
var patchTemp = kelvin.reduceConnectedComponents({
  reducer: ee.Reducer.mean(),
  labelBand: 'labels'
});
Map.addLayer(
  patchTemp,
  {min: 303, max: 304, palette: ['yellow', 'red']},
  'Mean temperature'
);
```

### Image collection

#### Get images from an image collection
```
var imagecollection = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA');

var imagefiltered = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA')
                    .filterBounds(ee.Geometry.Point(-122.262, 37.8719))
                    .filterDate('2014-06-01', '2014-10-01')
                    .filter(ee.Filter.lt('CLOUD_COVER',50));
print(imagefiltered);
var imagefirst = ee.Image(
                    imagefiltered.sort('CLOUD_COVER')
                    .first());
Map.addLayer(imagefirst,
            {bands: ['B4','B3','B2'], min: 0, max: 0.3, gamma: [0.95, 1.1, 1]},
            'rgb');  
```

#### Get the number of images in a collection
```
var count = imagefiltered.size();
print('Count: ', count);
```

#### Get the date range of images in the collection.
```
var range = collection.reduceColumns(ee.Reducer.minMax(), ["system:time_start"])
print('Date range: ', ee.Date(range.get('min')), ee.Date(range.get('max')))
```

#### Get statistics for a property of the images in the collection.
```
var sunStats = collection.aggregate_stats('SUN_ELEVATION');
print('Sun elevation statistics: ', sunStats);
```

#### Sort
```
// sort by date
var s2col = ee.ImageCollection('COPERNICUS/S2_SR')
  .filterBounds(ee.Geometry.Point(-122.1, 37.2))
  .sort('system:time_start');
// sort by cloudiness
var s2col = ee.ImageCollection('COPERNICUS/S2_SR')
  .filterBounds(ee.Geometry.Point(-122.1, 37.2))
  .sort('CLOUDY_PIXEL_PERCENTAGE'); 
```

#### Limit the collection to the 10 most recent images.
```
var recent = collection.sort('system:time_start', false).limit(10);
print('Recent images: ', recent);
```

#### Calculate linear regression (Reducer.linearFit)
```
// This function adds a band representing the image timestamp.
var addTime = function(image) {
  return image.addBands(image.metadata('system:time_start')
    .divide(1000 * 60 * 60 * 24 * 365));
};

// Load a MODIS collection, filter to several years of 16 day mosaics,
// and map the time band function over it.
var collection = ee.ImageCollection('MODIS/006/MYD13A1')
  .filterDate('2004-01-01', '2010-10-31')
  .map(addTime);

// Select the bands to model with the independent variable first.
var trend = collection.select(['system:time_start', 'EVI'])
  // Compute the linear trend over time.
  .reduce(ee.Reducer.linearFit());

// Display the trend with increasing slopes in green, decreasing in red.
Map.setCenter(-96.943, 39.436, 5);
Map.addLayer(
    trend,
    {min: 0, max: [-100, 100, 10000], bands: ['scale', 'scale', 'offset']},
    'EVI trend');
    
```
For a more flexible approach to linear modelling, use one of the linear regression reducers which allow for a variable number of independent and dependent variables. Specifically, ee.Reducer.linearRegression() implements ordinary least squares regression (OLS). Alternatively, robustLinearRegression() uses a cost function based on regression residuals to iteratively de-weight outliers in the data.


#### Iteration over different images
Sometimes the map function is not enough, for example, when cumulative values (over time scale) are needed. Use ImageCollection.iterate funtion can loop over different images. [Tutorial](https://developers.google.com/earth-engine/ic_iterating)
```
var series = collection.filterDate('2011-01-01', '2014-12-31').map(function(image) {
    return image.subtract(mean).set('system:time_start', image.get('system:time_start'));
});
var accumulate = function(image, list) {
  var previous = ee.Image(ee.List(list).get(-1));
  var added = image.add(previous)
    .set('system:time_start', image.get('system:time_start'));
  return ee.List(list).add(added);
};
var cumulative = ee.ImageCollection(ee.List(series.iterate(accumulate, first)));
```

#### Image visualization using visualize and map
```
var s2col = ee.ImageCollection('COPERNICUS/S2_SR')
  .filterBounds(ee.Geometry.Point(-96.9037, 48.0395))
  .filterDate('2019-06-01', '2019-10-01');
var visArgs = {bands: ['B11', 'B8', 'B3'], min: 300, max: 3500};
var visFun = function(img) {
  return img.visualize(visArgs).copyProperties(img, img.propertyNames());
};
var s2colVis = s2col.map(visFun);
```

#### Image animation using getVideoThumbURL
```
var aoi = ee.Geometry.Polygon(
  [[[-179.0, 78.0], [-179.0, -58.0], [179.0, -58.0], [179.0, 78.0]]], null,
  false);
var tempCol = ee.ImageCollection('NOAA/GFS0P25')
  .filterDate('2018-12-22', '2018-12-23')
  .limit(24)
  .select('temperature_2m_above_ground');
var videoArgs = {
  dimensions: 768,
  region: aoi,
  framesPerSecond: 7,
  crs: 'EPSG:3857',
  min: -40.0,
  max: 35.0,
  palette: ['blue', 'purple', 'cyan', 'green', 'yellow', 'red']
};
print(ui.Thumbnail(tempCol, videoArgs));
// Alternatively, print a URL that will produce the animation when accessed.
print(tempCol.getVideoThumbURL(videoArgs));
```

#### Image filmstrip
```
// Define arguments for the getFilmstripThumbURL function parameters.
var filmArgs = {
  dimensions: 128,
  region: aoi,
  crs: 'EPSG:3857',
  min: -40.0,
  max: 35.0,
  palette: ['blue', 'purple', 'cyan', 'green', 'yellow', 'red']
};

// Print a URL that will produce the filmstrip when accessed.
print(tempCol.getFilmstripThumbURL(filmArgs));
```

#### Advanced image collection visualization
More techniques related to visualization, including overlays (image.blend) and transitions, are available [here](https://developers.google.com/earth-engine/ic_visualization#advanced_techniques)

### Reduce

#### Basic usage  
The ee.Reducer class specifies how data is aggregated. The reducers in this class can specify a simple statistic to use for the aggregation (e.g. minimum, maximum, mean, median, standard deviation, etc.), or a more complex summary of the input data (e.g. histogram, linear regression, list). Reductions may occur over time (imageCollection.reduce()), space (image.reduceRegion(), image.reduceNeighborhood()), bands (image.reduce()), or the attribute space of a FeatureCollection (featureCollection.reduceColumns() or FeatureCollection methods that start with aggregate_)
```
var extrema = collection.reduce(ee.Reducer.minMax());
```

#### Combining reducers
If your intent is to apply multiple reducers to the same inputs, it's good practice to combine() the reducers for efficiency.
```
// Combine the mean and standard deviation reducers.
var reducers = ee.Reducer.mean().combine({
  reducer2: ee.Reducer.stdDev(),
  sharedInputs: true
});
```

#### ImageCollection reduction
The output is computed pixel-wise, such that each pixel in the output is composed of the median value of all the images in the collection at that location. For basic statistics like min, max, mean, etc., ImageCollection has shortcut methods like min(), max(), mean(), etc.
```
// Load an image collection, filtered so it's not too much data.
var collection = ee.ImageCollection('LANDSAT/LT05/C01/T1')
  .filterDate('2008-01-01', '2008-12-31')
  .filter(ee.Filter.eq('WRS_PATH', 44))
  .filter(ee.Filter.eq('WRS_ROW', 34));
// Compute the median in each band, each pixel.
// Band names are B1_median, B2_median, etc.
var median = collection.reduce(ee.Reducer.median());
```

#### Image reduction
The output is also an image with number of bands equal to number of reducer outputs.
```
// Load an image and select some bands of interest.
var image = ee.Image('LANDSAT/LC08/C01/T1/LC08_044034_20140318')
    .select(['B4', 'B3', 'B2']);
// Reduce the image to get a one-band maximum value image.
var maxValue = image.reduce(ee.Reducer.max());
```

#### Statistics of an Image Region (image.reduceRegion)
```
// Reduce the region. The region parameter is the Feature geometry.
var meanDictionary = image.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: region.geometry(),
  scale: 30,
  maxPixels: 1e9
});
```
There are two ways to set the scale: by specifying the scale parameter, or by specifying a CRS and CRS transform.

#### Statistics of image regions (image.reduceRegions)
To get image statistics in multiple regions stored in a FeatureCollection, you can use image.reduceRegions() to reduce multiple regions at once.
```
var maineMeansFeatures = image.reduceRegions({
  collection: maineCounties,
  reducer: ee.Reducer.mean(),
  scale: 30,
});
```

#### Statistics of image neighborhood (reduceNeighborhood)
The reduction will occur in a sliding window over the input image, with the window size and shape specified by an ee.Kernel.
```
var texture = naipNDVI.reduceNeighborhood({
  reducer: ee.Reducer.stdDev(),
  kernel: ee.Kernel.circle(7),
});
```

#### Statistics of FeatureCollection Columns (featureCollection.reduceColumns)
```
// Compute a weighted mean and display it.
print(aFeatureCollection.reduceColumns({
  reducer: ee.Reducer.mean(),
  selectors: ['foo'],
  weightSelectors: ['weight']
}));
```

#### Raster to Vector conversion (reduceToVectors)
```
// Load a Japan boundary from the Large Scale International Boundary dataset.
var japan = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017')
  .filter(ee.Filter.eq('country_na', 'Japan'));

// Load a 2012 nightlights image, clipped to the Japan border.
var nl2012 = ee.Image('NOAA/DMSP-OLS/NIGHTTIME_LIGHTS/F182012')
  .select('stable_lights')
  .clipToCollection(japan);

// Define arbitrary thresholds on the 6-bit nightlights image.
var zones = nl2012.gt(30).add(nl2012.gt(55)).add(nl2012.gt(62));
zones = zones.updateMask(zones.neq(0));

// Convert the zones of the thresholded nightlights to vectors.
var vectors = zones.addBands(nl2012).reduceToVectors({
  geometry: japan,
  crs: nl2012.projection(),
  scale: 1000,
  geometryType: 'polygon',
  eightConnected: false,
  labelProperty: 'zone',
  reducer: ee.Reducer.mean()
});

// Display the thresholds.
Map.setCenter(139.6225, 35.712, 9);
Map.addLayer(zones, {min: 1, max: 3, palette: ['0000FF', '00FF00', 'FF0000']}, 'raster');

// Make a display image for the vectors, add it to the map.
var display = ee.Image(0).updateMask(0).paint(vectors, '000000', 3);
Map.addLayer(display, {palette: '000000'}, 'vectors'); 
```

#### Vector to Raster conversion (reduceToImage)
```
// Load a collection of US counties.
var counties = ee.FeatureCollection('TIGER/2018/Counties');

// Make an image out of the land area attribute.
var landAreaImg = counties
  .filter(ee.Filter.notNull(['ALAND']))
  .reduceToImage({
    properties: ['ALAND'],
    reducer: ee.Reducer.first()
});

// Display the county land area image.
Map.setCenter(-99.976, 40.38, 5);
Map.addLayer(landAreaImg, {
  min: 3e8,
  max: 1.5e10,
  palette: ['FCFDBF', 'FDAE78', 'EE605E', 'B63679', '711F81', '2C105C']
});
```
### Join
Joins are used to combine elements from different collections (e.g. ImageCollection or FeatureCollection) based on a condition specified by an ee.Filter. The filter is constructed with arguments for the properties in each collection that are related to each other. Specifically, leftField specifies the property in the primary collection that is related to the rightField in the secondary collection. The type of filter (e.g. equals, greaterThanOrEquals, lessThan, etc.) indicates the relationship between the fields. 

#### Simple joins (Join.simple)
A simple join returns elements from the primary collection that match any element in the secondary collection according to the match condition in the filter. 
```
// Load a Landsat 8 image collection at a point of interest.
var collection = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA')
    .filterBounds(ee.Geometry.Point(-122.09, 37.42));

// Define start and end dates with which to filter the collections.
var april = '2014-04-01';
var may = '2014-05-01';
var june = '2014-06-01';
var july = '2014-07-01';

// The primary collection is Landsat images from April to June.
var primary = collection.filterDate(april, june);

// The secondary collection is Landsat images from May to July.
var secondary = collection.filterDate(may, july);

// Use an equals filter to define how the collections match.
var filter = ee.Filter.equals({
  leftField: 'system:index',
  rightField: 'system:index'
});

// Create the join.
var simpleJoin = ee.Join.simple();

// Apply the join.
var simpleJoined = simpleJoin.apply(primary, secondary, filter);

// Display the result.
print('Simple join: ', simpleJoined);
```

#### Other joins 
* __Join.inverted__: Retain all images in the primary collection that are not in the secondary collection.
* __Join.inner__: Enumerate all matches between the elements of two collections.
