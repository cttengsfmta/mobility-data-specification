# Interchange Protocol
All data shall be transmitted as message bodies for HTTPS requests or replies. The use of encryption is mandatory. Access to every service shall be controlled using HTTP basic authentication. The mechanism for exchanging authentication credentials is not described in this document.

# Geographic Data

References to geographic datatypes (Point, MultiPolygon, etc.) imply coordinates encoded in the [WGS 84 (EPSG:4326)](https://en.wikipedia.org/wiki/World_Geodetic_System) standard GPS projection expressed as [Decimal Degrees](https://en.wikipedia.org/wiki/Decimal_degrees).

Any references to a a `point` type mean a [GeoJSON Point geometry object](https://tools.ietf.org/html/rfc7946#section-3.1.2)

# Timestamps

The `timestamp` type is an integer containing the offset in milliseconds from the start of the epoch in [Coordinated Universal Time (UTC)](https://www.nhc.noaa.gov/aboututc.shtml).

[ts]: #timestamps

# UUIDs

Any reference to a "UUID" in these documents means any version of an [RFC 4122 UUID](https://tools.ietf.org/html/rfc4122)

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
