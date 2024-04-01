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

### Define the datacube parameters

To generate a Level 2 ARD (Analysis ready data) with `FORCE L2PS` is necessary to define the datacube in the parameter file.

Empty parameter files can be generated with `force-parameter`

`DO_REPROJ`: Indicates if images should be reprojected to the target coordinate system

`DO_TILE`: Indicates if the images should be tiled to chips that intersect with the grid system

`PROJECTION`: Defines the target coordinate system

`ORIGIN_LAT` and `ORIGIN_LONG`: Indicate the origin coordinates

`TILE_SIZE`

`BLOCK_SIZE`: Block size of the image chips. Block are stripes

## FORCE Level 1 Processing System (L1PS)

### Cloud storage Downloader


This section covers the process of querying and downloading Landsat and Sentinel-2
level 1 data from cloud services.

Download the metadata catalogues:

```bash
# Download metadata catalogues as a first step
force-level1-csd -u path/to/data
```

#### Parameterization

![Parameters](media/force_eo_basic_syntax_level_1.png)

##### Mandatory arguments

`metadata-dir`: Directory where metadata is stored (CSV file)

`level-1-datapool`: An existing directory. The files will be stored here

`queue`: Queue files that will be downloaded

`aoi`: Area of interest with:

+ (1) User-supplied coordinates. The polygon must be closed (fist (X,Y) and last (X,Y) are equal). (X,Y) must be given as decimal degrees with negative values for west and South coordinates. Either specify the path to a file (one coordinate pair per line?) or the coordinates on the command-line.

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
force-parameter /data/force/param/l2ps.prm LEVEL2
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

A DEM is necessary in l2ps to:

- Enhanced cloud and cloud shadow detection,
- Permorn Atmospheric correction, and to
- perform the topographic correction.

