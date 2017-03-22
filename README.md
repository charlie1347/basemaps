# Basemaps in matplotlib/bokeh

Matplotlib's "basemap" ability allows the option of using shapefiles to add basemaps to your plots. When experimenting with this function, I ran into the following error:

~~~~
ValueError: shapefile must have lat/lon vertices  - it looks like this one has vertices
in map projection coordinates. You can convert the shapefile to geographic
coordinates using the shpproj utility from the shapelib tools
(http://shapelib.maptools.org/shapelib-tools.html)
~~~~

Some googling turned up a post stating that GDAL's ogr2ogr could handle these conversions, as opposed to figuring out how to use the shapelib tools.

The first requirement is to set up a working version of GDAL on your computer - I compiled this from source in Linux. Once I had GDAL working, I used ogr2ogr to convert my shapefile. ogr2ogr contains documentation on how to run it from the command line [here](http://www.gdal.org/ogr2ogr.html). Alternatively, I experimented with the ogr2ogr Python module, taken from [here](http://svn.osgeo.org/gdal/trunk/gdal/swig/python/samples/ogr2ogr.py). The Python module takes roughly the same syntax as the command line ogr2ogr, with individual arguments passed as a list.

The first shapefile I was working with was a shapefile of all the buildings in London, TQTL_LON.shp, downloaded from [here](https://docs.google.com/uc?id=0B0kRhiw4fD7uQzU3MzBMRzFfSFk&export=download). Looking at the original TQTL_LON.prj file, the shapefile is using the British National Grid projection - ogr2ogr is smart enough to figure this out on its own. ogr2ogr can also clip your plots to just a specific area by feeding it a bounding box of coordinates. 

~~~~
ogr2ogr.main(["", "-f", "ESRI Shapefile", 
                "-clipdst", "-0.237947", "51.449814", "0.007965", "51.553407",
                "Converted basemaps/London_buildings.shp", 
                "Raw basemaps/TQTL_LON.shp", 
                "-t_srs", "EPSG:4326"])
~~~~

This code takes the original "TQTL_LON.shp" shapefile, clips it to a bounding box, and reprojects it into a new ESRI Shapefile called "London_buildings.shp", using the coordinate system EPSG:4326 (also known as WGS84). 

I took a second shapefile, TQ_Roadlink.shp, from the Ordnance Survey website [here](https://www.ordnancesurvey.co.uk/opendatadownload/products.html#OPROAD), this time of all of the UK's roads. Running the above code on this shapefile however produced the following error:

~~~~
ValueError: readshapefile can only handle 2D shape types
~~~~

A simple fix - ogr2ogr can deal with this by forcing the coordinate dimensions (from XYZ to just XY). 

~~~~
ogr2ogr.main(["", "-f", "ESRI Shapefile",
                "-clipdst", "-0.237947", "51.449814", "0.007965", "51.553407",
                "Converted basemaps/London_roads.shp",
                "Raw basemaps/TQ_RoadLink.shp",
                "-t_srs", "EPSG:4326",
                "-dim", "2"])
~~~~

Once I had converted both shapefiles, matplotlib could then easily plot both as basemaps (see ipynb for full code):

# Buildings

![Matplotlib buildings](/buildings-matplotlib.PNG)

# Roads

![Matplotlib roads](/roads-matplotlib.PNG)

# Bokeh

To add basemaps onto bokeh plots, I needed to convert the shapefiles into geoJSON format. Again, ogr2ogr can handle this:

~~~~
ogr2ogr.main(["", "-f", "GeoJSON", 
                "-clipdst", "-0.237947", "51.449814", "0.007965", "51.553407",
                "Converted basemaps/London_buildings.geojson", 
                "Raw basemaps/TQTL_LON.shp", 
                "-t_srs", "EPSG:4326"])
~~~~

Once the shapefiles are in a geoJSON format, I used bokeh's GeoJSONDataSource to add them both as "patches" to my bokeh plots (again, see ipynb file for full code):

![Bokeh](/bokeh.PNG)
