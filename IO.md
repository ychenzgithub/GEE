## IO
### [Export images](https://developers.google.com/earth-engine/exporting)
#### to Google Drive
```
//javascript
Export.image.toDrive({
  image: landsat,
  description: 'imageToDriveExample',
  scale: 30,
  region: geometry
});
```

Exporting data with the Python API requires the use of the ee.batch module, which accepts arguments differently than the JavaScript Export module. Specifically, it does not accept an argument dictionary and the region parameter cannot be an ee.Geometry object. Use named arguments and GeoJSON geometry coordinates instead. Additionally, export tasks must be started by calling the start() method on a defined export task. Started task status can be queried by calling the status() method on the task.

```
# python
task = ee.batch.Export.image.toDrive(image=myImg,
                                     region=myRegion.getInfo()['coordinates'],
                                     description='myDescription',
                                     folder='myGDriveFolder',
                                     fileNamePrefix='myFilePrefix',
                                     scale=myScale,
                                     crs=myCRS)
task.start()
task.status()
```
