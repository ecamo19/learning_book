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

## Level 1 cloud storage Downloader

This section covers the process of querying and downloading Landsat and Sentinel-2
level 1 data from cloud services.

Download the metadata catalogues:

```bash
# Download metadata catalogues as a first step
force-level1-csd -u path/to/data
```

### Parameterization

![Parameters](media/force_eo_basic_syntax_level_1.png)

#### Mandatory arguments

`metadata-dir`: Directory where metadata is stored (CSV file)

`level-1-datapool`: An existing directory. The files will be stored here

`queue`: Queue files that will be downloaded

`aoi`: Area of interest with:

+ (1) User-supplied coordinates. The polygon must be closed (fist (X,Y) and last (X,Y) are equal). (X,Y) must be given as decimal degrees with negative values for west and South coordinates. Either specify the path to a file (one coordinate pair per line?) or the coordinates on the command-line

+ (2) A shapefile (point/polygon/line). EPSG4326 projection recommended

+ (3) Scene identifier: Landsat (Path/Row) as PPPRRR. Make sure to keep leading zeros (i.e. 181034 not 18134).

	+ Sentinel-2 (MGRS tile) as "TXXXXX". Make sure to keep the leading T before the MGRS tile number.
	
	 + Either specify the path to a file (one ID per line) or give the ID's on the command-line. If on command line, provide a comma separated list
	 
#### Optional Arguments

+ (1) Cloud cover (-c | --cloudcover): Range between 0-100%.

+ (2) Date range (-d | --daterange): Range dates with the format YYYYMMDD,YYYYMMDD

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

+ (6) Keep metadata (-k | --keep-meta). Will write the results of the query to the level 1 datapool directory. Two files will be created if Landsat and Sentinel-2 data is queried at the same time. Filename: csd_metadata_YYYY-MM-DDTHH-MM-SS

+ (7) Check for FORCE Level-2 log files (-l | --logs). Check and and remove any products from the search that have been processed previously. Note that this only checks for the presence of log files, not for actual Level-2 products.

### Querying and downloading data for Australia
```bash
#
mkdir australia
cd australia
force-level1-csd -u 
```