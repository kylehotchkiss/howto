## Generate GeoJSON of interstates in USA

Useful for projects that need to bind to roads. Since there are 2.5 millon miles of roads in the USA, we'll limit to interstate highways.

1. Run `brew install gdal`
2. Go to [Natural Earth](http://www.naturalearthdata.com/downloads/) website and download all the shapefiles. 
3. Open downloaded zip, and copy `10m_cultural/ne_10m_roads_north_america` `.dbx`, `.shp`, and `.shx` files  to your working directory.
4. Run 
```
ogr2ogr \
-f GeoJSON \
-where "country = 'United States' AND class = 'Interstate'" \
roads.geojson \
ne_10m_roads_north_america.shp
```
5. Your GeoJSON is ready in `roads.geojson`. You can verify by uploading to http://geojson.io.
