# Sat-Pipeline CLI
This project supports Planet assets such as PlanetScope and RapidEye imagery along with Digital globe assets including Quickbird, GeoEye, WorldView-2 and WorldView-3 using associated metadata files. 

Planet Inc is a company that is currently trying to map the entire planet every single day and as the constellation size increases the data value and repeat cycle increases as well.(https://www.planet.com/) The planet labs API consists of searching for satellite imagery type assets using a structured geojson

DigitalGlobe is an company that is currently running request based remote sensing data acquisiton and runs an array of high resolution satellites and its interest also include machine learning algorthms for object detection. The current DG system does not allow for a data download pipeline but this setup is for automated metadata parsing and asset uploading to Google Earth Engine. 

Google Earth Engine is a distributed computing based planetary remote sensing system which is build on a scalable architecture approach and allows for the processing and handling of data. GEE only hosts publicly available data and additional data can be added by the user using a storage quota. The system evolves and change log is maintained at GEE (https://earthengine.google.com/)

1) Planet API is first used to define a bounding area and download the images to a storage can be Amazon S3 bucket/Google Cloud Storage Bucket of local or hosted storage held in volume. (This is pull function and requires API Key from Planet)

2) Digital Globe datasets are requests directly or via third party vendor along with additional IMD and XML files. The image must be converted to geotiff if they are not already standard geotiff.

3) The metadata file is sliced from the xml input files accompanying each asset into a csv file to be read along with asset/image id.

