## Vector data: feature and feature collection

### Geometry

#### Define geometry
```
var point = ee.Geometry.Point([1.5, 1.5]);
var lineString = ee.Geometry.LineString(
  [[-35, -9], [35, -9], [35, 9], [-35, 9]]);
var linearRing = ee.Geometry.LinearRing(
  [[-35, -10], [35, -10], [35, 10], [-35, 10], [-35, -10]]);
var rectangle = ee.Geometry.Rectangle([-40, -20, 40, 20]);
var polygon = ee.Geometry.Polygon([
  [[-5, 0], [65, 0], [65, 20], [-5, 20], [-5, 20]]
]);
var circle = point.buffer(1000000);
```
#### Get information and metadata
```
print('Polygon printout: ', polygon);

// Print polygon area in square kilometers.
print('Polygon area: ', polygon.area().divide(1000 * 1000));

// Print polygon perimeter length in kilometers.
print('Polygon perimeter: ', polygon.perimeter().divide(1000));

// Print the geometry as a GeoJSON string.
print('Polygon GeoJSON: ', polygon.toGeoJSONString());

// Print the GeoJSON 'type'.
print('Geometry type: ', polygon.type());

// Print the coordinates as lists.
print('Polygon coordinates: ', polygon.coordinates());

// Print whether the geometry is geodesic.
print('Geodesic? ', polygon.geodesic());
      
```

#### Show geometry
```
Map.centerObject(point,4);
Map.addLayer(point,{},'point');
Map.addLayer(circle,{color:'a6bddb'},'circle');
Map.addLayer(lineString,{color:'red'},'linestring');
Map.addLayer(linearRing,{},'linearring');
Map.addLayer(rectangle,{color:'orange'},'rectangle');
Map.addLayer(polygon,{color:'blue'},'polygon');
```

#### geometric operations
```
// Compute the centroid of the polygon.
var centroid = polygon.centroid();

// Create a right-inside polygon.
var holePoly = ee.Geometry.Polygon({
  coords: [
    [[-35, -10], [-35, 10], [35, 10], [35, -10], [-35, -10]]
  ],
  evenOdd: false
});

// Create an even-odd version of the polygon.
var evenOddPoly = ee.Geometry({
  geoJson: holePoly,
  evenOdd: true
});

// Create a point to test the insideness of the polygon.
var pt = ee.Geometry.Point([1.5, 1.5]);

// Check insideness with a contains operator.
print(holePoly.contains(pt));       // false
print(evenOddPoly.contains(pt));    // true
    
// Create two circular geometries.
var poly1 = ee.Geometry.Point([-50, 30]).buffer(1e6);
var poly2 = ee.Geometry.Point([-40, 30]).buffer(1e6);

// Display polygon 1 in red and polygon 2 in blue.
Map.setCenter(-45, 30);
Map.addLayer(poly1, {color: 'FF0000'}, 'poly1');
Map.addLayer(poly2, {color: '0000FF'}, 'poly2');

// Compute the intersection, display it in blue.
var intersection = poly1.intersection(poly2, ee.ErrorMargin(1));
Map.addLayer(intersection, {color: '00FF00'}, 'intersection');

// Compute the union, display it in magenta.
var union = poly1.union(poly2, ee.ErrorMargin(1));
Map.addLayer(union, {color: 'FF00FF'}, 'union');

// Compute the difference, display in yellow.
var diff1 = poly1.difference(poly2, ee.ErrorMargin(1));
Map.addLayer(diff1, {color: 'FFFF00'}, 'diff1');

// Compute symmetric difference, display in black.
var symDiff = poly1.symmetricDifference(poly2, ee.ErrorMargin(1));
Map.addLayer(symDiff, {color: '000000'}, 'symmetric difference');
    
```

#### convert geodesic geometries to planar geometries
```
var planarPolygon = ee.Geometry(polygon, null, false);
Map.addLayer(planarPolygon,{color:'green'},'polygon');
```

### Feature
#### create feature
```
var polyFeature = ee.Feature(polygon, {foo: 42, bar: 'tart'});
print(polyFeature);
Map.addLayer(polyFeature, {}, 'feature');
```
#### get or set feature properties
```
// Make a feature and set some properties.
var feature = ee.Feature(ee.Geometry.Point([-122.22599, 37.17605]))
  .set('genus', 'Sequoia').set('species', 'sempervirens');

// Get a property from the feature.
var species = feature.get('species');
print(species);

// Set a new property.
feature = feature.set('presence', 1);

// Overwrite the old properties with a new dictionary.
var newDict = {genus: 'Brachyramphus', species: 'marmoratus'};
var feature = feature.set(newDict);

// Check the result.
print(feature);
      
```

