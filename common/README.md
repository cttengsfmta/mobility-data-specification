# Interchange Protocol
All data shall be transmitted as message bodies for HTTPS requests or replies. The use of encryption is mandatory. Access to every service shall be controlled using HTTP basic authentication. The mechanism for exchanging authentication credentials is not described in this document.

# Geographic Data

References to geographic datatypes (Point, MultiPolygon, etc.) imply coordinates encoded in the [WGS 84 (EPSG:4326)](https://en.wikipedia.org/wiki/World_Geodetic_System) standard GPS projection expressed as [Decimal Degrees](https://en.wikipedia.org/wiki/Decimal_degrees).

Whenever an individual location coordinate measurement is presented, it must be
represented as a GeoJSON [`Feature`](https://tools.ietf.org/html/rfc7946#section-3.2) object with a corresponding [`timestamp`][ts] property and [`Point`](https://tools.ietf.org/html/rfc7946#section-3.1.2) geometry:

```json
{
    "type": "Feature",
    "properties": {
        "timestamp": 1538770678558
    },
    "geometry": {
        "type": "Point",
        "coordinates": [
            -118.46710503101347,
            33.9909333514159
        ]
    }
}
```
## Route

A route is represented as a GeoJSON [`FeatureCollection`](https://tools.ietf.org/html/rfc7946#section-3.3), which includes every observed `Point` in the route.

The route must include at least 2 `Point`s, a start point and end point. Additionally, it must include all possible GPS samples collected by a provider.

```json
"route": {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "properties": {
                    "timestamp": 1538770678558
                },
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        -118.46710503101347,
                        33.9909333514159
                    ]
                }
            },
            {
                "type": "Feature",
                "properties": {
                    "timestamp": 1538770678558
                },
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        -118.464851975441,
                        33.990366257735
                    ]
                }
            }
        ] }
```

# Timestamps

The `timestamp` type is an integer containing the offset in milliseconds from the start of the epoch in [Coordinated Universal Time (UTC)](https://www.nhc.noaa.gov/aboututc.shtml).

[ts]: #timestamps

# Response Format

Responses must be `UTF-8` encoded `application/json` and must minimally include the MDS `version` and a `data` payload:

```json
{
    "version": "x.y.z",
    "data": {
        "trips": [{
            "provider_id": "...",
            "trip_id": "...",
        }]
    }
}
```
All response fields must use `lower_case_with_underscores`.

## Error Body
In cases where an error is returned, the response code shall be 400 and the response body shall have the following fields

Field Name        | Type      | Required  | Defines
------------------| --------- | --------- | --------
errors            | Array     | Yes       | A list of strings. Must contain at least one error message
\-                | String    | Yes       | Informative error message

Example:
```json
{
  "version": "x.y.z",
  "data": {
      "errors": [
      "box_se must be specified if box_nw is specified!",
      "end_time happens before start_time!"
    ]
  }
}
```
