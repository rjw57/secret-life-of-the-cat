# Extracting the GPS traces from Horizon's Secret Life of the Cat

The BBC's Horizon programme recently had a special entitled [The Secret Life of
the Cat](http://www.bbc.co.uk/programmes/b02xcvhw) which had an associated
[interactive map](http://www.bbc.co.uk/news/science-environment-22567526) on
the BBC website. A colleague at work expressed interest in getting the GPS
trace data.

Unfortunately the [research
project](http://www.rvc.ac.uk/SML/People/awilson/BBC-Horizon-the-secret-life-of-the-cat.cfm)
at the Royal Vetinary College does not make the GPS traces available on it's
website. This repository contains my efforts to re-engineer the GPS traces from
the BBC's interactive map. The results can be seen in the `geojson/` and `kml/`
directories. GitHub now has a direct visualisation feature which means that the
GeoJSON files can be viewed inline on a map.

In addition, I've uploaded the KML files generated to [a custom Google
map](https://www.google.co.uk/maps/ms?msid=205632188049427480820.0004df1e4c44ec8afff9f&msa=0&ll=51.184817,-0.527945&spn=0.006173,0.016512).

## Scraping the website

The excellent Developer Console for Google Chrome revealed that the bulk of the
mapping work on the site was being done in
http://news.bbcimg.co.uk/news/special/2013/newsspec_5380/js/compiled/desktop/ns_all.js.
After beautifying this file and poking around for some data, we can find some
Javascript which creates an object which is in `raw-website-data.json` in this
repository.

This JSON object contains a single array of cat records which itself contains
the traces along with timestamps. Success! Well, not quite. The trace positions
appear to be in pixel co-ordinates on the detail maps. Fail. Well, not quite.
The detail maps themselves are fairly easily exctracted, see the `imagery/`
directory. Using my own [foldbeam](https://github.com/rjw57/foldbeam) program,
I downloaded a GeoTIFF of the Sharnley Green area using the following command:

```console
$ foldbeam-render -p 27700 -l 501000 -r 505000 -b 142000 -t 146000 -w 4000 -o imagery/road.tiff
```

This file is available in the `imagery/` directory in this repository. It is
the Sarnley Green area. The GeoTIFF is rendered by Foldbeam using OpenStreetMap
tiles but re-projected into the British Ordinance Survey Grid. Using the GIMP,
I then manually aligned the maps as shown in `imagery.road.xcf`. Ah, but the
individual map images are of the wrong scale. Luckily if you look at the raw
website data you see that the number of pixels per metre for each map is
recorded. This can be used, as shown in the `reprojection.ipynb` IPython web
notebook, to calculate the appropriate scaling for each layer.

After manual alignment, the location of the maps in the road GeoTIFF is
recorded. Some simple manipulation outlines in `reprojection.ipynb` can then be
used to convert pixel locations in the maps to pixel locations in the
OpenStreetMap road image. Since Foldbeam saves a GeoTIFF, we can use the Python
[GDAL](http://gdal.org/) wrapper to load the road image in and get the
transform from pixel co-ordinates straight into Ordinance Survery grid
co-ordinates.

We're nearly there. GeoJSON files by default use
[WGS84](https://en.wikipedia.org/wiki/World_Geodetic_System) latitudes and
longitudes for storing positions. We again use the Python wrappers to transfrom
from one projection to the next. At this point it's a simple job to write out a
little GeoJSON file for each cat.

The [OGR](http://gdal.org/ogr) library comes with a handy little utility that
can be used to convert the GeoJSON files into KML files which can be loaded
into Google Earth or onto Google Maps. They're generated by a single command:

```console
$ for i in *.geojson; do ogr2ogr -f KML ../kml/`basename $i geojson`kml $i; done
```

And... we're done.

## Issues

Unfortunately, I couldn't work out where Orlando, the cat whose map is an
insert, is based. If anyone wants to send me a pull-request fixing the map
location, I'd be more than happy to accept it.

