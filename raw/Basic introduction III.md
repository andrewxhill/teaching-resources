## Creating your accounts

Set you up with accounts here, [signup](https://cartodb.com/signup).

## What is CartoDB?

* A quick look at what CartoDB is and how people use it

## Let's take a look

* Tour of dashboard
* Common data
* Uploading data
* Tour of table and map view
* Publishing maps
* Public profile

## Your first map in 45 seconds!

Our first dataset will be the Cambridge roof polygons! [Go here](http://www.mass.gov/anf/research-and-tech/it-serv-and-support/application-serv/office-of-geographic-information-massgis/datalayers/ftpstructures.html) but don't download it yet.

Let's look at how to import from a URL.

#### Looking around

* Table and map view
* Styling data
* Thematic maps!

**Onward!**

Let's do the exact same thing with [Somerville](http://www.mass.gov/anf/research-and-tech/it-serv-and-support/application-serv/office-of-geographic-information-massgis/datalayers/ftpstructures.html).

## You can SQL

Let's join the data using a ```UNION``` function

```sql
SELECT 
  the_geom, area_sq_ft, local_id, moved, sourcedata, sourcedate, sourcetype, struct_id, town_id 
FROM 
  structures_poly_49
UNION ALL
SELECT 
  the_geom, area_sq_ft, local_id, moved, sourcedata, sourcedate, sourcetype, struct_id, town_id 
FROM 
  structures_poly_274
```

Now, use the Options menu to ```Create table from query``` and call it ```neighborhood```

![from query](http://i.imgur.com/zifw6tU.png)

## Our third dataset

Let's go grab the shapefile for [Boston libraries](http://www.mass.gov/anf/research-and-tech/it-serv-and-support/application-serv/office-of-geographic-information-massgis/datalayers/libraries.html) and import it into our accounts.

#### Write values into a new column

Create a new column in our ```neighborhood``` table called ```library```

![library](http://i.imgur.com/aM5wjkW.png)

Now, we can write values into that column using relational SQL!

```
UPDATE 
  neighborhood 
SET 
  library = 
    (
      SELECT 
        ST_Distance(the_geom, neighborhood.the_geom, true) 
      FROM 
        libraries_pt 
      ORDER BY 
        the_geom <-> neighborhood.the_geom 
      LIMIT 1
    )
```

## Bonus: Transit data

Grab [MBTA stations](http://www.mass.gov/anf/research-and-tech/it-serv-and-support/application-serv/office-of-geographic-information-massgis/datalayers/mbta.html)

Add new column in ```neighborhoods``` called ```mbta```

Run the distance calculation again

```sql
UPDATE 
neighborhood 
SET 
mbta = (
  SELECT 
  ST_Distance(the_geom, neighborhood.the_geom, true) 
  FROM mbta_node 
  ORDER BY the_geom <-> neighborhood.the_geom 
  LIMIT 1
  )
```

## Publishing a multilayer map

Create a *Visualization*

![vis](http://i.imgur.com/GbXcjMr.png)

## Publishing maps

* Visualizations
* Multilayer map
* Layout
* Publishing options
