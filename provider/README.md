# Mobility Data Specification: **Provider**

This specification contains a data standard for *mobility as a service* providers to define a RESTful API for municipalities to access on-demand. See the [common section](../common/README.md) for definitions shared between specification documents.

## Table of Contents

* [General Information](#general-information)
* [Trips](#trips)
* [Status Changes](#status-changes)
* [Realtime Data](#realtime-data)

## General Information

The following information applies to all `provider` API endpoints.

### Data Retention Requirement

All the endpoints which provide historical data must do so for at least the previous two years. What shall be returned for specific endpoints in the case where the data does not exist back that far is specified for each endpoint where it is applicable.

### JSON Schema

MDS defines [JSON Schema](https://json-schema.org/) files for [`trips`][trips-schema] and [`status_changes`][sc-schema].

`provider` API responses must validate against their respective schema files. The schema files always take precedence over the language and examples in this and other supporting documentation meant for *human* consumption.

### UUIDs for Devices

MDS defines the *device* as the unit that transmits GPS signals for a particular vehicle. A given device must have a UUID (`device_id` below) that is unique within the Provider's fleet.

Additionally, `device_id` must remain constant for the device's lifetime of service, regardless of the vehicle components that house the device.

## Trips

A trip represents a journey taken by a *mobility as a service* customer with a geo-tagged start and stop point.

The trips API allows a user to query historical trip data.

Endpoint: `/trips`\
Method: `GET`

Response: See the [`trips` schema][trips-schema] for the expected format.

Data: `{ "trips": [] }`, an array of objects with the following structure

| Field | Type    | Required/Optional | Comments |
| ----- | -------- | ----------------- | ----- |
| `provider_id` | UUID | Required | A UUID for the Provider, unique within MDS |
| `provider_name` | String | Required | The public-facing name of the Provider |
| `device_id` | UUID | Required | A unique device ID in UUID format |
| `vehicle_id` | String | Required | The Vehicle Identification Number visible on the vehicle itself |
| `vehicle_type` | Enum | Required | See [vehicle types](#vehicle-types) table |
| `propulsion_type` | Enum[] | Required | Array of [propulsion types](#propulsion-types); allows multiple values |
| `trip_id` | UUID | Required | A unique ID for each trip |
| `trip_duration` | Integer | Required | Time, in Seconds |
| `trip_distance` | Integer | Required | Trip Distance, in Meters |
| `route` | GeoJSON `FeatureCollection` | Required | See [Routes](#routes) detail below |
| `accuracy` | Integer | Required | The approximate level of accuracy, in meters, of `Points` within `route` |
| `start_time` | [timestamp][ts] | Required | |
| `end_time` | [timestamp][ts] | Required | |
| `parking_verification_url` | String | Optional | A URL to a photo (or other evidence) of proper vehicle parking |
| `standard_cost` | Integer | Optional | The cost, in cents, that it would cost to perform that trip in the standard operation of the System |
| `actual_cost` | Integer | Optional | The actual cost, in cents, paid by the customer of the *mobility as a service* provider |
| `membership_type` | Enum | Optional | See [membership types](#membership-types) table |

### Trips Query Parameters

The trips API should allow querying trips with a combination of query parameters.

* `device_id`
* `vehicle_id`
* `start_time`: filters for trips where `start_time` occurs at or after the given time
* `end_time`: filters for trips where `end_time` occurs at or before the given time
* `bbox`

All of these query params will use the *Type* listed above with the exception of `bbox`, which is the bounding-box.

For example:

```
bbox=-122.4183,37.7758,-122.4120,37.7858
```

Gets all trips within that bounding-box where any point inside the `route` is inside said box. The order is defined as: southwest longitude, southwest latitude, northeast longitude, northeast latitude (separated by commas).

Trips for the entire service area shall be returned if a bounding box is not specified. Trips for all devices shall be returned if a device Id is not specified. The provider shall return the empty JSON object for requests with an end_time that happened before the earliest available trip. A provider shall return an error response as described in the [common section](../common/README.md) for requests with a start_time in the future or an end_time which is smaller than the start_time.

### Vehicle Types

| `vehicle_type` |
|--------------|
| bicycle      |
| scooter      |

### Propulsion Types

| `propulsion_type` |
|-----------------|
| human           |
| electric_assist |
| electric        |
| combustion      |

### Membership Types

| `membership_type` |
|--------------|
| subscriber             |
| subscriber_low_income  |
| single_ride            |
| single_ride_low_income |

## Status Changes

The status of the inventory of vehicles available for customer use.

This API allows a user to query the historical availability for a system within a time range.

Endpoint: `/status_changes`\
Method: `GET`

Response: See the [`status_changes` schema][sc-schema] for the expected format.

Data: `{ "status_changes": [] }`, an array of objects with the following structure

| Field | Type | Required/Optional | Comments |
| ----- | ---- | ----------------- | ----- |
| `provider_id` | UUID | Required | A UUID for the Provider, unique within MDS |
| `provider_name` | String | Required | The public-facing name of the Provider |
| `device_id` | UUID | Required | A unique device ID in UUID format |
| `vehicle_id` | String | Required | The Vehicle Identification Number visible on the vehicle itself |
| `vehicle_type` | Enum | Required | see [vehicle types](#vehicle-types) table |
| `propulsion_type` | Enum[] | Required | Array of [propulsion types](#propulsion-types); allows multiple values |
| `event_type` | Enum | Required | See [event types](#event-types) table |
| `event_type_reason` | Enum | Required | Reason for status change, allowable values determined by [`event type`](#event-types) |
| `event_time` | [timestamp][ts] | Required | Date/time that event occurred, based on device clock |
| `event_location` | GeoJSON [Point Feature][geo] | Required | |
| `battery_pct` | Float | Required if Applicable | Percent battery charge of device, expressed between 0 and 1 |
| `associated_trips` | UUID[] | Optional based `event_type_reason` | Array of UUID's. For `user`-generated event types, associated trips (foreign key to Trips API) |

### Status Changes Query Parameters

The status_changes API should allow querying status changes with a combination of query parameters.

* `start_time`: filters for status changes where `event_time` occurs at or after the given time
* `end_time`: filters for status changes where `event_time` occurs at or before the given time
* `bbox`: filters for status changes where `event_location` is within defined bounding-box. The order is definied as: southwest longitude, southwest latitude, northeast longitude, northeast latitude (separated by commas). 

Example:

```
bbox=-122.4183,37.7758,-122.4120,37.7858
```

Status changes for the entire service area shall be returned if a bounding box is not specified. Status changes for all devices shall be returned if a device Id is not specified. The provider shall return the empty JSON object for requests with an end_time that happened before the earliest available trip. A provider shall return an error response as described in the [common section](../common/README.md) for requests with a start_time in the future or an end_time which is smaller than the start_time.

### Event Types

| `event_type` | Description | `event_type_reason` | Description |
| ---------- | ---------------------- | ------- | ------------------ |
| `available` | A device becomes available for customer use | `service_start` | Device introduced into service at the beginning of the day (if program does not operate 24/7) |
| | | `user_drop_off` | User ends reservation |
| | | `rebalance_drop_off` | Device moved for rebalancing |
| | | `maintenance_drop_off` | Device introduced into service after being removed for maintenance |
| `reserved` | A customer reserves a device (even if trip has not started yet) | `user_pick_up` | Customer reserves device |
| `unavailable` | A device is on the street but becomes unavailable for customer use | `maintenance` | A device is no longer available due to equipment issues |
| | | `low_battery` | A device is no longer available due to insufficient battery |
| `removed` | A device is removed from the street and unavailable for customer use | `service_end` | Device removed from street because service has ended for the day (if program does not operate 24/7) |
| | | `rebalance_pick_up` | Device removed from street and will be placed at another location to rebalance service |
| | | `maintenance_pick_up` | Device removed from street so it can be worked on |

<<<<<<< HEAD
## Realtime Data

All MDS compatible `provider` APIs must expose a public [GBFS](https://github.com/NABSA/gbfs) feed as well. Given that GBFS hasn't fully [evolved to support dockless mobility](https://github.com/NABSA/gbfs/pull/92) yet, we follow the current guidelines in making bike information avaliable to the public. 

  - `system_information.json` is always required
  - `free_bike_status.json` is required for MDS
  - `station_information.json` and `station_status.json` don't apply for MDS

=======
## Service Areas

Gets the list of service areas available to the provider.

Endpoint: `/service_areas`
Method: `GET`
Query Parameters:

| Parameter | Type | Required/Optional | Description |
| ----- | ---- | ----------------- | ----- |
| `service_area_id` | UUID  | Optional | If provided, retrieve a specific service area (e.g. a retired or old service area). If omitted, will return all active service areas. |

Response:

| Field | Types  | Required/Optional | Other |
| ----- | ---- | ----------------- | ----- |
| `service_area_id` | UUID | Required |  |
| `service_start_date` | timestamp | Required | Date at which this service area became effective |
| `service_end_date` | timestamp | Required | Date at which this service area was replaced. If currently effective, place NaN |
| `service_area` | MultiPolygon | Required | |
| `prior_service_area` | UUID | Optional | If exists, the UUID of the prior service area. |
| `replacement_service_area` | UUID | Optional | If exists, the UUID of the service area that replaced this one |
| `type` | Enum | Required |  See [area types](#area-types) table |

### Area types

| `type` | Description |
|------------| ----------- |
| unrestricted | Areas where devices may be picked up/dropped off. A provider's unrestricted area shall be contained completely inside the agency's unrestricted area for the provider in question, but it need not cover the entire agency unrestricted area. See the agency version of the service areas endpoint |
| agency_restricted | Areas where the agency does not allow device pick-up/drop-off |
| agency_preferred_pick_up | Areas where users are encouraged by the agency to pick up devices |
| agency_preferred_drop_off | Areas where users are encouraged by the agency to drop off devices |
| provider_restricted | Areas where the provider does not allow device pick-up/drop-off |
| provider_preferred_pick_up | Areas where users are encouraged by the provider to pick up devices |
| provider_preferred_drop_off | Areas where users are encouraged by the provider to drop off devices |

[Top][toc]

### Realtime Data
>>>>>>> Add service area type to the agency API. Add service areas to the provider API


[Top][toc]

[sc-schema]: status_changes.json
[toc]: #table-of-contents
[trips-schema]: trips.json