### FeatureCollection
#### from list of feature
```
var features = [
ee.Feature(rectangle, {name: 'Voronoi'}),
ee.Feature(point, {name: 'Thiessen'}),
ee.Feature(planarPolygon, {name: 'Dirichlet'})
];
print(features);
var fromList = ee.FeatureCollection(features);
```

#### from table datasets
```
var fc = ee.FeatureCollection('TIGER/2016/Roads');
Map.setCenter(-73.9596, 40.7688, 12);
Map.addLayer(fc, {}, 'Census roads');
```

#### featureCollection metadata
```
// Load watersheds from a data table.
var sheds = ee.FeatureCollection('USGS/WBD/2017/HUC06')
  // Filter to the continental US.
  .filterBounds(ee.Geometry.Rectangle(-127.18, 19.39, -62.75, 51.29))
  // Convert 'areasqkm' property from string to number.
  .map(function(feature){
    var num = ee.Number.parse(feature.get('areasqkm'));
    return feature.set('areasqkm', num);
  });

// Show metadata
print('First watershed', sheds.first());
print('Count:', sheds.size());

// Print stats for an area property.
print('Area stats:', sheds.aggregate_stats('areasqkm'));
```

#### reduce featurecollection
```
// To aggregate data in the properties of a FeatureCollection, use featureCollection.reduceColumns()
// Load watersheds from a Fusion Table and filter to the continental US.
var sheds = ee.FeatureCollection('ft:1IXfrLpTHX4dtdj1LcNXjJADBB-d93rkdJ9acSEWK')
  .filterBounds(ee.Geometry.Rectangle(-127.18, 19.39, -62.75, 51.29));

// This function computes the squared difference between an area property
// and area computed directly from the feature's geometry.
var areaDiff = function(feature) {
  // Compute area in sq. km. directly from the geometry.
  var area = feature.geometry().area().divide(1000 * 1000);
  // Compute the differece between computed area and the area property.
  var diff = area.subtract(feature.get('AreaSqKm'));
  // Return the feature with the squared difference set to the 'diff' property.
  return feature.set('diff', diff.pow(2));
};

// Map the difference function over the collection.
var rmse = ee.Number(sheds
    .map(areaDiff)
    // Reduce to get the mean squared difference.
    .reduceColumns(ee.Reducer.mean(), ['diff'])
    .get('mean'))
    // Compute the square root of the mean square to get RMSE.
    .sqrt();

// Print the result.
print('RMSE=', rmse);
```

```
// To overlay features on imagery, use image.reduceRegions() with featurCollection as a parameter
// Load an image of daily precipitation in mm/day.
var precip = ee.Image(ee.ImageCollection('NASA/ORNL/DAYMET_V3').first());

// Load watersheds from a Fusion Table and filter to the continental US.
var sheds = ee.FeatureCollection('ft:1IXfrLpTHX4dtdj1LcNXjJADBB-d93rkdJ9acSEWK')
  .filterBounds(ee.Geometry.Rectangle(-127.18, 19.39, -62.75, 51.29));

// Add the mean of each image as new properties of each feature.
var withPrecip = precip.reduceRegions(sheds, ee.Reducer.mean());

// This function computes total rainfall in cubic meters.
var prcpVolume = function(feature) {
  // Precipitation in mm/day -> meters -> sq. meters.
  var volume = ee.Number(feature.get('prcp'))
    .divide(1000).multiply(feature.geometry().area());
  return feature.set('volume', volume);
};

var highVolume = withPrecip
  // Map the function over the collection.
  .map(prcpVolume)
  // Sort descending.
  .sort('volume', false)
  // Get only the 5 highest volume watersheds.
  .limit(5);

// Print the resulting FeatureCollection
print(highVolume);
    
```

### Vector to Raster Interpolation

