Data: [Denver Traffic Accidents](http://data.denvergov.org/dataset/city-and-county-of-denver-traffic-accidents)

Go to your ```traffic_accidents``` table and let's start by creating a new visualization

![Imgur](http://i.imgur.com/IZC4upz.png)

## Overview of the Torque Wizard

*key things*

1. When does it require new server requests?

2. Overview of ```duration```, ```steps```, ```trails```, ```resolution```

## Torque Category

![Imgur](http://i.imgur.com/HT39DAI.png)

## Advanced Torque

#### frame-offset

```css
#traffic_accidents[frame-offset=2] {
 marker-width:10;
 marker-fill-opacity:0.225; 
}
```

Links directly with ```-torque-frame-count:512;```

#### The ```Map``` element

```css
Map {
-torque-frame-count:512;
-torque-animation-duration:30;
-torque-time-attribute:"cartodb_id";
-torque-aggregation-function:"CDB_Math_Mode(torque_category)";
-torque-resolution:2;
-torque-data-aggregation:linear;
}
```

#### The aggregation-function

It's SQL embedded in CartoCSS!

```sql
CDB_Math_Mode(district_i)
-- or
count(*)
-- or
sum(income)
```

##### Using the stored values of aggregation for style

```css
#traffic_accidents[value=1] {
   marker-fill: #A6CEE3;
}
```

#### Categorical maps

![Imgur](http://i.imgur.com/pJfOPLL.png)

Categorical maps bring together the aggregate function and ```CDB_Math_Mode``` to create predictable ```values``` for styling. Let's take a look

```css
Map {
-torque-frame-count:512;
-torque-animation-duration:30;
-torque-time-attribute:"cartodb_id";
-torque-aggregation-function:"CDB_Math_Mode(district_i)";
-torque-resolution:2;
-torque-data-aggregation:linear;
}

#traffic_accidents{
  comp-op: source-over;
  marker-fill-opacity: 0.9;
  marker-line-color: #FFF;
  marker-line-width: 1.5;
  marker-line-opacity: 1;
  marker-type: ellipse;
  marker-width: 6;
  marker-fill: #FF9900;
}
#traffic_accidents[frame-offset=1] {
 marker-width:8;
 marker-fill-opacity:0.45; 
}
#traffic_accidents[frame-offset=2] {
 marker-width:10;
 marker-fill-opacity:0.225; 
}
#traffic_accidents[value=1] {
   marker-fill: #A6CEE3;
}
#traffic_accidents[value=2] {
   marker-fill: #1F78B4;
}
#traffic_accidents[value=3] {
   marker-fill: #B2DF8A;
}
#traffic_accidents[value=4] {
   marker-fill: #33A02C;
}
#traffic_accidents[value=5] {
   marker-fill: #FB9A99;
}
#traffic_accidents[value=6] {
   marker-fill: #E31A1C;
}
#traffic_accidents[value=7] {
   marker-fill: #FDBF6F;
}
```

#### Bubble maps

Values are calculated *within* each geotemporal bin. So, the combination of ```-torque-frame-count``` and ```-torque-resolution```

![Imgur](http://i.imgur.com/ncv5dhp.png)

Let's take a look at one to start,

```css
Map {
-torque-frame-count:256;
-torque-animation-duration:30;
-torque-time-attribute:"first_occu";
-torque-aggregation-function:"count(cartodb_id)";
-torque-resolution:2;
-torque-data-aggregation:linear;
}
#traffic_accidents{
  comp-op: lighter;
  marker-fill-opacity: 0.9;
  marker-line-color: #FFF;
  marker-line-width: 0;
  marker-line-opacity: 1;
  marker-type: ellipse;
  marker-width: 1;
  marker-fill: #F84F40;
}
#traffic_accidents[value>2] {
 marker-width:2;
}
#traffic_accidents[value>3] {
 marker-width:4;
}
#traffic_accidents[value>4] {
 marker-width:8;
}
#traffic_accidents[frame-offset=1] {
 marker-width:2.5;
 marker-fill-opacity:0.2; 
}
#traffic_accidents[frame-offset=2] {
 marker-width:4.5;
 marker-fill-opacity:0.1; 
}
```

##### Discrete continuous bubbles

![Imgur](http://i.imgur.com/wGM7xdJ.png)

A look at the ```Map``` element

```css
Map {
-torque-frame-count:24;
-torque-animation-duration:30;
-torque-time-attribute:"first_occu";
-torque-aggregation-function:"count(cartodb_id)";
-torque-resolution:16;
-torque-data-aggregation:linear;
}
```

And setting up the bubbles

```
#traffic_accidents{
  comp-op: lighter;
  marker-fill-opacity: 0.9;
  marker-line-color: #FFF;
  marker-line-width: 0;
  marker-line-opacity: 1;
  marker-type: ellipse;
  marker-width: 1;
  marker-fill: #F84F40;
}
#traffic_accidents[value>3] {
 marker-width:2;
}
#traffic_accidents[value>5] {
 marker-width:3;
}
#traffic_accidents[value>7] {
 marker-width:6;
}
#traffic_accidents[value>10] {
 marker-width:8;
}
```

##### Torque-Zero

Back to the category map

```css
Map {
-torque-frame-count:1;
-torque-animation-duration:0;
-torque-time-attribute:"cartodb_id";
-torque-aggregation-function:"CDB_Math_Mode(district_i)";
-torque-resolution:1;
-torque-data-aggregation:linear;
}

#traffic_accidents{
  comp-op: source-over;
  marker-fill-opacity: 0.9;
  marker-line-color: #FFF;
  marker-line-width: 1.5;
  marker-line-opacity: 1;
  marker-type: ellipse;
  marker-width: 6;
  marker-fill: #FF9900;
}
#traffic_accidents[value=1] {
   marker-fill: #A6CEE3;
}
#traffic_accidents[value=2] {
   marker-fill: #1F78B4;
}
#traffic_accidents[value=3] {
   marker-fill: #B2DF8A;
}
#traffic_accidents[value=4] {
   marker-fill: #33A02C;
}
#traffic_accidents[value=5] {
   marker-fill: #FB9A99;
}
#traffic_accidents[value=6] {
   marker-fill: #E31A1C;
}
#traffic_accidents[value=7] {
   marker-fill: #FDBF6F;
}
```

Better control over intensity/heat maps

![Imgur](http://i.imgur.com/evH2Uiq.png)

```css
Map {
-torque-frame-count:1;
-torque-animation-duration:0;
-torque-time-attribute:"cartodb_id";
-torque-aggregation-function:"count(district_i)";
-torque-resolution:1;
-torque-data-aggregation:linear;
}

#traffic_accidents{
  comp-op: source-over;
  marker-fill-opacity: 0.9;
  marker-line-color: #FFF;
  marker-line-width: 0;
  marker-line-opacity: 1;
  marker-type: ellipse;
  marker-width: 2;
  marker-fill: #FFCC00;
}
#traffic_accidents[value>1] {
   marker-fill: #FFA300;
}
#traffic_accidents[value>2] {
   marker-fill: #FF9900;
}
#traffic_accidents[value>3] {
   marker-fill: #FF5C00;
}
#traffic_accidents[value>4] {
   marker-fill: #FF6600;
}
#traffic_accidents[value>5] {
   marker-fill: #FF2900;
}
```

#### Animated grids (bonus)

Create new column ```offense_n```, type ```number```

![Imgur](http://i.imgur.com/XDil6Wi.png)

Run the following update:

```sql
UPDATE traffic_accidents SET offense_n = 1 WHERE offense_ty = 'traf-vehicular-homicide';
UPDATE traffic_accidents SET offense_n = 2 WHERE offense_ty = 'traffic-accident-dui-duid';
UPDATE traffic_accidents SET offense_n = 3 WHERE offense_ty = 'traf-vehicular-assault';
UPDATE traffic_accidents SET offense_n = 4 WHERE offense_ty = 'traffic-accident';
UPDATE traffic_accidents SET offense_n = 5 WHERE offense_ty = 'traffic-accident-hit-and-run';
```

And the header:

```css
Map {
-torque-frame-count:64;
-torque-animation-duration:30;
-torque-time-attribute:"first_occu";
-torque-aggregation-function:"CDB_Math_Mode(offense_n)";
-torque-resolution:16;
-torque-data-aggregation:linear;
}

#traffic_accidents{
  comp-op: source-over;
  marker-fill-opacity: 0.6;
  marker-line-color: #FFF;
  marker-type: rectangle;
  marker-width: 8;
  marker-fill: #FF9900;
}
```