## Asset

The online JavaScript Code Editor can be used to upload data to GEE asset. However, this approach will only allow you to upload single files, one by one instead of using a batch process. To more advanced asset management, the GEE command line tool is needed.

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
So far, you need to follow LocalDrive –> Google Cloud Storage (bucket) –> GEE (asset) to upload local data to GEE asset. 

__earthengine upload__ can be used to move data from cloud storage to GEE asset.
```
earthengine upload image --asset_id=users/username/asset_id gs://bucket/image.tif  // upload from google cloud storage to assets
earthengine upload table --asset_id=users/username/myUploadedShapefile gs://bucket/foo.shp
```

### Google cloud (gcloud)
#### Authentication
Most of the operations you perform in Cloud Storage must be [authenticated](https://cloud.google.com/storage/docs/authentication).
```
gcloud auth login      // authentication
gcloud config set project PROJECT_ID   // set project
```
#### gsutil
[gsutil](https://cloud.google.com/storage/docs/gsutil) is a Python application that lets you access Cloud Storage from the command line.
```
gsutil mb gs://my_bucket                               // # Create a bucket, a folder for data staging
gsutil ls gs://                                        // # list buckets you have in your project
gsutil -m cp *.tif gs://my_bucket/folder_A/            // # Uploading files from local disk into Google Cloud
```
