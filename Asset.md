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
__earthengine upload__ can be used to move data from cloud storage to GEE asset. Please check next section to learn the approach of uploading local data to gcloud.
```
earthengine upload image --asset_id=users/username/asset_id gs://bucket/image.tif  // upload from google cloud storage to assets
earthengine upload table --asset_id=users/username/myUploadedShapefile gs://bucket/foo.shp
```
This approach will only move files into a collection without passing properties such as timestamp, which is very important once that you need to filter by date. The following example shows one approach to attach timestamp and some other properties to each file uploaded into a collection.
```
# ******************** Bash script ********************************************#
#!/bin/bash
# Define static variables
gcBucket="bucket_climate/monthly/tmean/"
imgCol="users/gponce/usda_ars/image_collections/climate/tmean/"
# Create asset in GEE
earthengine create collection $imgCol
strProv="(string)provider=USDA-ARS-SWRC"
# Get file names to extract date and call ingestion command for each file to be added into an asset as image collection
# Example of filenames used here are from a monthly timeseries: rainfall_US_20140101.tif
for file in *.tif; do
    var=$file                       # Get the file name
    yr=${var:12:4}                  # Extract year from the filename, to get this index use the filename and check the index for the year within the filename.  0-based-index
    mon=${var:16:2}                 # Same for extracting month
    vdate=$yr"-"$mon"-01T12:00:00"  # Concatenate strings to get a valid date format to use in the data ingestion co:mmand
    # Remove filename extension ".tif"
    name=$(echo $file | cut -f 1 -d '.')
    asset=$imgCol$name
    # For testing before running, use the following echo statament to print out the final command
    #echo earthengine upload image --asset_id="${asset}" --pyramiding_policy=sample --time_start="${vdate}" --property="\x22${strProv}\x22" --nodata_value=-9999 gs://$gcBucket$file
    # Call the ingestion command with the corresponding parameters to populate properties at each image ingested
    earthengine upload image --asset_id="${asset}" --pyramiding_policy=sample --time_start="${vdate}" --property="${strProv}" --nodata_value=-9999 gs://$gcBucket$file
done
```
Once the files are uploaded into GEE you can find the image collection at the path you specified above or you can go use to JS code editor and type something like:
```
var my_image_collecton = ee.ImageCollection('users/my_user/climate/image_collections/tmean')
```

### Google cloud (gcloud)
So far, you need to follow LocalDrive –> Google Cloud Storage (bucket) –> GEE (asset) to upload local data to GEE asset. 

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


### Reference
* [A tutorial of uploading data to GEE asset](https://www.tucson.ars.ag.gov/notebooks/uploading_data_2_gee.html)
