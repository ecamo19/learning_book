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

  

Sentinel-2 (MGRS tile) as "TXXXXX". Make sure to keep the leading T before the MGRS tile number.

  

Either specify the path to a file (one ID per line) or give the ID's on the command-line. If on command line, provide a comma separated list