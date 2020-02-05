## Asset

### Command line tool
The earthengine tool is a utility program that allows you to manage Earth Engine assets and tasks from the command line. It is installed automatically when you install the Python API.  [Description](https://developers.google.com/earth-engine/command_line) 

#### get help
```
earthengine command -h
```
#### Authenticate
```
earthengine authenticate
```
#### Assests
```
earthengine asset info users/username/asset_id               // get assets info
earthengine create folder users/username/folder_id           // create new folder
earthengine create collection users/username/collection_id   // create new image collection
earthengine ls -r users/username                             // list contents
earthengine cp source destination                            // copy asset
earthengine mv source destimation                            // move asset
```
#### Projects
```
earthengine set_project foo-project                          // set project
```
#### Upload from google cloud storage
```
earthengine upload image --asset_id=users/username/asset_id gs://bucket/image.tif  // upload from google cloud storage to assets
earthengine upload table --asset_id=users/username/myUploadedShapefile gs://bucket/foo.shp
```
