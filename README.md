# quadtree-contours

This is a JavaScript implementation of an algorithm for efficiently computing contours (isolines) of raster data such as digital elevation models, inspired primarily by [this article](https://www.mattkeeter.com/projects/contours/). Unlike the more common "marching squares" algorithm, it does not have to visit every input pixel for every contour line.

See the [geotiff-contour.js](examples/geotiff-contour.js) example for how to generate GeoJSON contour lines from a GeoTIFF elevation model.

First, it builds a min-max mipmap of the input data -- a pair of image pyramids using minimum and maximum as the reduction operators. Each higher level of the pyramid is half the resolution of the previous, and a pixel stores the min or max of the 4 corresponding pixels covering the same area at the level below. The lowest level of both the min and max pyramids is simply the original image, while the highest level is a single pixel with the minimum and maximum values found across the entire input raster.

This mipmap can be reused for all the contour lines on the same raster. Starting at the top of the pyramid, this mipmap is interpreted as a quadtree for a particular contour level value. It selects the cells of the mipmap where the contour level lies between the cell's min and max, and recursively subdivides such cells by proceeding down to more detailed mipmap levels. It recurses into cells that may contain the contour line, and traces out the boundary between cells that are entirely above the contour level and those that are entirely below. These boundary line segments are then sorted and reassembled into vector rings and lines.

This animation shows the changes to the quadtree structure as the contour level sweeps upward through an elevation model of a canyon:
![Animation](https://i.imgur.com/cQY3kaC.gif)
(generated by [this example](/examples/draw-mipmap-quadtree.js))
