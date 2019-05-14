---
title: World Map
aliases: ["/ref/charts/world-map"]
weight: 500
---

The _world map_ widget can display geographical data on a world map. Different
map projections are supported such as the standard mercator or orthographic projection.

## Input Format

| Field         | Type    | Description                                                                                                     |
|---------------|---------|-----------------------------------------------------------------------------------------------------------------|
| `weight`      | number  | A weight value in range [0;1]. This is used to scale opacity and color values.                                  |
| `latitude`    | number  | The latitude coordinate in range [-90;90] degrees. Values outside the range will wrap.                          |
| `longitude`   | number  | The longitude coordinate in range [0;360] degrees. Values outside the range will wrap.                          |
| `geohash`     | string  | OPTIONAL. A base32 [geohash](https://en.wikipedia.org/wiki/Geohash) string to calculate latitude and longitude. |  

If both `geohash` and `latitude` and `longitude` are specified, `geohash` is ignored.

## Usage

The world map widget can be used with any input that satisfies the format above, but you
will usually want to bucket locations using geohashing. Humio provides a function {{< function "worldMap" >}}
that helps you conform to the format:

### Example 1: IP Addresses

```humio
worldMap(ip=ip)
```

### Example 2: Existing Geo-Coordinates {#example-2}

```humio
worldMap(lat=myLatitudeField, lon=myLongitudeField, magnitude=avg(responseTime))
```

### Example 3: Alternative

You do not have to use the {{< function "worldMap" >}} function in order to use the widget.
You can also just provide data that conforms to the input format. The results in [Example 2]({{< relref "#example-2" >}})
can be reproduced using the following query:

```humio
geohash(lat=myLatitudeField, lon=myLongitudeField)
| groupBy(_geohash, function=avg(responseTime, as=magnitude))
```

Here we use the {{< function "geoHash" >}} function to acheive the same bucketing
of points as the {{< function "worldMap" >}} function does for us.
If we did not use geohashing we risk getting way too many points, making the
widget very slow.
