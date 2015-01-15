## Brief introduction to CartoDB

* Tour of dashboard
* Common data
* Uploading data
* Tour of table and map view
* Publishing maps
* Public profile

## Creating your accounts

Set you up with accounts here, [https://cartodb.com/signup?plan=academy](https://cartodb.com/signup?plan=academy). This will give you a free upgrade to our **student** plan, since today, we are all students :)

## Your first map in 30 seconds!

Here, we'll make a map of US States. You can grab the data from Common Data in your account, you can find the link to Common Data up top.

![common data](http://i.imgur.com/sFlZNKl.png)

* Table and map view
* Styling data
* Thematic maps!

**Onward!**

## Your second map!

How to import from a URL. First right click and 'copy' [this link](https://dl.dropboxusercontent.com/u/1307405/CartoDB/workshop/severe-wind.csv.zip)

Let's customize our maps

* Basemaps
* Wizards
* Pop ups
* Publishing

## Access to full SQL!

One of the most unique features of CartoDB is that your account is actually a database and we give you full SQL control of your data! If more interesting is that the database is spatial, so you can do all sorts of analysis + mapping.

*Mini demo*

##### Hands on

* Add a new column called, `wind_events` choose `number` for type
* Use SQL to count the points in each polygon in our table!

```sql
    UPDATE 
      ne_50m_admin_1_states 
    SET 
      wind_events = (
        SELECT 
          count(*) 
        FROM 
          severe_wind 
        WHERE 
          ST_Intersects(the_geom, ne_50m_admin_1_states.the_geom)
      )
```

Let's make a map!

## Publishing maps - Windy Politicians

* Visualizations
* Labels and annotations
* Layout
* Publishing options

