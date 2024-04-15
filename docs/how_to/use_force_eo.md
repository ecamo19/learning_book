## [What are Levels?](https://force-eo.readthedocs.io/en/latest/howto/l2-ard.html#what-are-levels "Permalink to this heading")

Remote sensing products are grouped in a hierarchical classification scheme.

- Level 0 data are the measurements taken onboard the satellite - they are not available to users.

- Level 1 data are radiometrically calibrated and georectified.

- Level 2 data most notably include some sort of atmospheric correction and probably other corrections like topographic correction.

- Level 3 data are temporal Level 2 aggregates, e.g. pixel based composites or statistical aggregations like multitemporal averages.

- Level 4 products are model output (classifications etc.), often derived from multi-temporal or multi-sensor measurements.

## [What are Analysis Ready Data?](https://force-eo.readthedocs.io/en/latest/howto/l2-ard.html#what-are-analysis-ready-data "Permalink to this heading")

 ARD are readily usable for any application without much further processing. Such data need to be well corrected for:
 
 + Atmospheric and other effects
 
 + Have undergone a very good and aggressive cloud screening
 
 + Are accompanied by pixel-based quality indicators (cloud masks but also other criteria)
 
 + Provided in a regular non-overlapping grid system without any redundancy in a single coordinate system (at least on the continental scale) in the form of data cubes.

## The datacube

This section contain information about:

+ What a datacube is
+ How is parameterized
+ How you can find a POI
+ How to visualize the tiling grid
+ How to display cubed data
### Datacube concept

FORCE use a data cube that contains data with two characteristics:
+ All data are in the __same coordinates system__
+ The data are organized in regular, non-overlapping tiles

![Datacube concept](media/force_eo_datacube.png)


## FORCE Level 1 Processing System (L1PS)

### Create the skeleton of a FORCE project

```bash 
# Create a FORCE project. Save the following function as create_force_eo_project.sh

#!/bin/bash

# This bash function create the recomemded folder structure to store any files
# related a force project. For more info go to
# (https://force-eo.readthedocs.io/en/latest/howto/l2-ard.html)

# Create flag for while loop
flag=true

# Check if force is installed -------------------------------------------------
if ! command -v force &> /usr/local/bin/force; then

	echo 'FORCE not found. Make sure that is installed in /usr/local/bin/'

	echo 'Read https://force-eo.readthedocs.io/en/latest/setup/requirements.html for installation instructions'

	# Change flag to close while loop
	flag=false
fi

# Loop for creating folder structure ------------------------------------------
while $flag; do

	# Read user's input
	printf "\nPress CTRL+D or enter q for quitting the program."

	printf "\nEnter a name for your force project.\n"

	read force_dir

	# Exit if letter q is entered
	if [ "$force_dir" == "q" ]; then

		# Change flag to close while loop
		flag=false
		
		# Print message
		echo "Program closed"

	# Check if folder exist
	elif [ -d "$force_dir" ]; then

		# Print message
		printf "\nFolder called "$force_dir" already exist. Choose another name.\n"

	# Create folder structure if force_dir is not AND does not exist
	elif ! { [ "$force_dir" == "quit" ] && [ -d "$force_dir" ]; }; then

		# Create main directory
		mkdir $force_dir

		# Create subdirectories at $force_dir

		# Folder for saving the param file
		mkdir -p "$force_dir/data/catalogue"

		# Folder for saving the param file
		mkdir -p "$force_dir/data/force/param"

		# The queue.txt files is saved here
		mkdir -p "$force_dir/data/force/level_1"

		# Folder for saving the log file
		mkdir -p "$force_dir/data/force/log"

		# Folder for temporarily unpacking zip/tar.gz containers
		mkdir -p "$force_dir/data/force/temp"

		# Folder for saving the output
		mkdir -p "$force_dir/data/force/level_2"

		# Folder for saving the Digital Elevation Model (DEM)
		mkdir -p "$force_dir/data/force/misc/dem"

		# Folder for saving the Water Vapor Database for water vapor correction
		mkdir -p "$force_dir/data/force/misc/wvdb"

		mkdir -p "$force_dir/data/force/provenance"

		# Add empty parameter file
		force-parameter "$force_dir/data/force/param/l2ps.prm" LEVEL2

		# Download catalogue
		printf "\nDownloading FORCE catalogue\n"

		force-level1-csd -u "$force_dir/data/catalogue"

		# Print message
		printf "\nSuccess! FORCE project named "$force_dir" created at: $PWD\n"

		# Change flag to close while loop
		flag=false

	# Break code if unknown condition is met
	else
		# Print message
		echo "Failed creating folder structure"

		# Break code
		break

	# Close if-else
	fi
	
# Close while loop
done
```

