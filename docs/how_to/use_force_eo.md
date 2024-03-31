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

![Datacube concept](media/force_datacube.png

### Define the datacube parameters

To generate a Level 2 ARD (Analysis ready data) with `FORCE L2PS` is necessary to define the datacube in the parameter file.

Empty parameter files can be generated with `force-parameter`

`DO_REPROJ`: Indicates if images should be reprojected to the target coordinate system

`DO_TILE`: Indicates if the images should be tiled to chips that intersect with the grid system

`PROJECTION`: Defines the target coordinate system

`ORIGIN_LAT` and `ORIGIN_LONG`: Indicate the origin coordinates

`TILE_SIZE`

  

`BLOCK_SIZE`: Block size of the image chips. Block are stripes