#### Inverse distance weighted interpolation
```
// Load a Fusion Table corresponding to mean annual PM2.5 concentrations at points.
var airQualityMeasurements = ee.FeatureCollection('ft:14BLob4jGA6au2MB1cx0GhxYDvZc-lVLAWhqBAZuN');
Map.addLayer(airQualityMeasurements, {}, 'Mean annual PM2.5 concentrations (micrograms/m^3)');

// This is the name of the property to interpolate.
var propertyToInterpolate = 'ArithmeticMean';

// Combine mean and SD reducers for efficiency.
var combinedReducer = ee.Reducer.mean().combine({
    reducer2: ee.Reducer.stdDev(),
    sharedInputs: true
});

// Estimate global mean and standard deviation (SD) from the points.
var stats = airQualityMeasurements.reduceColumns({
  reducer: combinedReducer,
  selectors: [propertyToInterpolate]
});

// Do the interpolation, valid to 50 kilometers.
var interpolatedPM25 = airQualityMeasurements.inverseDistance({
  range: 50 * 1000,
  propertyName: propertyToInterpolate,
  mean: stats.get('mean'),
  stdDev: stats.get('stdDev'),
  gamma: 0.5
});

// Visualize the resulting interpolated raster.
var vis = {min: 0, max: 15, palette: ['blue', 'green', 'red']};
Map.setCenter(-121.7944, 36.9235, 7);
Map.addLayer(interpolatedPM25, vis, 'Interpolated PM2.5 concentration');
```

#### Kriging
```
// Load an image of sea surface temperature (SST).
var sst = ee.Image('NOAA/AVHRR_Pathfinder_V52_L3/20120802025048')
  .select('sea_surface_temperature')
  .rename('sst')
  .divide(100);

// Define a geometry in which to sample points
var geometry = ee.Geometry.Rectangle([-65.60, 31.75, -52.18, 43.12]);

// Sample the SST image at 1000 random locations.
var samples = sst.addBands(ee.Image.pixelLonLat())
  .sample({region: geometry, numPixels: 1000})
  .map(function(sample) {
    var lat = sample.get('latitude');
    var lon = sample.get('longitude');
    var sst = sample.get('sst');
    return ee.Feature(ee.Geometry.Point([lon, lat]), {sst: sst});
  });

// Interpolate SST from the sampled points.
var interpolated = samples.kriging({
  propertyName: 'sst',
  shape: 'exponential',
  range: 100 * 1000,
  sill: 1.0,
  nugget: 0.1,
  maxDistance: 100 * 1000,
  reducer: 'mean',
});

var colors = ['00007F', '0000FF', '0074FF',
              '0DFFEA', '8CFF41', 'FFDD00',
              'FF3700', 'C30000', '790000'];
var vis = {min:-3, max:40, palette: colors};

Map.setCenter(-60.029, 36.457, 5);
Map.addLayer(interpolated, vis, 'Interpolated');
Map.addLayer(sst, vis, 'Raw SST');
Map.addLayer(samples, {}, 'Samples', false);
    
```



### Show feature and featurecolletion
```
// Load a FeatureCollection from a Fusion Table: 'global 200' ecoregions.
var g200 = ee.FeatureCollection('ft:11SfWB6oBS1iWGiQxEOqF_wUgBJL7Bux-pWU-mqd5');

// Display as default and with a custom color.
Map.addLayer(g200, {}, 'default display');
Map.addLayer(g200, {color: 'FF0000'}, 'colored');
    
// For additional display options, use featureCollection.draw()
Map.addLayer(g200.draw({color: '006600', strokeWidth: 5}), {}, 'drawn');
        
// Create an empty image into which to paint the features, cast to byte.
var empty = ee.Image().byte();

// For more control over how a FeatureCollection is displayed, use image.paint() with the FeatureCollection as an argument. Unlike draw(), which outputs a three-band, 8-bit display image, image.paint() outputs an image with the specified numeric value ‘painted’ into it. 
// Paint all the polygon edges with the same number and width, display.
var outline = empty.paint({
  featureCollection: g200,
  color: 1,
  width: 3
});
Map.addLayer(outline, {palette: 'FF0000'}, 'edges');

// If the width parameter is not provided, the interior of the features is painted:
var fills = empty.paint({
  featureCollection: ecoregions,
  color: 'BIOME_NUM',
});
Map.addLayer(fills, {palette: palette, max: 14}, 'colored fills');

// Paint both the fill and the edges.
var filledOutlines = empty.paint(ecoregions, 'BIOME_NUM').paint(ecoregions, 0, 2);
Map.addLayer(filledOutlines, {palette: ['000000'].concat(palette), max: 14}, 'edges and fills');
```