```bash
# Create a FORCE project
bash create_force_eo_project.sh
```

### Cloud storage Downloader


This section covers the process of querying and downloading Landsat and Sentinel-2
level 1 data from cloud services.

Download the metadata catalogues:

```bash
# Download metadata catalogues as a first step
force-level1-csd -u path/to/catalogue
```

#### Parameterization

![Parameters](media/force_eo_basic_syntax_level_1.png)

##### Mandatory arguments

`metadata-dir`: Directory where metadata is stored (CSV file)

`level-1-datapool`: An existing directory. The files will be stored here

`queue`: Queue files that will be downloaded

`aoi`: Area of interest with:

+ (1) User-supplied coordinates. The polygon must be closed (fist (X,Y) and last (X,Y) are equal). (X,Y) must be given as decimal degrees with negative values for west and South coordinates. Either specify the path to a file (one coordinate pair per line?) or the coordinates on the command-line. [This webpage](http://bboxfinder.com/) could be used to define an `aoi`

+ (2) A shapefile (point/polygon/line). EPSG4326 projection recommended.

+ (3) Scene identifier: Landsat (Path/Row) as PPPRRR. Make sure to keep leading zeros (i.e. 181034 not 18134).

	+ Sentinel-2 (MGRS tile) as "TXXXXX". Make sure to keep the leading T before the MGRS tile number.
	
	 + Either specify the path to a file (one ID per line) or give the ID's on the command-line. If on command line, provide a comma separated list.
	
	
	 
##### Optional Arguments

+ (1) Cloud cover (-c | --cloudcover): Range between 0-100%.

+ (2) Date range (-d | --daterange): Range dates with the format YYYYMMDD,YYYYMMDD.

+ (3) Sensor (-s | --sensor): Default: LT04,LT05,LE07,LC08,S2A,S2B.
	+ Landsat
		+ LT04 = Landsat 4 TM
		+ LT05 = Landsat 5 TM
		+ LE07 = Landsat 7 ETM+
		+ LC08 = Landsat 8 OLI
	+ Sentinel
		+ S2A - Sentinel-2A MSI
		+ S2B - Sentinel-2B MSI
		+ 
+ (4) Tier level (-t | --tier): Landsat collection tier (Default: T1). Valid tiers = T1, T2, RT.

+ (5) Dry run (-n | --no-act). Will trigger a dry run that will only return the number of images and their total data volume.

+ (6) Keep metadata (-k | --keep-meta). Will write the results of the query to the level 1 datapool directory. Two files will be created if Landsat and Sentinel-2 data is queried at the same time. Filename: csd_metadata_YYYY-MM-DDTHH-MM-SS.

+ (7) Check for FORCE Level-2 log files (-l | --logs). Check and and remove any products from the search that have been processed previously. Note that this only checks for the presence of log files, not for actual Level-2 products.


#### Querying and downloading data 

```bash
# Create folder to store the catalog
mkdir catalog 

# Download catalog 
force-level1-csd -u catalog

# Pay attention!!! This will return an error
force-level1-csd -u /catalog
```

```bash
# Create the folder to store the data
mkdir downloaded_data
```

```bash
# Get summary of the data. This is achieve by using the -n
# the queue.txt will be created
force-level1-csd -n catalog downloaded_data downloaded_data/queue.txt long/lat 
```

```bash
# Download data
force-level1-csd catalog downloaded_data downloaded_data/queue.txt long/lat 

# Example. PAY attention the f
force-level1-sentinel2 downloaded_data downloaded_data/queue.txt "25.43/-12.46, 25.94/-12.46, 25.94/-11.98, 25.39/-11.99, 25.43/-12.46" 
```

![Example output](media/force_eo_level1_csd_example_output.png)

##### Saving metadata

The metadata can be very helpful when creating statistics and visualizations about data availability over time, cloud cover distribution over time, data volume (e.g., per sensor or year), etc. Using the `-k | --keep-meta` option will save the results of the current query to the Level 1 datapool folder under the file name.


```bash
# Download data
force-level1-csd -k catalog downloaded_data downloaded_data/queue.txt long/lat 
```

## FORCE Level 2 Processing System (L2PS)

L2PS converts each Level 1 image to ARD specification.  This includes three main processing steps:

1. Cloud and cloud shadow detection
2. Radiometric correction
3. Data cubing

For Sentinel-2, two additional options are implemented:

1. Resolution merging, i.e. increase the spatial resolution of the 20m bands to 10m
2. Co-registration with Landsat time series


![L2PS](media/force_eo_l2ps.png)

### Folder structure

```bash
mkdir /data/force/level1
mkdir /data/force/level2
mkdir /data/force/param
mkdir /data/force/log
mkdir /data/force/misc
mkdir /data/force/temp
mkdir /data/force/provenance
```


![Folder structure](media/force_eo_folder_structure.png)

### Parameter file

```bash
# Create empty parameter file
force-parameter data/force/param/l2ps.prm LEVEL2
```

#### Input / Output


```bash
# Edit parameters inside parameter file
nano l2ps.prm

# Add queue file created with force-level1-csd function
FILE_QUEUE = /data/force/level1/queue.txt

# Set the directories for output, logfiles and temporary data
DIR_LEVEL2 = /data/force/level2
DIR_LOG = /data/force/log
DIR_TEMP = /data/force/temp
```

#### Digital Elevation model (DEM)

In principle, FORCE L2PS can be used without a DEM (FILE_DEM = NULL). In this case, the surface is assumed to be flat at z = 0m a.s.l. However it is __strongly advised to use a DEM__.

A DEM is necessary to:

- Enhanced cloud and cloud shadow detection. This step is used to distinguish cloud shadows from water and topographic shadows. 

- Perform an atmospheric correction. This step is used to scale the optical depths with altitude. 

- Perform the topographic correction. The topographic correction is of course relying on the DEM

##### Data format

- The unit must be meters.
- The Nodata value shouldn’t be 0, which is a valid elevation.
- The DEM must cover the complete image(s) to be processed.


The DEM should match the resolution of the Level 1 image data as closely as possible. It is advised to use a finer resolution. However, as it is hard to acquire high spatial resolution DEMs, especially for larger areas, lower resolution works too. Often, we use the 30m SRTM DEM or 30m ASTER DEM, or a combination thereof, e.g. SRTM filled with ASTER (SRTM is a bit better, but there are holes in mountainous regions, and coverage is only 60°N-60°S).

Use the https://earthexplorer.usgs.gov/ for downloading data

```bash
# Prepare a text file that holds all the filepath
find /data/Dagobah/global/dem/srtm -name '*.tif' >/data/Earth/global/dem/srtm.txt
```

```bash
cat /data/Dagobah/global/dem/srtm.txt
```

 
```bash
# Use the gdalbuildvrt command to generate the virtual mosaic.
gdalbuildvrt -input_file_list /data/Dagobah/global/dem/srtm.txt /data/Earth/global/dem/srtm.vrt
```

```bash
# Check the file created
head -n 14 /data/Dagobah/global/dem/srtm.vrt
```

###### Optionally: Use R to download the data
 The idea about how to get a DEM from a place comes from this [video](https://www.youtube.com/watch?v=kGadI6_ZIR4)

```r
library(elevatr)
library(geodata)
elevation_30s(country="FRA", path="./Desktop")

# The data downloaded in the tutorial has a resolution of 1arc (~63.6*63.6)
```

#### Datacube parameters

To generate a Level 2 ARD (Analysis ready data) with `FORCE L2PS` is necessary to define the datacube in the parameter file.

`DO_REPROJ`:  Indicates whether the images should be reprojected to the target coordinate system - or stay in their original UTM projection.

`DO_TILE`: Indicates whether the images should be tiled to chips that intersect with the grid system - or stay in the original reference system (WRS-2/MGRS).

`PROJECTION`: Defines the target coordinate system. This projection should ideally be valid for a large geographic extent. The projection needs to given as “WKT” string. 

You can verify your projection (and convert to WKT from another format) using gdalsrsinfo function. If this fails, you need to fix the projection - otherwise FORCE L2PS will likely fail, too.

```r
gdalUtils::gdalsrsinfo()
```


`ORIGIN_LAT` and `ORIGIN_LONG`: Indicate the origin coordinates

`TILE_SIZE`:  Tile size (in target units, commonly in meters). Tiles are square.

`BLOCK_SIZE`: Block size (in target units, commonly in meters) of the image chips. Blocks are stripes, i.e. they are as wide as the tile and as high as specified here. The blocks represent the internal structure of the GeoTiffs, and represent the primary processing unit of the force-higher-level routines.

#### Radiometric correction

The default parameter file already has all radiometric corrections enabled, and this is the setup I commonly use for generating ARD. This includes atmospheric correction with multiple scattering effects, image-based AOD estimation, topographic correction, adjacency effect correction, and nadir BRDF correction. 

The only thing that needs to be changed (__and only if processing Landsat data__) is the parameterization of the water vapor correction.

#### Cloud detection

The default parameter file already has meaningful values for the cloud correction. I usually don’t tweak the Fmask parameters.

You can probably change the maximum cloud cover parameters to your liking. 

The `MAX_CLOUD_COVER_FRAME` parameter cancels the processing of images that exceed the given threshold.  The processing will be canceled right after cloud detection and thus saves quite some processing time. 

In my opinion, heavily clouded images are most often of little use, and even if cloud detection flags some pixels as “clear”, they are usually somewhat contaminated, e.g. in transition zones from clear-sky to cloud. Therefore, I commonly do not go up to 100%. The `MAX_CLOUD_COVER_TILE` parameter is similar, but it works on a per tile basis. It suppresses the output for chips (tiled image) that exceed the given threshold.

#### Resolution merge

This parameter defines the method used for improving the spatial resolution of Sentinel-2’s 20m bands to 10m. 

It defaults to the [ImproPhe code](https://ieeexplore.ieee.org/document/7452606), which is a data fusion option with both decent performance and quality. Let’s keep this method, but feel free to try the other options.

#### Co-Registration

Since v. 3.0, FORCE is able to perform a co-registration of Sentinel-2 images with Landsat time series. For starters, we will not use this option, but see the [Coregistration](https://force-eo.readthedocs.io/en/latest/howto/coreg.html#tut-coreg) tutorial.

#### Parallel Processing

```BASH
# Adjust these paramters
NPROC = 
NTHREAD = 
DELAY = 
```

#### Output options

The default output options are already my usual setup for ARD generation. 

The output files will be stored as compressed GeoTiff images with internal blocks for partial access. Note that metadata are written to the FORCE domain, thus they only show up if you look into all metadata domains, e.g. The Bottom-of-Atmosphere reflectance product and the Quality Assurance Information are written by default - and they can’t be disabled. 

I typically generate additional quicklooks (`OUTPUT_OVV`). If you want to generate pixel based composites in the next step, you should additionally output the `OUTPUT_DST`, `OUTPUT_VZN`, and `OUTPUT_HOT` products. The `OUTPUT_AOD` and `OUTPUT_WVP` products are not used by any higher level submodule - they are only useful for validation purposes.

### Processing

```bash
# Run these command
force-level2 /data/force/param/l2ps.prm
```

After processing, I recommend to check the logfiles, which we have written to `/data/force/log`

```bash
ls /data/force/log | tail
```

The logfiles report the percentage of data cover (how many pixels are not no-data), water cover, snow cover and cloud cover. 

Then, aerosol optical depth @ 550 nm (scene average), and the number of dark targets for retrieving aerosol optical depth (over water/vegetation) are printed. 

Then, the number of products written (number of tiles), and a supportive success indication is printed. In the case the overall cloud coverage is higher than allowed, the image is skipped. The processing time (real time) is appended at the end.

```bash
cat /data/force/log/* | tail
```

#### Create plot report 

```bash
force-level2-report /data/force/log
```

## Output format

For more details, see the [Output Format](https://force-eo.readthedocs.io/en/latest/components/lower-level/level2/format.html#level2-format).

The output data are organized in data cubes. The tiles manifest as directories in the file system, and the images are stored within. This is decribed in more detail in the [The Datacube](https://force-eo.readthedocs.io/en/latest/howto/datacube.html#tut-datacube).

Basically, for each tile, you get a time series of square image chips that always show the same extent.

Each dataset consists of a _BOA_ and _QAI_ product, which are Bottom-of-Atmosphere reflectance and Quality Assurance Information. Depending on parameterization ,there are more products, e.g. _OVV_ for image overviews (quicklooks).

The reflectance products are multi-band images and consist of 6 bands for Landsat (Landsat legacy bands), and 10 bands for Sentinel-2 (land surface bands). All bands are provided at the same spatial resolution, typically 30m for Landsat and 10m for Sentinel-2.

QAI are provided bit-wise for each pixel. QAI are essential for making your analyses a success, therefore, please have a look at the [Quality Assurance Information](https://force-eo.readthedocs.io/en/latest/howto/qai.html#tut-qai) tutorial.

Metadata are written to all output products. Note that FORCE-specific metadata will be written to the FORCE domain, and thus are probably not visible unless the FORCE domain (or all domains) are specifically requested: