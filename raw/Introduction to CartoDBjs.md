## Part 1 - Data

##### US Drought Monitor data

http://droughtmonitor.unl.edu/MapsAndData/GISData.aspx

Look for ```2014-09-02    SHP```

tablename becomes, ```usdm_20140902```. let's rename it to ```usdm_september_2014```

## Part 2 - Explore the interface

### Creating map types

* Simple 
* Choropleth
* Category

### Publishing a map

Using visualizations and a quick look at options

#### A couple of tricks

* map versions
* Limiting a visualization to a single zoom
* Publishing datasets
* get simple url

## Part 3 - CartoCSS

http://droughtmonitor.unl.edu/MapsAndData/Metadata.aspx#color

*  A detour to [gas service](https://dl.dropboxusercontent.com/u/1307405/maps-material/class1/gas_stations.zip)
*  A detour to Citibike example

#### Advanced: Zoom based styles

Example with http://nijel.org/nysewage/ssoreports.csv

```css
    #ssoreports [ zoom = 4] {
       marker-fill:  white;
       marker-opacity: 0.4;
    }
```

#### Making legends match

Use the legend interface to update our colors and labels

## Part 4 - Shapefiles

* Download US States from
http://www.nationalatlas.gov/atlasftp.html?openChapters=chpbound#chpbound
* How to replace missing PRJ (using guess from USDM file)

## Part 5 - Spatial SQL

A list of some of my favorites: https://gist.github.com/andrewxhill/7862128

For us, we need a question to answer. Let's find the proportion of California covered by D5 drought. 

http://droughtmonitor.unl.edu/AboutUs/ClassificationScheme.aspx

#### Filter our States

```sql
SELECT * FROM statesp020 LIMIT 1
```

```sql
SELECT * FROM statesp020 WHERE cartodb_id = 1729
```

```sql
SELECT * FROM statesp020 WHERE state = 'California'
```

```sql
SELECT * FROM statesp020 WHERE state = 'California' LIMIT 1
```

Eeek, this dataset has a row for every polygon in California! So, let's create a dataset where we merge each state into a single row.

```
SELECT ST_Union(ST_MakeValid(the_geom)) the_geom, state FROM statesp020  GROUP BY state 
```

Call it **us_states**

### Intersection

http://en.wikipedia.org/wiki/Portland,_Oregon

```sql 
SELECT * FROM usdm_september_2014 WHERE ST_Intersects(the_geom, CDB_LatLng(45.52, -122.681944))
```

##### Subqueries

```sql 
SELECT * FROM usdm_september_2014 WHERE ST_Intersects(the_geom, (SELECT the_geom FROM us_states WHERE state='California'))
```


#### Side track: Writing points into a polygon

```sql
UPDATE target_table SET some_value = (SELECT count(*) FROM source_table WHERE ST_Intersects(the_geom, target_table.the_geom)
```

Let me show you! **live demo**
 
##### When to use the_geom_webmercator

**the_geom** is great for measuring, filtering, and transforming. this is lat/lng in wgs84

**the_geom_webmercator** is needed when you want to show the result on the map. this is webmercator (srid = 3857) 

##### Dynamic result 

```sql
SELECT cartodb_id, dm, ST_Intersection(the_geom, (SELECT the_geom FROM us_states WHERE state='California')) the_geom FROM usdm_september_2014 
```

Ah, no **the_geom_webmercator** so nothing appears on the map!

```sql
SELECT cartodb_id, dm, ST_Transform(ST_Intersection(the_geom, (SELECT the_geom FROM us_states WHERE state='California')),3857) the_geom_webmercator FROM usdm_september_2014
```

Whoa!

### Measurements

```sql
SELECT ST_Area(the_geom) FROM us_states WHERE state='California'
```

The result is a meaningless unit of measurement here

```sql
SELECT (ST_Area(the_geom::geography)/1000000)::int FROM us_states WHERE state='California'
```

##### Store the result for later

Create a new column in ```us_states``` called, area_sqkm. Now run,

```sql
UPDATE us_states SET area_sqkm = (ST_Area(the_geom::geography)/1000000)::int
```

## Part 6 - CartoDB.js

#### A quick look at cartodb.js

#### A quick look at odyssey.js

### A starting point

You can download a blank template here

[Demo template](https://dl.dropboxusercontent.com/u/1307405/maps-material/class2/workshop/index.html.zip)

There isn't much in there but some style and a few elements for our page. We'll fill in all the rest! 

### Setting up our map

#### Basemaps from CartoDB

```js
        L.tileLayer(some url to a basemap).addTo(map);
```

https://github.com/CartoDB/cartodb/wiki/BaseMaps-available

'http://{s}.api.cartocdn.com/base-midnight/{z}/{x}/{y}.png'

#### Loading our layer

We are going to use the [createLayer](http://docs.cartodb.com/cartodb-platform/cartodb-js.html#cartodbcreatelayermap-layersource--options--callback). It allows a bit more customization over the [createVis](http://docs.cartodb.com/cartodb-platform/cartodb-js.html#cartodbcreatevismapid-vizjsonurl-options--callback) but takes a bit more setup. createVis for example, wouldn't need us to rechoose a basemap, it would have pulled it right from our Editor style

Here is what we'll use

```js
        // Use createLayer to add our layer to the map
        // See http://docs.cartodb.com/cartodb-platform/cartodb-js.html
        cartodb.createLayer(map, 'your viz.json url')
        .addTo(map)
        .done(function(layer) {
          // run the getStats here

          // put radio listener here
        });
```

##### Adding legends and other options

See the options portion of the [createLayer](http://docs.cartodb.com/cartodb-platform/cartodb-js.html#cartodbcreatelayermap-layersource--options--callback) docs

```js
{legends: true}
```

### Load more data to use

Delete your original states table (with the unmerged polygons)

http://droughtmonitor.unl.edu/MapsAndData/GISData.aspx

* usdm_september_2013 
* usdm_september_2012 
* usdm_september_2011

### Add to our interface

First, we'll add some simple text to our webpage

```html
        <p>
          For September <span id="year">2014</span> in California the <a href="http://droughtmonitor.unl.edu/MapsAndData/GISData.aspx">United States Drought Monitor</a> reported: 
        </p>
        <p>
          <span id="total_percent" class="stat">0</span>% of California is experiencing a drought. 
        </p>
        <p>
          <span id="dm4_percent" class="stat">0</span>% of California is experiencing <a href="http://droughtmonitor.unl.edu/AboutUs/ClassificationScheme.aspx">exceptional drought</a>. 
        </p>
```

Next, we'll add an interactive menu to toggle different years

```html
        <h3 class="mid-title">Select a September</h3>

        <form>
          <fieldset class="radioset">
            <ul>
              <li><input type="radio" name="radiox" id="radio1" value="2011"><label for="radio1">2011</label>
              <li><input type="radio" name="radiox" id="radio2" value="2012"><label for="radio2">2012</label>
              <li><input type="radio" name="radiox" id="radio3" value="2013"><label for="radio3">2013</label>
              <li><input type="radio" name="radiox" id="radio4" value="2014" checked><label for="radio4">2014</label>
            </ul>
          </fieldset>
        </form>
```
#### Custom Javascript

We want to add a value to the webpage (not a map) based on the results of some query. For example, I want to know the proportion of California covered by a DM4 drought. That query would look like this,

```sql
SELECT 
    round(
        100*sum(
            (ST_Area(
                ST_Intersection(u.the_geom, s.the_geom)::geography
            )/1000000) / s.area_sqkm
        )::numeric,2
    ) percent 
FROM 
    us_states s, 
    usdm_september_2014 u 
WHERE 
    s.state = 'California' 
AND 
    dm = 4
```

We can run that in our Javascript with somethign simple, like,


```js
        var sql = cartodb.SQL({ user: username });
        sql.execute(our_query_above).done(function(data) {
            console.log(data)
        })
```

Here, I've created a function ```getStats``` that does that plus a bit more. It runs to queries, one that gets the total percent of california experiencing drought, the second just experiencing dm=4. It then writes the results to our webpage in placeholders we added above.

```js
      function getStats(year){
        var sql = cartodb.SQL({ user: username });
        // queryAll is the SQL necessary to measure the proportion of all of 
        // California in drought
        // we are swapping in our year value on the fly here
        var queryAll = "SELECT round(100*sum((ST_Area(ST_Intersection(u.the_geom, s.the_geom)::geography)/1000000) / s.area_sqkm)::numeric,2) percent FROM us_states s, usdm_september_"+ year +" u WHERE s.state = 'California' ";
        // queryAll is the SQL necessary to measure the proportion of all of 
        // California in extreme drought
        // we are swapping in our year value on the fly here
        var queryDM4 = "SELECT round(100*sum((ST_Area(ST_Intersection(u.the_geom, s.the_geom)::geography)/1000000) / s.area_sqkm)::numeric,1) percent FROM us_states s, usdm_september_"+ year +" u WHERE s.state = 'California' AND dm = 4";

        // This sets the year into the text description
        $('#year').html(year);

        // Here we will measure the drought for all CA
        sql.execute(queryAll).done(function(data) {
          // this uses jquery to rewrite the percent value
          $('#total_percent').html(data.rows[0].percent);
        })

        // Here we will measure the drought for CA with dm=4
        sql.execute(queryDM4).done(function(data) {
          // this uses jquery to rewrite the percent value
          $('#dm4_percent').html(data.rows[0].percent);
        }) 
      }
```

##### Run a function AFTER layer load 

We can run the ```getStats``` function just after our map is displayed. 

```js
          // initiate our stats with year = 2014
          // this is a silly little custom function
          getStats(2014);
```


#### Listen to user events on the HTML

Now we have a nice map and a little web interface. Next, we want to allow our users to see results for the 4 different years by simply clicking our radio buttons. 

Here, I've written a function that will listen to the radio. When it is clicked, it will get the year of the radio clicked and rerun ```getStats``` with the new year!

```js
          // create a listener for our year selector radio
          // this is jquery to listen to the radio click event
          $('input:radio[name="radiox"]').change(function(){
            var year = $(this).val();
            getStats(year);
            // Add layer update here

          });
```

#### Toggle layers, change style, and change sql

There are various ways to update your map. There are a lot of examples online of how to [toggle layers](http://bl.ocks.org/andrewxhill/10506396) and for [updating styles](http://bl.ocks.org/andrewxhill/8324313) using CartoDB.js. In our case, we actually want to change the map in a different way.

When a user clicks on a year, we want the data to change, but the style to remain the same. In this case, we can use setSQL to update the SQL query being run on the layer, selecting new data. In this case, we'll select from a totally new table!

```js
            layer.getSubLayer(0).setSQL("SELECT * FROM usdm_september_"+ year);
```

## Publish our little map on bl.ocks

I love to publish my map demos on [bl.ocks](bl.ocks.org ). If you don't know how, let's do it real quick!

# Now. Something crazy

Starting template: https://dl.dropboxusercontent.com/u/1307405/maps-material/class2/workshop/rainbow.html.zip

Starting map: https://team.cartodb.com/u/andrew/viz/75435924-3d33-11e4-818b-0e73339ffa50/map

Let's change it,

```js
layer.getSubLayer(0).setCartoCSS("#gilt_postal_codes{ polygon-fill: #FF2900; polygon-opacity: 0.7; line-width: 0; }");
```

A helper function (via [here](http://stackoverflow.com/questions/11866781/how-do-i-convert-an-integer-to-a-javascript-color))

```js
function toColor(num) {
    num >>>= 0;
    var b = num & 0xFF,
        g = (num & 0xFF00) >>> 8,
        r = (num & 0xFF0000) >>> 16;
    return "rgba(" + [r, g, b].join(",") + ")";
}
```

And, get the time of day

```js
var milliseconds = (new Date).getTime();
console.log(milliseconds);
```

Make it our polygon color

```js
var color = toColor(milliseconds);
layer.getSubLayer(0).setCartoCSS("#gilt_postal_codes{ polygon-fill: "+color+"; polygon-opacity: 0.7; line-width: 0; }");
```

How about adding a rule,

```js
var color = toColor(milliseconds);
layer.getSubLayer(0).setCartoCSS("#gilt_postal_codes{ polygon-fill: black; [state='CO']{polygon-fill: "+color+";} polygon-opacity: 0.7; line-width: 0;}");
```

Or another variation

```js
layer.getSubLayer(0).setCartoCSS("#gilt_postal_codes{ polygon-fill: black; line-width: 0; [state='CO']{line-width: 0.5; line-color: "+color+";} polygon-opacity: 0.7; }");
```

### An alternative

```js
      function hour2gray( hour ) {
          var color_part_dec = 255 * (hour / 24.0);
          var color_part_hex = Number(parseInt( color_part_dec , 10)).toString(16);
          return "#" + color_part_hex + color_part_hex + color_part_hex;
      }
```

and 

```js
                    var hours = (new Date).getHours();
                    var color = hour2gray(hours);
                    layer.getSubLayer(0).setCartoCSS("#gilt_postal_codes{ polygon-fill: green; [pop_total > 100000] { polygon-fill: "+color+";} polygon-opacity: 0.7; }");
```
