## UI
### Chart
#### Time series chart (ui.Chart.image.series)
```
// Load Landsat 8 top-of-atmosphere (TOA) input imagery.
var collection = ee.ImageCollection('LANDSAT/LC08/C01/T1_TOA').select('B[1-7]');

// Define a region of interest as a buffer around a point.
var geom = ee.Geometry.Point(-122.08384, 37.42503).buffer(500);

// Create and print the chart.
print(ui.Chart.image.series(collection, geom, ee.Reducer.mean(), 30));
```
#### Histogram (ui.Chart.image.histogram)
```
var image = ee.Image('LANDSAT/LC08/C01/T1/LC08_044034_20140318')
    .select('B[2-5]');
// Define a region of interest as a buffer around a point.
var geom = ee.Geometry.Point(-122.08384, 37.42503).buffer(500);
// Pre-define some customization options.
var options = {
  title: 'Landsat 8 DN histogram, bands 2-5',
  fontSize: 20,
  hAxis: {title: 'DN'},
  vAxis: {title: 'count of DN'},
  series: {
    0: {color: 'blue'},
    1: {color: 'green'},
    2: {color: 'red'},
    3: {color: 'magenta'}}};
// Make the histogram, set the options.
var histogram = ui.Chart.image.histogram(image, geom, 30)
    .setSeriesNames(['blue', 'green', 'red', 'NIR'])
    .setOptions(options);
// Display the histogram.
print(histogram);
```
#### Image region chart (ui.Chart.image.regions)
```
// Load and display a Landsat 8 image's reflective bands.
var image = ee.Image('LANDSAT/LC08/C01/T1_TOA/LC08_026047_20140216')
    .select(['B[1-7]']);
Map.addLayer(image, {bands: ['B5', 'B4', 'B3'], min: 0, max: 0.5});

// Define and display a FeatureCollection of three known locations.
var points = ee.FeatureCollection([
  ee.Feature(ee.Geometry.Point(-99.25260, 19.32235), {'label': 'park'}),
  ee.Feature(ee.Geometry.Point(-99.08992, 19.27868), {'label': 'farm'}),
  ee.Feature(ee.Geometry.Point(-99.21135, 19.31860), {'label': 'urban'})
]);
Map.addLayer(points);

// Define customization options.
var options = {
  title: 'Landsat 8 TOA spectra at three points near Mexico City',
  hAxis: {title: 'Wavelength (micrometers)'},
  vAxis: {title: 'Reflectance'},
  lineWidth: 1,
  pointSize: 4,
  series: {
    0: {color: '00FF00'}, // park
    1: {color: '0000FF'}, // farm
    2: {color: 'FF0000'}, // urban
}};

// Define a list of Landsat 8 wavelengths for X-axis labels.
var wavelengths = [0.44, 0.48, 0.56, 0.65, 0.86, 1.61, 2.2];

// Create the chart and set options.
var spectraChart = ui.Chart.image.regions(
    image, points, ee.Reducer.mean(), 30, 'label', wavelengths)
        .setChartType('ScatterChart')
        .setOptions(options);

// Display the chart.
print(spectraChart);
```
#### Time series in image regions (ui.Chart.image.seriesByRegion)
#### DOY series (ui.Chart.image.doySeries)
#### Charts by image classes (ui.Chart.image.byClass)
#### Feature charts  
Use ui.Chart.feature.byProperty() to plot each feature as a different series, with properties on the X-axis. Use ui.Chart.feature.byFeature() to plot specified properties, with features on the X-axis
#### Feature group charts (ui.Chart.feature.groups)
#### Array data charts (ui.Chart.array.values)
