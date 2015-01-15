
## Datasets

[Mass Census Blocks](http://www.mass.gov/anf/research-and-tech/it-serv-and-support/application-serv/office-of-geographic-information-massgis/datalayers/census2010.html)

[Corner stores](https://data.cityofboston.gov/dataset/Corner-Stores/4vcu-nshu)

## SQL in the Editor

### Applied Queries

These can be thought of like temporary filters. As long as yo apply them in the Editor (or later in CartoDB.js) the result is available for your maps, but your data on disc doesn't change. Examples:

**Only Blocks east of -71.90**

```sql
SELECT * FROM census2010blocks_poly 
 WHERE 
  ST_XMin(the_geom) > -71.90
``` 

**Only Blocks within 30Km of Boston center**

```sql
SELECT * FROM census2010blocks_poly 
 WHERE 
  ST_DWithin(the_geom, CDB_LatLng(42.358, -71.064), 30000, true)
``` 

They don't have to be geospatial

**Only Blocks with population > 100**

```sql
SELECT * FROM census2010blocks_poly 
 WHERE pop100_re > 100
```

### Results from analysis

You can create new values on the fly through SQL analysis, joins, and functions. Examples:

```sql
SELECT *, 
 ST_Distance(the_geom, CDB_LatLng(42.358, -71.064)) d 
FROM 
 census2010blocks_poly
```

Now, we can make a map with the resulting data without actually storing it on disc,

![Imgur](http://i.imgur.com/ZKsAqCP.png)

#### Analysis across tables

Here, let's take a look at measureing the distance of the closest fire station to each census block.

```sql
SELECT *, 
 (SELECT ST_Distance(the_geom, c.the_geom) FROM firestations_pt_mema 
  ORDER BY the_geom <-> c.the_geom 
  LIMIT 1) d 
FROM 
 census2010blocks_poly c
```

### Modifying data

Modifying data is the same as any database system. Simple INSERT, UPDATE, DELETE statements. 

**Drop all records not within 0.20 degree of Boston**

```sql
DELETE FROM census2010blocks_poly 
 WHERE NOT ST_DWithin(the_geom, CDB_LatLng(42.358, -71.064), 0.20); DELETE FROM firestations_pt_mema 
 WHERE NOT ST_DWithin(the_geom, CDB_LatLng(42.358, -71.064), 0.20)
``` 

_Notice we chained statements together here using the semicolon. This only works on write/deletes, a select chained together won't show up on the map_

These changes are now all permanent! No undo for deleting data!

We can also store the results in a table by simply creating a new column and using an **UPDATE** query. 

```sql
UPDATE 
 census2010blocks_poly 
SET 
 firestation_distance = 
  ST_Distance(the_geom,
  (SELECT 
   the_geom
   FROM firestations_pt_mema 
   ORDER BY the_geom <-> census2010blocks_poly.the_geom 
   LIMIT 1)
  )
```

## SQL API

The SQL gives you complete access to all the same functionality that you had in the editor. Primarily reading public datasets through SELECT statements. But also,

 * SELECT of private data through authenticated requests
 * authenticated INSERT
 * authenticated DELETE
 * authenticated UPDATE

Example anonymous SELECT

[http://andrew.cartodb.com/api/v2/sql?q=SELECT%20*%20FROM%20firestations_pt_mema%20LIMIT%2010](http://andrew.cartodb.com/api/v2/sql?q=SELECT%20*%20FROM%20firestations_pt_mema%20LIMIT%2010)

**Get the 10 closest stations to my location**

```sql 
SELECT * FROM firestations_pt_mema ORDER BY the_geom <-> my_location LIMIT 10
```

[SELECT * FROM firestations_pt_mema ORDER BY the_geom <-> CDB_LatLng(42.358, -71.064) LIMIT 10](http://andrew.cartodb.com/api/v2/sql?q=SELECT%20*%20FROM%20firestations_pt_mema%20ORDER%20BY%20the_geom%20%3C-%3E%20CDB_LatLng(42.358,%20-71.064)%20LIMIT%2010)

**include a rank number**

```sql
SELECT row_number() OVER () as rank, * FROM 
(SELECT * FROM firestations_pt_mema ORDER BY the_geom <-> CDB_LatLng(42.358, -71.064)) q
```

## CartoDB.js

The CartoDB.js library gives you a clean wrapper around the various APIs of CartoDB. For most users, that means that it allows them to take a map they have designed in the editor and integrate it in their website. It can be a lot of other things though. For example, searching a text column using the SQL API for an [autocomplete form element](http://bl.ocks.org/javisantana/7932459). Let's look at how SQL can be used with the map though.

### Map in four lines

```js
      function main(){
      	cartodb.createVis('map','{visurl}')
      }
      window.onload = main;
```

### Adding SQL to filter layers

```js
      function main(){

		var layer = L.tileLayer('http://{s}.basemaps.cartocdn.com/dark_nolabels/{z}/{x}/{y}.png',{
		  attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, &copy; <a href="http://cartodb.com/attributions">CartoDB</a>'
		});

		var map = L.map('map', {
		    scrollWheelZoom: false,
		    center: [42.358, -71.064],
		    zoom: 11
		}).addLayer(layer);


		var lat = 42.358,
		    lng = -71.064;

        cartodb.createLayer(map, {
          user_name: 'andrew',
          type: 'cartodb',
          sublayers: [{
             sql: 'select *, ST_Distance(the_geom, CDB_LatLng('+lat+','+lng+')) d from census2010blocks_poly',
             cartocss: '#layer { polygon-fill: #F00; polygon-opacity: 0.3; line-color: #F00; [d > 0.1] {polygon-fill: #70F; line-color:#70F;} }'
          }]
        })
        .addTo(map)
      }
      window.onload = main;
```
