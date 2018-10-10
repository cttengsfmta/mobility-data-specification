# Mobility Data Specification: **Agency**

This specification contains a collection of RESTful APIs used to specify the digital relationship between *mobility as a service* providers and the agencies that regulate them. See the [common section](../common/README.md) for definitions shared between specification documents.

* Authors: LADOT
* Date: 10 Aug 2018
* Version: ALPHA

## Table of Contents

* [register_vehicle](#register_vehicle)
* [deregister_vehicle](#deregister_vehicle)
* [update_vehicle_status](#update_vehicle_status)
* [start_trip](#start_trip)
* [end_trip](#start_trip)
* [update_trip_telemetry](#update_trip_telemetry)
* [service_areas](#service_areas)
* [Event types](#Event-Types)
* [Enum definitions](#enum-definitions)

## register_vehicle

The Vehicle Registration API is required in order to register a vehicle for use in the system. The API will require a valid `provider_id`.

Endpoint: `/register_vehicle`\
Method: `POST`

Body:

| Field | Type     | Required/Optional | Other |
| ----- | -------- | ----------------- | ----- |
| `unique_id` | UUID | Required | UUID provided by a provider to uniquely identify a vehicle |
| `provider_id` | String | Required | Issued by agency |
| `vehicle_type` | Enum | Required | Vehicle Type |
| `propulsion_type` | Enum | Required | Propulsion Type |

## deregister_vehicle

The remove-vehicle API is used to deregister a vehicle from the fleet.

Endpoint: `/deregister_vehicle`\
Method: `POST`\
API Key: `Required`

Body:

| Field | Type     | Required/Optional | Other |
| ----- | -------- | ----------------- | ----- |
| `unique_id` | UUID | Required | ID used in [Register](#register_vehicle) |
| `provider_id` | String | Required | Issued by agency |
| `reason_code` | Enum | Required | [Reason](#reason_code) for status change  |

## update_vehicle_status

This API is used by providers when a vehicle is either removed or returned to service.

Endpoint: `/update_vehicle_status`\
Method: `POST`

Body:

| Field | Type | Required/Optional | Other |
| ----- | ---- | ----------------- | ----- |
| `unique_id` | UUID | Required | ID used in [Register](#register_vehicle) |
| `provider_id` | String | Required | Issued by agency |
| `timestamp` | Timestamp | Required | Date/time that event occurred. |
| `location` | Point | Required | Location at the time of status change |
| `event_type` | Enum | Required | [Event Type](#event_type) for status change.  |
| `reason_code` | Enum | Required | [Reason](#reason_code) for status change.  |
| `battery_pct` | Float | Optional | Percent battery charge of device, expressed between 0 and 1 |

## start_trip

Endpoint: `/start_trip`\
Method: `POST`

Body:

| Field | Type | Required/Optional | Other |
| ----- | ---- | ----------------- | ----- |
| `trip_id` | UUID | Required | Uniquely identifies this trip |
| `unique_id` | UUID | Required | ID used in [Register](#register_vehicle) |
| `provider_id` | String | Required | Issued by agency |
| `timestamp` | Timestamp | Required | Date/time that event occurred. |
| `location` | Point | Required | Location at the time of status change |
| `accuracy` | Integer | Required | The approximate level of accuracy, in meters, represented by start_point and end_point. |
| `battery_pct` | Float | Optional | Percent battery charge of device, expressed between 0 and 1 |

## end_trip

Endpoint: `/end_trip`\
Method: `POST`

Body:

| Field | Type | Required/Optional | Other |
| ----- | ---- | ----------------- | ----- |
| `trip_id` | UUID | Required | See [start_trip](#start_trip) |
| `provider_id` | String | Required | Issued by agency |
| `timestamp` | Timestamp | Required | Date/time that event occurred. |
| `location` | Point | Required | Location at the time of status change |
| `accuracy` | Integer | Required | The approximate level of accuracy, in meters, represented by start_point and end_point. |
| `battery_pct` | Float | Optional | Percent battery charge of device, expressed between 0 and 1 |

## update_trip_telemetry

A trip represents a route taken by a provider's customer.   Trip data will be reported to the API every 5 seconds while the vehicle is in motion.

Endpoint: `/update_trip_telemetry`\
Method: `POST`

Body:

| Field | Type     | Required/Optional | Other |
| ----- | -------- | ----------------- | ----- |
| `trip_id` | UUID | Required | See [start_trip](#start_trip) |
| `provider_id` | String | Required | Issued by agency |
| `timestamp` | Timestamp | Required | Date/time that event occurred. |
| `location` | Point | Required | Location at the time of status change |
| `accuracy` | Integer | Required | The approximate level of accuracy, in meters, represented by start_point and end_point. |
| `battery_pct` | Float | Optional | Percent battery charge of device, expressed between 0 and 1 |

## service_areas

Gets the list of service areas available to the provider.

Endpoint: `/service_areas`\
Method: `GET`

Query Parameters:

| Parameter | Type | Required/Optional | Description |
| ----- | ---- | ----------------- | ----- |
| `provider_id` | UUID | Required | A UUID for the Provider, unique within MDS |
| `service_area_id` | UUID  | Optional | If provided, retrieve a specific service area (e.g. a retired or old service area). If omitted, will return all active service areas. |

Response:

| Field | Types  | Required/Optional | Other |
| ----- | ---- | ----------------- | ----- |
| `service_area_id` | UUID | Required |  |
| `service_start_date` | Unix Timestamp | Required | Date at which this service area became effective |
| `service_end_date` | Unix Timestamp | Required | Date at which this service area was replaced. If currently effective, place NaN |
| `service_area` | MultiPolygon | Required | |
| `prior_service_area` | UUID | Optional | If exists, the UUID of the prior service area. |
| `replacement_service_area` | UUID | Optional | If exists, the UUID of the service area that replaced this one |
| `type` | Enum | Required |  See [area types](#area-types) table |

### Area types

| `type` | Description |
|------------| ----------- |
| unrestricted | Areas where devices may be picked up/dropped off. A provider's unrestricted area shall be contained completely inside the agency's unrestricted area for the provider in question, but it need not cover the entire agency unrestricted area. See the provider version of the service areas endpoint |
| restricted | Areas where device pick-up/drop-off is not allowed |
| preferred_pick_up | Areas where users are encouraged to pick up devices |
| preferred_drop_off | Areas where users are encouraged to drop off devices |

### Event Types

| event_type | event_type_description |  reason | reason_description  |
| ---------- | ---------------------- | ------- | ------------------  |
| `available` |    A device becomes available for customer use    | `service_start` |    Device introduced into service at the beginning of the day (if program does not operate 24/7) |
| | | `user_drop_off` |    User ends reservation |
| | | `rebalance_drop_off` | Device moved for rebalancing |
| | | `maintenance_drop_off` |  Device introduced into service after being removed for maintenance |
| `reserved` | A customer reserves a device (even if trip has not started yet) | `user_pick_up` | Customer reserves device |
| `unavailable` |    A device is on the street but becomes unavailable for customer use | `default` |  Default state for a newly registered vehicle    |
| | | `low_battery` | A device is no longer available due to insufficient battery |
| | | ``maintenance`` | A device is no longer available due to equipment issues |
| `removed` | A device is removed from the street and unavailable for customer use | `service_end`| Device removed from street because service has ended for the day (if program does not operate 24/7) |
| | | `rebalance_pick_up` |    Device removed from street and will be placed at another location to rebalance service |
| | | `maintenance_pick_up`     | Device removed from street so it can be worked on |
| `inactive` | A device has been deregistered  |     |  |

## Enum Definitions

### vehicle_type

For `vehicle_type`, options are:

* `bike`
* `scooter`
* `recumbent`

### propulsion_type

For `propulsion_type`, options are:

* `human`
* `electric`
* `combustion`

### reason_code

For `reason_code`, options are:

* `rebalancing`
* `maintenance`