3) Take both the downloaded tiles and the batch ingestion manager created by Lucas (https://github.com/tracek/gee_asset_manager) and upload the imagery to Google Earth Engine (This is push function and requires google credentials)

The project will allow for the service to be a seamless pull, parse and push service(for Planet) and mostly (Parse & Push) for Digital Globe. The Pipeline serves as temporary storage upto a certain volume before a copy of the pull is pushed to both archival/long term storage and Earth Engine.

## Table of contents
* [Installation](#installation)
* [Prerequisites](#prerequisites)
* [Getting started](#getting-started)
	* [GeoObject to JSON](#geoobject-to-json)
    * [CLI Main Tool](#cli-main-tool)
    * [JSON Parse and Asset Download](#json-parse-and-asset-download)
    * [Metadata Parse Tool](#metadata-parse-tool)
* [Usage examples](#usage-examples)
    * [Choosing the Main tool](#choosing-the-main-tool)
	* [Converting to Structured JSON](#converting-to-structured-json)
	* [Creating JSON and Download Assets](#creating-json-and-download-assets)
	* [Metadata Parse](#metadata-parse)
	* [Upload to GEE](#upload-to-gee)
* [Credits](#credits)

## Installation
This tool can be run on both windows and linux distributions.


## Prerequisites
It assumes you use Python2.7 hopefully >=2.7.12 and that you have earth engine python API installed on your system. Instructions can be found [here] (https://developers.google.com/earth-engine/python_install)
In the download.py file edit the line with your Planet API Key and save[1].You can install all python dependencies by simply running[2]

```
	1) os.environ['PLANET_API_KEY']="Enter Your Planet Key Here"
	2) pip install -r requirements.txt
```

Command Line Interfaces are extremely useful since they can be created to be platform independent and they provide a short hand method to automate scripts and more concise and powerful means to control a program or operating systems.
There are three different tools and they are made to interact with each other working on both linux as well as windows systems. They are created in python and not bash and hence platform independent for as logn as dependencies are fulfilled.

## Getting started

The first step is to define a area that is available for download under your Planet API license. 
 * Go to geojson.io and once you have defined your area of interest click on save as GeoJson(A map.geojson file is created)
 * Place the map.geojson file inside the JSON Parser folder

 
## CLI Main Tool
This tool allows you to perform one of three tasks
1) Download Assets from planet including metadata files
2) Parse metadata tool allows us to parse metadata files from all assets
3) Last tool is the asset upload tool which uploads assets to GEE and uses parsed metadata from earlier step

```
usage: cli_toolsout.pyc [-h] [--tool TOOL] [--assetdown ASSETDOWN]
                        [--metadataparse METADATAPARSE] [--gee GEE]

Tool to download(PlanetScope/RapidEye assets),parse and upload
PlanetScope/RapidEye/Digital Globe assets to GEE choose --tool followed by
objective (ex: --tool DA/PM/UP --assetdown/--metadataparse/--gee "command")

optional arguments:
  -h, --help            show this help message and exit
  --tool TOOL           Download asset or Parse Metadata or Upload to
                        GEE?(DA/PM/UP)
  --assetdown ASSETDOWN
                        Give asset download argument: eg>> python
                        cli_jsonparse.pyc --start 2017-01-01 --end 2017-03-28
                        --cloud 0.2 --asset PS --geo ./map.geojson --activate
                        1000
  --metadataparse METADATAPARSE
                        Give metadata parse argument: eg>> python
                        cli_metadata.pyc --asset PS --mf ./psxml --mfile
                        ./psmetadata.csv --errorlog ./errorlog.csv
  --gee GEE             Give upload instructions: eg>> python geebam.py upload
                        -u johndoe@gmail.com --source
                        path_to_directory_with_tif -m path_to_metadata.csv
                        --dest users/johndoe/myfolder/mycollection --nodata
                        222


```

## GeoObject to JSON
This tool can be used to convert from kml(google earth file)/shapefile(ESRI) or geojson file from geojson.io to structured aoi.json used for querying planet datasets using API 1.0. The output is a file aoi.json which can then be used to query and download using cli_jsonparse tool.

```
usage: cli_aoi2json.pyc [-h] [--start START] [--end END] [--cloud CLOUD]
                        [--inputfile INPUTFILE] [--geo GEO]

Tool to convert KML, Shapefile or GeoJSON file to AreaOfInterest.JSON file
with structured query for use with Planet API 1.0

optional arguments:
  -h, --help            show this help message and exit
  --start START         Start date in YYYY-MM-DD?
  --end END             End date in YYYY-MM-DD?
  --cloud CLOUD         Maximum Cloud Cover(0-1) representing 0-100
  --inputfile INPUTFILE
                        Choose a kml/shapefile or geojson file for
                        AOI(KML/SHP/GJSON
  --geo GEO             map.geojson/aoi.kml/aoi.shp file
```
## JSON Parse and Asset Download
JSON Parse and asset download tool can be used as a stand alone tool to create image filters and download PlanetScope and RapidEye datasets.

```
usage: cli_jsonparse.pyc [-h] [--start START] [--end END] [--cloud CLOUD]
                        [--asset ASSET] [--geo GEO] [--activate ACTIVATE]

optional arguments:
  -h, --help           show this help message and exit
  --start START        Start date in YYYY-MM-DD?
  --end END            End date in YYYY-MM-DD?
  --cloud CLOUD        Maximum Cloud Cover(0-1) representing 0-100
  --asset ASSET        Whether PlanetScope or RapidEye assets(PS/RE)
  --geo GEO            map.geojson file
  --activate ACTIVATE  Enter estimated time for activation
```

## Metadata Parse Tool
The metadata parse tool is aptly named and is a standalone tool to parse PlanetScope/RapidEye/DigitalGlobe datasets and create outputs as csv files. Issues are logged into errorlogs.

```
usage: cli_metadata.pyc [-h] [--asset ASSET] [--mf MF] [--mfile MFILE]
                       [--errorlog ERRORLOG]

optional arguments:
  -h, --help           show this help message and exit
  --asset ASSET        Choose RapidEye/PlantScope/DigitalGlobe
                       MultiSpectral/DigitalGlobe Panchromatic
                       (RE/PS/DGMS/DGP)?
  --mf MF              Metadata folder?
  --mfile MFILE        Metadata filename to be exported along with Path.csv
  --errorlog ERRORLOG  Errorlog to be exported along with Path.csv
```
## Choosing the main tool
The main tool serves as a CLI interface to switch and move around between different tools and contains functionality all the way from creating json structure, to download, parse and upload to Earth Engine

```
To Download Assets						
python cli_toolsout.pyc --tool DA --assetdown "python cli_jsonparse.pyc --start 2017-01-01 --end 2017-03-28 --cloud 0.2 --asset PS --geo ./map.geojson --activate 1000"
If you already have an AOI and don't want to recreate it again and again use this instead
python cli_toolsout.pyc --tool DA --assetdown "python cli_jsonparse.pyc --asset PS --activate 1000"

To parse metadata once assets have been downloaded
python cli_toolsout.pyc --tool PM --metadataparse "python cli_metadata.pyc --asset PS --mf ./psxml --mfile ./psmetadata.csv --errorlog ./errorlog.csv"

To upload to Google Earth Engine
python cli_toolsout.pyc --tool UP --gee "python geebam.py upload -u johndoe@gmail.com --source path_to_directory_with_tif -m path_to_metadata.csv --dest users/johndoe/myfolder/mycollection --nodata 222"
```

## Converting to Structured JSON
This tool can be used to convert between multiple file formats to a structured json which is accepted by Planet API. It can read Google Earth kml files, ESRI shapefiles as well as geojson files downloaded from map.geojson. For use of use, it is better to use a bounding box to encompass area of interest, multivertex polygon to JSON don't always go well when querying using such complex json structures.

```
For KML Files
python cli_aoi2json.pyc --start 2017-01-01 --end 2017-04-02 --cloud 0.15 --inputfile KML --geo "./lybbox.kml"

For ESRI ShapeFiles
python cli_aoi2json.pyc --start 2017-01-01 --end 2017-04-02 --cloud 0.15 --inputfile SHP --geo "./lybbox.shp"
```

## Creating JSON and Download Assets
This can be used as a standalone to format JSON to query assets from Planet API

```
python cli_jsonparse.pyc --start 2017-01-01 --end 2017-03-28 --cloud 0.2 --asset PS --geo ./map.geojson --activate 1000

If AOI already exists
python cli_jsonparse.pyc --asset PS --activate 1000
```

## Metadata Parse
Metatadata parsing tool can handle four different kinds of asset metadata files for now, PlanetScope, RapidEye, DigitalGlobe MultiSpectral and Digital Globe Panchromatic

```
python cli_toolsout.pyc --tool PM --metadataparse python cli_metadata.pyc --asset PS --mf ./psxml --mfile ./psmetadata.csv --errorlog ./errorlog.csv
```

## Upload to GEE
Upload to GEE can be used as a standadlone outside the main CLI tool similar to all other tools in this package

```
python geebam.py upload -u johndoe@gmail.com --source path_to_directory_with_tif -m path_to_metadata.csv --dest users/johndoe/myfolder/mycollection --nodata 222
```

The next iteration of the tool will include methods to upload to different ftp sites or cloud storage.

# Credits

[JetStream](https://jetstream-cloud.org/) A portion of the work is suported by JetStream Grant TG-GEO160014.

Also supported by [Planet Labs Ambassador Program](https://www.planet.com/markets/ambassador-signup/)
