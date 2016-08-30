# Our API Specification

## Overview

Central to the WhereIsMyTransport platform is our transport API. It is based on [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer), [JSON](http://www.json.org/), [OAuth 2.0](http://oauth.net/2/) and [OpenID Connect](http://openid.net/connect/). These are standards which are broadly supported in the industry. 

### Introduction

#### API Endpoint

The following address is the standard URL endpoint to be used to access the various resources of the API.

`https://platform.whereismytransport.com`

For example, the [agencies endpoint](#agencies) would be queried at https://platform.whereismytransport.com/api/agencies.

#### HTTP verbs

The API uses standard HTTP verbs as part of the uniform interface and must be used in combination with the resource URIs in order to describe the requested action.

| Verb | Description |
| :-------- | :---------- |
| `GET` | To retrieve a resource or collection of resources. |
| `POST` | To create a resource. |

#### HTTP status codes 

The status code in an HTTP request's response describes the outcome of the performed action. Listed below are the supported status codes used in the API.

| Status code | Description |
| :------------ | :---------- |
| `200 Ok` | The GET request was successful. The retrieved resource is returned in the response body. |
| `201 Created` | The POST request was successful. The new resource is returned in the response body. |
| `400 Bad Request` | The request has invalid or missing parameters. |
| `401 Unauthorized` | Authentication failed. |
| `403 Forbidden` | Access denied to this resource. |
| `404 Not Found` | The requested resource does not exist. |
| `406 Not Acceptable` | The server cannot send data in a format requested in the Accept header. |
| `415 Unsupported Media Type` | The server does not support the format as specified in the Content-Type header. |
| `429 Too Many Requests` | Too many requests have occurred in a given amount of time. |
| `500 Internal Server Error` | The server has encountered an unexpected error. |

#### Accept type

The **Accept** header describes the format of the content that the client can accept. All requests must specify this header as **application/json**. If the **Accept** header is not specified or the type is set to a value other than **application/json** then a **406 Not Acceptable** [status code](#http-status-codes) will be returned.

### Sample request

```
GET api/agencies
Accept: application/json
```

#### Content type

The **Content-Type** header describes the format of the data being posted to the server. All POST requests must specify this header as **application/json**. If the **Content-Type** header is not specified or the type is set to a value other than **application/json** then a **415 Unsupported Media Type** [status code](#status-codes) will be returned.

### Sample request

```
POST api/journeys
Accept: application/json
Content-Type: application/json
```

#### Compression

The API compresses response data using GZIP compression as defined by the HTTP 1.1 specification. Disabling compression can be done by setting the **Allow-Compression** request header to **false**. The response will always have the **Content-Encoding** response header set to **gzip** when the response body is compressed accordingly.

**Note:** No other methods of compression are supported. Request compression is not supported.

#### HTTPS

All API access is performed over HTTPS only. If a resource is requested using **http://** then a **403 Forbidden** [status code](#status-codes) will be returned.

### Authorisation

The API uses the standards as set in the [OAuth 2.0](http://oauth.net/2/) and [OpenID Connect](http://openid.net/connect/) specifications. WhereIsMyTransport provides its own _security token service_ which issues tokens to applications so that they can authenticate themselves against the API. To authorise an application, one must first acquire a **client_id** and **client_secret** from the [Developer Portal](https://developer.whereismytransport.com).

Using client credentials one can make requests against the _security token service_ to retrieve a token.  When requesting a token, the scopes that are required must also be specified. Currently, the only scope available is `transportapi:all` which provides full access to the API.

**Note:** The content type of **application/x-www-form-urlencoded** must be used for this request.

#### Security token endpoint

The following is the full URI endpoint used to retrieve a token.

`https://identity.whereismytransport.com/connect/token`

##### Sample request

```
POST https://identity.whereismytransport.com/connect/token
Content-Type: application/x-www-form-urlencoded
client_id=6a73386c-5bbf-4b4b-ae09-0ed45d87b5ad
client_secret=neph3dext3Ct+QHc5wClxTWE1Y+AKNTS28UFH39SXy4=
grant_type=client_credentials
scope=transportapi:all
```

##### Sample response

```
201 Created
{
    "access_token": "eyJ0eXAiOiJ32aQiLCJhbGciOiJSUzI1NiIsIfg1iCI6ImEzck1VZ01Gd8d0UGNsTGE2eUYz...",
    "expires_in": 3600,
    "token_type": "Bearer"
}
```

Once this token is received, it must be added to the **Authorization** HTTP request header with the prefix of "Bearer" in order to perform successful requests against the API.

##### Sample request

```
GET api/agencies
Authorization: Bearer eyJ0eXAiOiJ32aQiLCJhbGciOiJSUzI1NiIsIfg1iCI6ImEzck1VZ01Gd8d0UGNsTGE2eUYz...
```

A 401 Unauthorised status code will be returned if the request is either missing the Authorization header or if the token has expired.

#### Token expiry

XYZ

### Errors

The API uses conventional HTTP [status code](#http-status-codes) to indicate the result of a request. Codes within the 200s indicate that the request was successful. Codes within the 400s indicate that the request was somehow badly formed (such as a missing or incorrectly formatted field). 500s are typically returned when something unexpected goes wrong on the server. The **error response model** below will be returned for any error.

#### Error response model 

| Field | Type | Description |
| :--------- | :--- | :---- |
| message | string | A general message indicating the type of error. |
| fields | Dictionary of error messages | Human-readable error messages for each problematic input or query field. Only returned for a 400 (Bad Request). |

##### Sample response

```
400 Bad Request
{
    "message": "The request has invalid or missing fields. Please visit our documentation.",
    "fields": {
        "timeType": [
            "Value 'After' is invalid. TimeType must be one of 'DepartAfter', 'ArriveBefore'."
        ],
        "maxItineraries": [
            "Value '10' is invalid. The maximum number of itineraries that may be requested is 5."
        ]
    }
}
```

### Identifiers

Almost all entities in the API are identified through the use of a globally unique identifier. This identifier is specified as a 22 character long, case-sensitive string of URL-friendly characters; __a__ to __z__, __A__ to __Z__, __0__ to __9__, - (hyphen) and _ (underscore). See the **id** field in the sample response below.

##### Sample request

```
GET api/agencies/5kcfZkKW0ku4Uk-A6j8MFA
```

##### Sample response

```
{
    "id": "5kcfZkKW0ku4Uk-A6j8MFA",
    "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
    "name": "MyCiTi",
    "culture": "en"
}
```

**Note:** Identifiers are case-sensitive.

### Resource Linking

The API presents a simple and elegant approach to referencing related resources. Every available resource has a hypertext reference field, **href**. This value represents a fully qualified URL to where the resource resides. Presenting such a field for every resource aids discoverability by allowing for new resources to be consumed by just embedding a new reference link.

For example, the sample below demonstrates a stop with its agency represented as a sub-resource, both having **id** and **href** fields.

##### Sample request

```
GET api/stops/eBTeYLPXOkWm5zyfjZVaZg
```

##### Sample response

```
200 Ok
{
    "id": "eBTeYLPXOkWm5zyfjZVaZg",
    "href": "https://platform.whereismytransport.com/api/stops/eBTeYLPXOkWm5zyfjZVaZg",
    "agency": {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
        "name": "MyCiTi",
        "culture": "en"
    },
    "name": "Adderley",
    "geometry": {
        "type": "Point",
        "coordinates": [
            18.424849,
            -33.920555
        ]
    },
    "modes": [
        "Bus"
    ]
}
```

### Excluding data 

In order to reduce payload, it is possible to exclude certain objects or collections from the model returned in the body of the HTTP response. This is done through the use of the **exclude** query. Fields which are _excludable_ are described in the specification with the [Excludable](#excludable) tag.

| Parameter | Type | Required | Description |
| :-------------- | :--- | :---- | :---- |
| exclude | string | Optional | A string of comma-separated object or collection names to excluded from the response. |

When excluding resource objects, the containing object with their **id** and **href** fields will remain. This is to preserve discoverability while still reducing payload. Collections, on the other hand, will be excluded entirely.

##### Sample request

The request below will exclude **geometry** and **directions** from the resource model.

```
POST api/journeys/8GYKddjcAk6j7aVUAMV3pw?exclude=geometry,directions
```

### Understanding Scheduled Data

All retrievable entities from the API constitute scheduled data. This means that entities may change over time. They may not even exist forever. An agency, for example, may schedule a line's name to change, not now, but only after a certain date.  A new stop could be scheduled to only be returned from the API at some given date. 

The important thing to note that is an entity could be deprecated in a future schedule. This means that any entity resource URI could return a **404 Not Found** [status code](#http-status-codes). Applications built on this API are highly encouraged to cater for this.

### Pagination 

Depending on the structure of a query, a lot of results could be returned from the API. For that reason, the results are paginated so to ensure that responses are easier to handle and that payload it kept to a manageable size.

| Parameter | Type | Required | Description |
| :-------------- | :--- | :---- | :---- |
| limit | int | Optional | The number of entities to be returned. The default and maximum is typically 100. |
| offset | int | Optional | The zero-based offset of the first entity returned. The default is 0.  |

##### Sample request

The request below will retrieve 10 stops from the 50th stop onwards.

```
GET api/stops?limit=10&offset=50
```

### Formatting Standards 

#### DateTime 

A typical format for encoding of date and time in JSON is to use the ISO 8601 standard. This is a well-established specification which is both human readable and widely supported by many web-based frameworks.

ISO 8601 date and time strings can be represented as "2016-11-19T07:22Z" (7:22 AM on the 19th November 2016).

More information can be found [here](https://en.wikipedia.org/wiki/ISO_8601).

**Note:** ISO 8601 dates are timzone-agnostic and so are communicated in UTC (Coordinated Universal Time).

#### Culture 

The appropriate culture format used in the API is based on RFC 4646. This unique name is a combination of an ISO 639 two-letter lowercase culture code associated with a language and an ISO 3166 two-letter uppercase subculture code associated with a country or region.

For example, _en-US_ refers to English (United States) and _en-ZA_ to English (South Africa).

The detailed specification can be found [here](https://www.ietf.org/rfc/rfc4646.txt).

#### Cost 

Monetary amounts are represented by the cost object, which is made up of an amount, as a decimal value, and the applicable currency code. The currency code is such as defined in ISO 4217. For example, **ZAR** represents the South African Rand. More information and a full list of currency codes can be found [here](https://en.wikipedia.org/wiki/ISO_4217).

##### Sample

The following cost object represents the value of R10,50.

```
{
    "cost": {
        "amount": 10.5,
        "currencyCode": "ZAR"
    }
}
```

#### GeoJSON 

[GeoJSON](http://geojson.org) is a JSON format for encoding geographic data structures. The API uses the _Point_, _MultiPoint_ and _LineString_ geometry types.

A typical GeoJSON structure consists of a **type** field and an array of **coordinates**. 

**Note:**  GeoJSON represents geographic coordinates with longitude first and then latitude, `[longitude, latitude]`. i.e. `[x, y]` in the Cartesian coordinate system.

##### Sample

The following GeoJSON Point represents the coordinates for Cape Town's city centre.

```
{
    "geometry": {
        "type": "Point",
        "coordinates": [
            18.417035,
            -33.926852
        ]
    }
}
```

#### Point 

In order to provide a geographic position through the query string, a comma-separated latitude and longitude must be provided.

##### Sample request

```
GET api/stops?point=-33.925430,18.436443&radius=1750
```

**Note: ** The order is latitude then longitude.

#### Bounding Box 

In order to provide a geographic bounding box through the query string, a comma-separated SW (south west) latitude, SW longitude, NE (north east) latitude and NE longitude must be provided in that order.  These coordiantes represent the south west and north east corners of the box.

```
GET api/stops?bbox=-33.944,18.36,-33.895,18.43
```

#### Distance 

Distance is returned as an object consisting of the distance value (an integer) and the associated unit symbol.

##### Sample response

```
{  
    "distance": {  
        "value":133,
        "unit":"m"
    }
}
```

**Note:** The API currently only supports the metric system. Distance is returned in metres.

## Specification

### Modes

The mode of transport describes the type of vehicle that is used along a line. The following table describes the modes currently supported by the API.

| Value | Description |
| :--------- | :--- | :---- |
| LightRail | A service for any light rail or street level system within a metropolitan area. |
| Subway | An underground rail service. |
| Rail | An overground railway transportation service. |
| Bus | A bus service utilising a road network. |
| Ferry | A boat service. |
| GroundCableCar | A service for street level cable cars where the cable runs beneath the car. |
| Gondola | An aerial cable car service. |
| Funicular | A rail system service designed for steep inclines. |
| Coach | A long distance bus service. |
| Air | A service for any air travel. |

### Agencies

An agency, or operator, is an organisation which provides and governs a transport service which is available for use by either the general public (in most cases) or through some private arrangement.

#### Agency response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | [identifier](#identifiers) | The identifier of the agency. |
| href | string | The hyperlink pointing to the agency. |
| name | string | The full name of the agency. |
| culture | string | The name of the [culture](#culture), based on RFC 4646. |

#### Retrieving agencies

Retrieves a collection of agencies.

`GET api/agencies?point={point}&radius={int}&bbox={BoundingBox}&agencies={agencies}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| point | [Point](#point) | The point from where to search for nearby agencies. Agencies will be returned in order of their distance from this point (from closest to furthest). |
| radius | integer | The distance in metres from the point to search for nearby agencies. This filter is optional. |
| bbox | [Bounding Box](#bounding-box) | The bounding box from where to retrieve agencies. This will be ignored if a point is provided in the query.  |
| agencies | Array of [Identifier](#identifiers) | Comma-separated list of agency identifiers through which to filter the results by. |
| limit | int | See [Pagination](#pagination). |
| offset | int | See [Pagination](#pagination). |

##### Sample request

```
GET api/agencies?bbox=-33.94,18.36,-33.89,18.43
```

##### Sample response

```
200 Ok
[
    {
        "id": "xp_eNbqkYEaZP2YZkHwQqg",
        "href": "https://platform.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
        "name": "Metrorail Western Cape",
        "culture": "en"
    },
    {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
        "name": "MyCiTi",
        "culture": "en"
    },
    {
        "id": "PO83DTm4oEuJ19prwicxHw",
        "href": "https://platform.whereismytransport.com/api/agencies/PO83DTm4oEuJ19prwicxHw",
        "name": "Nelson Mandela Gateway",
        "culture": "en"
    }
]
```

#### Retrieving a specific agency

Retrieves an agency by its identifier.

`GET api/agencies/{agencyId}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| agencyId | [Identifier](#identifiers) | The identifier of the agency. |

##### Sample request

```
GET api/agencies/5kcfZkKW0ku4Uk-A6j8MFA
```

##### Sample response

```
200 Ok
{
    "id": "5kcfZkKW0ku4Uk-A6j8MFA",
    "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
    "name": "MyCiTi",
    "culture": "en"
}
```

### Stops

A location where passengers can board or alight from a transport vehicle. 

#### Stop response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the stop. |
| href | string | The hyperlink pointing to the stop. |
| agency | [Agency](#agency-model) | **[**[Excludable](#excludable)**]** The agency. |
| name | string | The full name of the stop. |
| code | string | If available, the passenger code of the stop. |
| geometry | [GeoJSON](#geojson) Point | The geographic point of the stop. |
| modes | Array of [Mode](#modes) | The modes that are served by this stop. |
| parentStop | [Stop](#stop-model) | **[**[Excludable](#excludable)**]** If applicable, the parent stop. |

#### Retrieving stops

Retrieves a collection of stops.

`GET api/stops?point={point}&radius={int}&bbox={bbox}&modes={modes}&agencies={agencies}&servesLines={identifier}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| point | [Point](#point) | The point from where to search for nearby stops. Stops will be returned in order of their distance from this point (from closest to furthest). |
| radius | integer | The distance in metres from the point to search for nearby stops. This filter is optional. |
| bbox | [Bounding Box](#bounding-box) | The bounding box from where to retrieve stops. This will be ignored if a point is provided in the query.  |
| modes | string | A string of comma-separated [transport modes](#modes). |
| agencies | Array of [Identifier](#identifiers) | A string of comma-separated agency identifiers to filter the results by. |
| servesLines | Array of [Identifier](#identifiers) | A string of comma-separated line identifiers to filter the results by. |
| showChildren | bool | Specifies whether or not to display children stops that satisfy the query parameters. Default is false. |
| limit | int | See [Pagination](#pagination). |
| offset | int | See [Pagination](#pagination). |

##### Sample request

```
GET api/stops?agencies=5kcfZkKW0ku4Uk-A6j8MFA&servicesLines=kYdaW1_dKUe7WZegmV1bFw&point=-33.923400,18.421586&radius=200
```

##### Sample response

```
200 Ok
Content-Type: application/json
[
    {
        "id": "k42lx2toK0yJiVGvsM4WSA",
        "href": "https://platform.whereismytransport.com/api/stops/k42lx2toK0yJiVGvsM4WSA",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
            "name": "MyCiTi",
            "culture": "en"
        },
        "name": "Groote Kerk",
        "geometry": {
            "type": "Point",
            "coordinates": [
                18.421188,
                -33.923939
            ]
        },
        "modes": [
            "Bus"
        ]
    },
    {
        "id": "-rMw7Fg3GEqwJIsiDa4xMA",
        "href": "https://platform.whereismytransport.com/api/stops/-rMw7Fg3GEqwJIsiDa4xMA",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
            "name": "MyCiTi",
            "culture": "en"
        },
        "name": "Darling",
        "geometry": {
            "type": "Point",
            "coordinates": [
                18.422529,
                -33.92407
            ]
        },
        "modes": [
            "Bus"
        ]
    }
]
```

#### Retrieving a specific stop

Retrieves a stop by its identifier.

`GET api/stops/{stopId}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| stopId | [Identifier](#identifiers) | The identifier of the stop. |

##### Sample request

```
GET api/stops/eBTeYLPXOkWm5zyfjZVaZg
```

##### Sample response

```
200 Ok
{
    "id": "eBTeYLPXOkWm5zyfjZVaZg",
    "href": "https://platform.whereismytransport.com/api/stops/eBTeYLPXOkWm5zyfjZVaZg",
    "agency": {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
        "name": "MyCiTi",
        "culture": "en"
    },
    "name": "Adderley",
    "geometry": {
        "type": "Point",
        "coordinates": [
            18.424849,
            -33.920555
        ]
    },
    "modes": [
        "Bus"
    ]
}
```

#### Retrieving child stops for some parent stop

Retrieves all children of the parent stop specified by its identifier.

`GET api/stops/{stopId}/stops`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| stopId | [Identifier](#identifiers) | The identifier of the parent stop. |

##### Sample request

```
GET api/stops/E8qYuZ4nEUSLS13pskx1Qg/stops
```

##### Sample response

```
200 Ok
[
    {
        "id": "qiCODz6ky0qZ9agqLgF44Q",
        "href": "https://platform.whereismytransport.com/api/stops/qiCODz6ky0qZ9agqLgF44Q",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
        },
        "name": "Acacia",
        "code": "FAT073NB",
        "geometry": {
            "type": "Point",
            "coordinates": [
                18.503569,
                -33.578148
            ]
        },
        "modes": [
            "Bus"
        ],
        "parentStop": {
            "id": "E8qYuZ4nEUSLS13pskx1Qg",
            "href": "https://platform.whereismytransport.com/api/stops/E8qYuZ4nEUSLS13pskx1Qg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
            },
            "name": "Acacia",
            "geometry": {
                "type": "Point",
                "coordinates": [
                    18.503632,
                    -33.578362
                ]
            },
            "modes": [
                "Bus",
                "Bus"
            ]
        }
    },
    {
        "id": "N_hUhxwX-EaNn5SRMk2VJg",
        "href": "https://platform.whereismytransport.com/api/stops/N_hUhxwX-EaNn5SRMk2VJg",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
        },
        "name": "Acacia",
        "code": "FAT073SB",
        "geometry": {
            "type": "Point",
            "coordinates": [
                18.503695,
                -33.578577
            ]
        },
        "modes": [
            "Bus"
        ],
        "parentStop": {
            "id": "E8qYuZ4nEUSLS13pskx1Qg",
            "href": "https://platform.whereismytransport.com/api/stops/E8qYuZ4nEUSLS13pskx1Qg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
            },
            "name": "Acacia",
            "geometry": {
                "type": "Point",
                "coordinates": [
                    18.503632,
                    -33.578362
                ]
            },
            "modes": [
                "Bus",
                "Bus"
            ]
        }
    }
]
```

### Stop Timetables

A timetable of vehicles arriving and departing from a stop along their respective routes.

#### Stop Timetable response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| arrivalTime | [DateTime](#datetime) | The arrival time of the vehicle at this stop along its route. |
| departureTime | [DateTime](#datetime) | The departure time of the vehicle from this stop along its route. |
| vehicle | [Vehicle](#vehicle-model) | If available, identifying information for the vehicle running at this time. |
| line | [Line](#line-model) | The line from which the vehicle is traveling. |

#### Retrieving a stop timetable

Retrieves a timetable for a stop, consisting of a list of occurrences of a vehicle calling at this stop in order of arrival time.

`GET api/stops/{stopId}/timetables?earliestArrivalTime={DateTime}&limit={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| stopId | [Identifier](#identifiers) | The identifier of the stop. |
| earliestArrivalTime | [DateTime](#datetime) | The earliest arrival date and time to include in the timetable. |
| limit | int | The maximum number of entities to be returned. Default is 10. |

##### Sample request

```
GET api/stops/eBTeYLPXOkWm5zyfjZVaZg/timetables?limit=2
```

##### Sample response

```
200 Ok
[
    {
        "arrivalTime": "2016-08-29T14:54:00Z",
        "departureTime": "2016-08-29T14:54:00Z",
        "vehicle": {},
        "line": {
            "id": "vBk_jw2saU-gfZCgo_JvLg",
            "href": "https://platform.whereismytransport.com/api/lines/vBk_jw2saU-gfZCgo_JvLg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
                "name": "MyCiTi",
                "culture": "en"
            },
            "name": "Route 103 to Civic Centre",
            "mode": "Bus",
            "colour": "#ffb7dee8",
            "textColour": "#ffffffff"
        }
    },
    {
        "arrivalTime": "2016-08-29T15:03:00Z",
        "departureTime": "2016-08-29T15:03:00Z",
        "vehicle": {},
        "line": {
            "id": "vBk_jw2saU-gfZCgo_JvLg",
            "href": "https://platform.whereismytransport.com/api/lines/vBk_jw2saU-gfZCgo_JvLg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
                "name": "MyCiTi",
                "culture": "en"
            },
            "name": "Route 103 to Civic Centre",
            "mode": "Bus",
            "colour": "#ffb7dee8",
            "textColour": "#ffffffff"
        }
    }
]
```

### Lines

A grouping together of routes marketed to passengers as a single section of the transport network.

#### Line response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the line. |
| href | string | The hyperlink pointing to the line. |
| agency | [Agency](#agency-model) | **[**[Excludable](#excludable)**]** The line's agency. |
| name | string | If available, the full name of the line, . Either name or shortName will exist. |
| shortName | string | If available, the short name of the line. Either name or shortName will exist. |
| description | string | If available, a description of the line. |
| mode | [Mode](#modes) | The transport mode of the line. |
| colour | string | The assigned colour of the line as an 8-character (ARGB) hexadecimal number. For example, #FFFF0000 (red with 100% opacity). |
| textColour | string | The colour of the text when drawn against the colour of the line so to provide sufficient contrast for legibility. Also an 8-character (ARGB) hexadecimal number. For example, #CCFFFFFF (white with 50% opacity). |

#### Retrieving lines

Retrieves a collection of lines.

`GET api/lines?agencies={agencies}&servesStops={stops}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| agencies | Array of [Identifier](#identifiers) | A comma-separated list of agency identifiers to filter the results by. |
| servesStops | Array of [Identifier](#identifiers) | A comma-separated list of stop identifiers that represent stops which the returned lines must serve or visit. |
| limit | int | See [Pagination](#pagination). |
| offset | int | See [Pagination](#pagination). |

##### Sample request

```
GET api/lines?agencies=5kcfZkKW0ku4Uk-A6j8MFA&limit=2
```

##### Sample response

```
200 Ok
[
    {
        "id": "-29sdEyVLkel5ouTHN5Bsg",
        "href": "https://platform.whereismytransport.com/api/lines/-29sdEyVLkel5ouTHN5Bsg",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
            "name": "MyCiTi",
            "culture": "en"
        },
        "name": "A01 - Airport to Airport",
        "mode": "Bus",
        "colour": "#ffa7a9ac",
        "textColour": "#ffffffff"
    },
    {
        "id": "kYdaW1_dKUe7WZegmV1bFw",
        "href": "https://platform.whereismytransport.com/api/lines/kYdaW1_dKUe7WZegmV1bFw",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
            "name": "MyCiTi",
            "culture": "en"
        },
        "name": "A01 - Airport to Civic Centre",
        "mode": "Bus",
        "colour": "#ffa7a9ac",
        "textColour": "#ffffffff"
    }
]
```

#### Retrieving a specific line

Retrieves a line by its identifier.

`GET api/lines/{lineId}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| lineId | [Identifier](#identifiers) | The identifier of the line. |

##### Sample request

```
GET api/lines/rBD_j-ZRdEiiHMc9lNzQtA
```

##### Sample response

```
200 Ok
{
    "id": "rBD_j-ZRdEiiHMc9lNzQtA",
    "href": "https://platform.whereismytransport.com/api/lines/rBD_j-ZRdEiiHMc9lNzQtA",
    "agency": {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
        "name": "MyCiTi",
        "culture": "en"
    },
    "name": "Route 102 to Civic Centre",
    "mode": "Bus",
    "colour": "#ff7fd3f9",
    "textColour": "#ffffffff"
}
```

### Line Timetables

A timetable of vehicles travelling on a line.

#### Line Timetable response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| vehicle | [Vehicle](#vehicle-model) | If available, identifying information for the vehicle running at this time. |
| waypoints | Array of [Waypoint](#waypoint-model) | The sequence of ordered way points that make up this line timetable. |

#### Retrieving a line timetable

Retrieves a timetable for a line, consisting of a list of departures on this line in order of departure time.

`GET api/lines/{lineId}/timetables?earliestDepartureTime={DateTime}&departureStopId={stop}&arrivalStopId={stop}&limit={int}`

| Query Parameter | Type | Notes |
| :-------------- | :--- | :---- |
| lineId | [Identifier](#identifiers) | Required line identifier to get timetables by. |
| earliestDepartureTime | [DateTime](#datetime) | Optional earliest departure time on that line to be included in the timetable. |
| departureStopId | [Identifier](#identifiers) | Optional stop identifier - bounds results to only occur after this stop. |
| arrivalStopId | [Identifier](#identifiers) | Optional stop identifier - bounds results to only occur before this stop. |
| limit | int | The maximum number of entities to be returned. Default is 10. |

#####Sample request

```
GET api/lines/rBD_j-ZRdEiiHMc9lNzQtA/timetables?departureStopId=fbJnFbrZ906L0_jC09_eJw&limit=2&exclude=stop
```

#####Sample response

```
200 Ok
[
    {
        "vehicle": {
            "direction": "OneDirection"
        },
        "waypoints": [
            {
                "stop": {
                    "id": "fbJnFbrZ906L0_jC09_eJw",
                    "href": "https://platform.whereismytransport.com/api/stops/fbJnFbrZ906L0_jC09_eJw"
                },
                "arrivalTime": "2016-08-29T16:17:00Z",
                "departureTime": "2016-08-29T16:17:00Z"
            },
            {
                "stop": {
                    "id": "eBTeYLPXOkWm5zyfjZVaZg",
                    "href": "https://platform.whereismytransport.com/api/stops/eBTeYLPXOkWm5zyfjZVaZg"
                },
                "arrivalTime": "2016-08-29T16:22:00Z",
                "departureTime": "2016-08-29T16:22:00Z"
            },
            {
                "stop": {
                    "id": "84AqtsFm-0yAwX67o9-2vg",
                    "href": "https://platform.whereismytransport.com/api/stops/84AqtsFm-0yAwX67o9-2vg"
                },
                "arrivalTime": "2016-08-29T16:26:00Z",
                "departureTime": "2016-08-29T16:26:00Z"
            }
        ]
    },
    {
        "vehicle": {
            "direction": "OneDirection"
        },
        "waypoints": [
            {
                "stop": {
                    "id": "fbJnFbrZ906L0_jC09_eJw",
                    "href": "https://platform.whereismytransport.com/api/stops/fbJnFbrZ906L0_jC09_eJw"
                },
                "arrivalTime": "2016-08-30T16:17:00Z",
                "departureTime": "2016-08-30T16:17:00Z"
            },
            {
                "stop": {
                    "id": "eBTeYLPXOkWm5zyfjZVaZg",
                    "href": "https://platform.whereismytransport.com/api/stops/eBTeYLPXOkWm5zyfjZVaZg"
                },
                "arrivalTime": "2016-08-30T16:22:00Z",
                "departureTime": "2016-08-30T16:22:00Z"
            },
            {
                "stop": {
                    "id": "84AqtsFm-0yAwX67o9-2vg",
                    "href": "https://platform.whereismytransport.com/api/stops/84AqtsFm-0yAwX67o9-2vg"
                },
                "arrivalTime": "2016-08-30T16:26:00Z",
                "departureTime": "2016-08-30T16:26:00Z"
            }
        ]
    }
]
```

### Journeys

A journey is the traveling of a passenger from a departure point to an arrival point.  A journey can consist of zero to many possible itineraries, each a travel option in getting from A to B. An itinerary consists of one to many legs, describing the path and mode of transport, to take in order to complete the journey.

#### Journey response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the journey. |
| href | string | The hyperlink pointing to the journey. |
| geometry | [GeoJSON](#geojson) MultiPoint | An ordered GeoJSON MultiPoint representing the departure and arrival points for the the journey. |
| time | [DateTime](#datetime) | The requested date and time for the journey.  |
| timeType | [TimeType](#time-type) | Specifies whether this is an ArriveBefore or DepartAfter request. |
| profile | [Profile](#profile) | The profile used to calculate and order itineraries. |
| fareProducts | Array of [Identifier](#identifiers) | The list of [fare products](#fare-products) identifiers to try to use when calculating the journey fare. |
| maxItineraries | integer | The maximum number of itineraries to return. |
| only | [Filter](#filter) | The explicit set of modes and agencies used. |
| omit | [Filter](#filter) | The explicit set of modes and agencies omitted. |
| itineraries | Array of [Itinerary](#itinerary-model) | **[**[Excludable](#excludable)**]** The available itineraries for this journey. |

#### Creating a journey

Creating a new journey is done by posting the journey's criteria to the resource.

`POST api/journeys`

| Field | Type | Required | Description |
| :--------- | :--- | :--- | :---- |
| geometry | [GeoJSON](#geojson) MultiPoint | Required | An ordered GeoJSON MultiPoint representing the departure and arrival points for the the journey. Exactly two points must be provided. |
| time | [DateTime](#datetime) | Optional | The requested date and time for the journey. Defaults to Now. |
| timeType | [TimeType](#time-type) | Optional | Specifies whether this is an ArriveBefore or DepartAfter request. Defaults to DepartAfter. |
| profile | [Profile](#profile) | Required | The profile used to calculate and order itineraries. |
| fareProducts | Array of [Identifier](#identifiers) | Optional | The list of [fare products](#fare-products) identifiers to try to use when calculating the journey fare. |
| maxItineraries | integer | Optional | The maximum number of itineraries to return. This must be a value between or including 1 and 5. Default is 3. |
| only | [Filter](#filter) | Optional | The explicit set of modes or agencies to use. If unset, all modes and agencies are used. |
| omit | [Filter](#filter) | Optional | The explicit set of modes or agencies to exclude. Omit will always take preference. |

### Time Type

The time type specifies can either be **DepartAfter** or **ArriveBefore**.

**DepartAfter** (the default) indicates that the journey must be calculated to depart after the specified time, at the earliest.

**ArriveBefore** indicates that the journey must be calculated to arrive before the specified time, at the latest.

### Profile

The profile specified how the itineraries should be prioritised.

**ClosestToTime** (the default) returns itineraries absolutely closest to the requested date; earliest for **DepartAfter**, and latest for **ArriveBefore**.

**FewestTransfers** returns itineraries with fewest connections between vehicles, and then also prioritising closest to time.

### Filter

The profile used to calculate the itinerary options for a journey.

| Field | Type | Description |
| :--------- | :--- | :--- |
| agencies | Array of [Identifier](#identifiers) | A list of agencies to consider for the filter. |
| modes | Array of [Mode](#mode) | A list of modes to consider for the filter. |

##### Sample request

```
POST api/journeys
{
    "geometry": {
        "type": "MultiPoint",
        "coordinates": [
            [
                18.422326,
                -33.922843
            ],
            [
                18.473162,
                -33.966573
            ]
        ]
    },
    "omit": {
        "modes": [
            "Rail"
        ]
    },
    "maxItineraries": 1
}
```

##### Sample response

```
201 Created
Content-Type: application/json
{
    "id": "PEP5VsjJ6kuo6KZxAQjq2Q",
    "href": "https://platform.whereismytransport.com/api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q",
    "geometry": {
        "type": "MultiPoint",
        "coordinates": [
            [
                18.422326,
                -33.922843
            ],
            [
                18.473162,
                -33.966573
            ]
        ]
    },
    "time": "2016-08-29T16:04:32Z",
    "timeType": "DepartAfter",
    "profile": "ClosestToTime",
    "fareProducts": [],
    "maxItineraries": 1,
    "only": {
        "agencies": [],
        "modes": []
    },
    "omit": {
        "agencies": [],
        "modes": [
            "Rail"
        ]
    },
    "itineraries": [
        {
            "id": "5CxmV36Blk6n6qZxAQjrdQ",
            "href": "https://platform.whereismytransport.com/api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q/itineraries/5CxmV36Blk6n6qZxAQjrdQ",
            "departureTime": "2016-08-29T16:43:31Z",
            "arrivalTime": "2016-08-29T17:50:40Z",
            "distance": {
                "value": 14635,
                "unit": "m"
            },
            "duration": 3429,
            "legs": [
                {
                    "href": "https://platform.whereismytransport.com/api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q/itineraries/5CxmV36Blk6n6qZxAQjrdQ/legs/0",
                    "type": "Walking",
                    "distance": {
                        "value": 1372,
                        "unit": "m"
                    },
                    "duration": 988,
                    "waypoints": [
                        {
                            "location": {
                                "address": "11 Adderley, Parliament Street, Cape Town Ward 77, Cape Town Subcouncil 16, Cape Town, City of Cape Town, Western Cape, 8001, South Africa",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.422326,
                                        -33.922843
                                    ]
                                }
                            },
                            "arrivalTime": "2016-08-29T16:43:31Z",
                            "departureTime": "2016-08-29T16:43:31Z"
                        },
                        {
                            "stop": {
                                "id": "McWcQewKAUCZbluWHQk5kQ",
                                "href": "https://platform.whereismytransport.com/api/stops/McWcQewKAUCZbluWHQk5kQ",
                                "agency": {
                                    "id": "2yQYhQPxpEeYUIprNP__TQ",
                                    "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                                    "name": "Jammie Shuttle",
                                    "culture": "en"
                                },
                                "name": "Hiddingh Hall",
                                "code": "754628200",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.413234,
                                        -33.929981
                                    ]
                                },
                                "modes": [
                                    "Bus"
                                ]
                            },
                            "arrivalTime": "2016-08-29T17:00:00Z",
                            "departureTime": "2016-08-29T17:00:00Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.4223,
                                -33.922827
                            ],
                            [
                                18.42168,
                                -33.923467
                            ],
                            [
                                18.421217,
                                -33.923917
                            ],
                            [
                                18.420237,
                                -33.92481
                            ],
                            [
                                18.42014,
                                -33.924868
                            ],
                            [
                                18.420078,
                                -33.924887
                            ],
                            [
                                18.420015,
                                -33.924886
                            ],
                            [
                                18.419058,
                                -33.925823
                            ],
                            [
                                18.418622,
                                -33.925791
                            ],
                            [
                                18.41841,
                                -33.925824
                            ],
                            [
                                18.418241,
                                -33.925905
                            ],
                            [
                                18.418014,
                                -33.926072
                            ],
                            [
                                18.417499,
                                -33.926566
                            ],
                            [
                                18.417441,
                                -33.926552
                            ],
                            [
                                18.417388,
                                -33.926577
                            ],
                            [
                                18.417372,
                                -33.926624
                            ],
                            [
                                18.417401,
                                -33.926668
                            ],
                            [
                                18.416866,
                                -33.927199
                            ],
                            [
                                18.416883,
                                -33.927226
                            ],
                            [
                                18.416863,
                                -33.927284
                            ],
                            [
                                18.416793,
                                -33.927301
                            ],
                            [
                                18.416759,
                                -33.927286
                            ],
                            [
                                18.416138,
                                -33.927863
                            ],
                            [
                                18.416099,
                                -33.927834
                            ],
                            [
                                18.415707,
                                -33.9282
                            ],
                            [
                                18.415237,
                                -33.927869
                            ],
                            [
                                18.414735,
                                -33.928346
                            ],
                            [
                                18.414637,
                                -33.928372
                            ],
                            [
                                18.414562,
                                -33.928367
                            ],
                            [
                                18.414347,
                                -33.928281
                            ],
                            [
                                18.414215,
                                -33.928229
                            ],
                            [
                                18.413789,
                                -33.928104
                            ],
                            [
                                18.413662,
                                -33.928084
                            ],
                            [
                                18.413411,
                                -33.928568
                            ],
                            [
                                18.413096,
                                -33.929245
                            ],
                            [
                                18.412878,
                                -33.929749
                            ],
                            [
                                18.412986,
                                -33.929745
                            ],
                            [
                                18.413276,
                                -33.929937
                            ],
                            [
                                18.41326,
                                -33.92993
                            ]
                        ]
                    },
                    "directions": [
                        {
                            "instruction": "Continue onto Adderley Street",
                            "distance": {
                                "value": 314,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left onto Government Avenue",
                            "distance": {
                                "value": 136,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right",
                            "distance": {
                                "value": 502,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left onto Queen Victoria Street",
                            "distance": {
                                "value": 109,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Continue onto Grey's Pass",
                            "distance": {
                                "value": 67,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left onto Orange Street, M3",
                            "distance": {
                                "value": 199,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn sharp left",
                            "distance": {
                                "value": 44,
                                "unit": "m"
                            }
                        }
                    ]
                },
                {
                    "href": "https://platform.whereismytransport.com/api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q/itineraries/5CxmV36Blk6n6qZxAQjrdQ/legs/1",
                    "type": "Transit",
                    "distance": {
                        "value": 8554,
                        "unit": "m"
                    },
                    "duration": 1200,
                    "line": {
                        "id": "yGZHGrc3sUOhNFLoer-Z_g",
                        "href": "https://platform.whereismytransport.com/api/lines/yGZHGrc3sUOhNFLoer-Z_g",
                        "agency": {
                            "id": "2yQYhQPxpEeYUIprNP__TQ",
                            "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                            "name": "Jammie Shuttle",
                            "culture": "en"
                        },
                        "name": "Route 10 to UCT South",
                        "shortName": "Route 10",
                        "mode": "Bus",
                        "colour": "#ffeccb69",
                        "textColour": "#ffffffff"
                    },
                    "vehicle": {},
                    "waypoints": [
                        {
                            "stop": {
                                "id": "McWcQewKAUCZbluWHQk5kQ",
                                "href": "https://platform.whereismytransport.com/api/stops/McWcQewKAUCZbluWHQk5kQ",
                                "agency": {
                                    "id": "2yQYhQPxpEeYUIprNP__TQ",
                                    "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                                    "name": "Jammie Shuttle",
                                    "culture": "en"
                                },
                                "name": "Hiddingh Hall",
                                "code": "754628200",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.413234,
                                        -33.929981
                                    ]
                                },
                                "modes": [
                                    "Bus"
                                ]
                            },
                            "arrivalTime": "2016-08-29T17:00:00Z",
                            "departureTime": "2016-08-29T17:00:00Z"
                        },
                        {
                            "stop": {
                                "id": "C5UPegWudUa0h8LSSsVvrg",
                                "href": "https://platform.whereismytransport.com/api/stops/C5UPegWudUa0h8LSSsVvrg",
                                "agency": {
                                    "id": "2yQYhQPxpEeYUIprNP__TQ",
                                    "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                                    "name": "Jammie Shuttle",
                                    "culture": "en"
                                },
                                "name": "UCT South",
                                "code": "1383136465",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.459573,
                                        -33.960395
                                    ]
                                },
                                "modes": [
                                    "Bus"
                                ]
                            },
                            "arrivalTime": "2016-08-29T17:20:00Z",
                            "departureTime": "2016-08-29T17:20:00Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.41326,
                                -33.92993
                            ],
                            [
                                18.41326,
                                -33.92993
                            ],
                            [
                                18.41324,
                                -33.92991
                            ],
                            [
                                18.41317,
                                -33.92986
                            ],
                            [
                                18.41293,
                                -33.9297
                            ],
                            [
                                18.4129,
                                -33.92976
                            ],
                            [
                                18.41279,
                                -33.92999
                            ],
                            [
                                18.41278,
                                -33.9301
                            ],
                            [
                                18.41279,
                                -33.93016
                            ],
                            [
                                18.41282,
                                -33.93027
                            ],
                            [
                                18.41284,
                                -33.9303
                            ],
                            [
                                18.41286,
                                -33.93037
                            ],
                            [
                                18.41293,
                                -33.93048
                            ],
                            [
                                18.41306,
                                -33.93068
                            ],
                            [
                                18.41333,
                                -33.9311
                            ],
                            [
                                18.41338,
                                -33.93118
                            ],
                            [
                                18.41352,
                                -33.93127
                            ],
                            [
                                18.41371,
                                -33.93153
                            ],
                            [
                                18.414,
                                -33.93196
                            ],
                            [
                                18.41446,
                                -33.93265
                            ],
                            [
                                18.41469,
                                -33.93297
                            ],
                            [
                                18.41479,
                                -33.9331
                            ],
                            [
                                18.41489,
                                -33.93327
                            ],
                            [
                                18.41495,
                                -33.93334
                            ],
                            [
                                18.415,
                                -33.93339
                            ],
                            [
                                18.41503,
                                -33.93342
                            ],
                            [
                                18.41513,
                                -33.93351
                            ],
                            [
                                18.41522,
                                -33.93358
                            ],
                            [
                                18.41538,
                                -33.93366
                            ],
                            [
                                18.41549,
                                -33.93371
                            ],
                            [
                                18.41561,
                                -33.93374
                            ],
                            [
                                18.41574,
                                -33.93377
                            ],
                            [
                                18.41583,
                                -33.93379
                            ],
                            [
                                18.41586,
                                -33.93379
                            ],
                            [
                                18.41594,
                                -33.9338
                            ],
                            [
                                18.41605,
                                -33.9338
                            ],
                            [
                                18.41614,
                                -33.9338
                            ],
                            [
                                18.4162,
                                -33.9338
                            ],
                            [
                                18.41692,
                                -33.93383
                            ],
                            [
                                18.41707,
                                -33.93383
                            ],
                            [
                                18.41727,
                                -33.93384
                            ],
                            [
                                18.41738,
                                -33.93385
                            ],
                            [
                                18.41786,
                                -33.93387
                            ],
                            [
                                18.41814,
                                -33.93388
                            ],
                            [
                                18.41836,
                                -33.93389
                            ],
                            [
                                18.41864,
                                -33.93392
                            ],
                            [
                                18.41885,
                                -33.93395
                            ],
                            [
                                18.41914,
                                -33.934
                            ],
                            [
                                18.41926,
                                -33.93401
                            ],
                            [
                                18.41963,
                                -33.93406
                            ],
                            [
                                18.41986,
                                -33.93409
                            ],
                            [
                                18.42009,
                                -33.93412
                            ],
                            [
                                18.42031,
                                -33.93416
                            ],
                            [
                                18.42048,
                                -33.9342
                            ],
                            [
                                18.42069,
                                -33.93423
                            ],
                            [
                                18.42089,
                                -33.93426
                            ],
                            [
                                18.42093,
                                -33.93426
                            ],
                            [
                                18.42102,
                                -33.93427
                            ],
                            [
                                18.42132,
                                -33.93429
                            ],
                            [
                                18.42156,
                                -33.93431
                            ],
                            [
                                18.42182,
                                -33.93431
                            ],
                            [
                                18.42196,
                                -33.93431
                            ],
                            [
                                18.42211,
                                -33.93431
                            ],
                            [
                                18.42231,
                                -33.9343
                            ],
                            [
                                18.42257,
                                -33.93428
                            ],
                            [
                                18.42263,
                                -33.93428
                            ],
                            [
                                18.42303,
                                -33.93426
                            ],
                            [
                                18.42345,
                                -33.93424
                            ],
                            [
                                18.42371,
                                -33.93423
                            ],
                            [
                                18.42396,
                                -33.93422
                            ],
                            [
                                18.42421,
                                -33.93422
                            ],
                            [
                                18.42426,
                                -33.93421
                            ],
                            [
                                18.42432,
                                -33.9342
                            ],
                            [
                                18.42447,
                                -33.93416
                            ],
                            [
                                18.42455,
                                -33.93414
                            ],
                            [
                                18.42462,
                                -33.93412
                            ],
                            [
                                18.42476,
                                -33.93406
                            ],
                            [
                                18.42491,
                                -33.93399
                            ],
                            [
                                18.42506,
                                -33.93392
                            ],
                            [
                                18.42526,
                                -33.93381
                            ],
                            [
                                18.42529,
                                -33.9338
                            ],
                            [
                                18.42537,
                                -33.93375
                            ],
                            [
                                18.4255,
                                -33.93369
                            ],
                            [
                                18.42563,
                                -33.93363
                            ],
                            [
                                18.42576,
                                -33.93359
                            ],
                            [
                                18.42588,
                                -33.93355
                            ],
                            [
                                18.42595,
                                -33.93353
                            ],
                            [
                                18.42605,
                                -33.93349
                            ],
                            [
                                18.42611,
                                -33.93348
                            ],
                            [
                                18.42619,
                                -33.93346
                            ],
                            [
                                18.42626,
                                -33.93344
                            ],
                            [
                                18.42627,
                                -33.93344
                            ],
                            [
                                18.42643,
                                -33.93345
                            ],
                            [
                                18.42648,
                                -33.93346
                            ],
                            [
                                18.42652,
                                -33.93346
                            ],
                            [
                                18.42659,
                                -33.93349
                            ],
                            [
                                18.4267,
                                -33.93354
                            ],
                            [
                                18.4268,
                                -33.9336
                            ],
                            [
                                18.42688,
                                -33.93364
                            ],
                            [
                                18.42697,
                                -33.93369
                            ],
                            [
                                18.42705,
                                -33.93372
                            ],
                            [
                                18.4271,
                                -33.93374
                            ],
                            [
                                18.42715,
                                -33.93376
                            ],
                            [
                                18.42724,
                                -33.93378
                            ],
                            [
                                18.42733,
                                -33.93381
                            ],
                            [
                                18.4277,
                                -33.93389
                            ],
                            [
                                18.42776,
                                -33.9339
                            ],
                            [
                                18.4278,
                                -33.93391
                            ],
                            [
                                18.42797,
                                -33.93394
                            ],
                            [
                                18.42814,
                                -33.93397
                            ],
                            [
                                18.42826,
                                -33.93399
                            ],
                            [
                                18.42855,
                                -33.93404
                            ],
                            [
                                18.42864,
                                -33.93406
                            ],
                            [
                                18.42882,
                                -33.93408
                            ],
                            [
                                18.42891,
                                -33.9341
                            ],
                            [
                                18.42937,
                                -33.93416
                            ],
                            [
                                18.42953,
                                -33.93419
                            ],
                            [
                                18.42966,
                                -33.93423
                            ],
                            [
                                18.42994,
                                -33.93433
                            ],
                            [
                                18.43043,
                                -33.93451
                            ],
                            [
                                18.43115,
                                -33.93478
                            ],
                            [
                                18.43156,
                                -33.93493
                            ],
                            [
                                18.43163,
                                -33.93496
                            ],
                            [
                                18.43184,
                                -33.93504
                            ],
                            [
                                18.43206,
                                -33.9351
                            ],
                            [
                                18.43249,
                                -33.93522
                            ],
                            [
                                18.43308,
                                -33.93537
                            ],
                            [
                                18.43339,
                                -33.93545
                            ],
                            [
                                18.43353,
                                -33.93549
                            ],
                            [
                                18.43369,
                                -33.93555
                            ],
                            [
                                18.43382,
                                -33.93562
                            ],
                            [
                                18.43394,
                                -33.9357
                            ],
                            [
                                18.43405,
                                -33.93579
                            ],
                            [
                                18.43416,
                                -33.93594
                            ],
                            [
                                18.43423,
                                -33.93605
                            ],
                            [
                                18.43428,
                                -33.93615
                            ],
                            [
                                18.43439,
                                -33.93636
                            ],
                            [
                                18.43445,
                                -33.93646
                            ],
                            [
                                18.43452,
                                -33.93656
                            ],
                            [
                                18.43458,
                                -33.93663
                            ],
                            [
                                18.43461,
                                -33.93667
                            ],
                            [
                                18.43474,
                                -33.93679
                            ],
                            [
                                18.43484,
                                -33.93685
                            ],
                            [
                                18.43494,
                                -33.9369
                            ],
                            [
                                18.435,
                                -33.93692
                            ],
                            [
                                18.43509,
                                -33.93695
                            ],
                            [
                                18.4352,
                                -33.93698
                            ],
                            [
                                18.43532,
                                -33.937
                            ],
                            [
                                18.43565,
                                -33.93703
                            ],
                            [
                                18.43569,
                                -33.93703
                            ],
                            [
                                18.43579,
                                -33.93704
                            ],
                            [
                                18.43593,
                                -33.93707
                            ],
                            [
                                18.43609,
                                -33.9371
                            ],
                            [
                                18.43616,
                                -33.93712
                            ],
                            [
                                18.43627,
                                -33.93714
                            ],
                            [
                                18.43647,
                                -33.93722
                            ],
                            [
                                18.43666,
                                -33.93732
                            ],
                            [
                                18.43685,
                                -33.93743
                            ],
                            [
                                18.43699,
                                -33.93751
                            ],
                            [
                                18.43717,
                                -33.93762
                            ],
                            [
                                18.43728,
                                -33.93769
                            ],
                            [
                                18.43738,
                                -33.93772
                            ],
                            [
                                18.43747,
                                -33.93775
                            ],
                            [
                                18.43764,
                                -33.93777
                            ],
                            [
                                18.43812,
                                -33.93781
                            ],
                            [
                                18.43909,
                                -33.93789
                            ],
                            [
                                18.43947,
                                -33.93793
                            ],
                            [
                                18.43954,
                                -33.93794
                            ],
                            [
                                18.43969,
                                -33.93797
                            ],
                            [
                                18.43983,
                                -33.938
                            ],
                            [
                                18.43991,
                                -33.93803
                            ],
                            [
                                18.44006,
                                -33.9381
                            ],
                            [
                                18.44035,
                                -33.93829
                            ],
                            [
                                18.44062,
                                -33.93848
                            ],
                            [
                                18.44073,
                                -33.93854
                            ],
                            [
                                18.44084,
                                -33.9386
                            ],
                            [
                                18.44097,
                                -33.93865
                            ],
                            [
                                18.44101,
                                -33.93866
                            ],
                            [
                                18.4411,
                                -33.93869
                            ],
                            [
                                18.44126,
                                -33.93871
                            ],
                            [
                                18.44145,
                                -33.9387
                            ],
                            [
                                18.44159,
                                -33.93868
                            ],
                            [
                                18.44168,
                                -33.93867
                            ],
                            [
                                18.44181,
                                -33.93862
                            ],
                            [
                                18.44197,
                                -33.93854
                            ],
                            [
                                18.44218,
                                -33.93845
                            ],
                            [
                                18.4425,
                                -33.9383
                            ],
                            [
                                18.4426,
                                -33.93827
                            ],
                            [
                                18.44277,
                                -33.93821
                            ],
                            [
                                18.44284,
                                -33.93819
                            ],
                            [
                                18.44291,
                                -33.93818
                            ],
                            [
                                18.44313,
                                -33.93815
                            ],
                            [
                                18.44405,
                                -33.93803
                            ],
                            [
                                18.44429,
                                -33.93802
                            ],
                            [
                                18.44444,
                                -33.93802
                            ],
                            [
                                18.44458,
                                -33.93804
                            ],
                            [
                                18.44473,
                                -33.93807
                            ],
                            [
                                18.44487,
                                -33.9381
                            ],
                            [
                                18.4467,
                                -33.93867
                            ],
                            [
                                18.44709,
                                -33.93879
                            ],
                            [
                                18.44735,
                                -33.93888
                            ],
                            [
                                18.44755,
                                -33.93897
                            ],
                            [
                                18.44769,
                                -33.93905
                            ],
                            [
                                18.44782,
                                -33.93912
                            ],
                            [
                                18.44797,
                                -33.93922
                            ],
                            [
                                18.44819,
                                -33.93939
                            ],
                            [
                                18.44828,
                                -33.93948
                            ],
                            [
                                18.44837,
                                -33.93956
                            ],
                            [
                                18.44853,
                                -33.93976
                            ],
                            [
                                18.44858,
                                -33.93983
                            ],
                            [
                                18.44872,
                                -33.93997
                            ],
                            [
                                18.44874,
                                -33.93999
                            ],
                            [
                                18.44883,
                                -33.94007
                            ],
                            [
                                18.44904,
                                -33.94019
                            ],
                            [
                                18.44917,
                                -33.94025
                            ],
                            [
                                18.44934,
                                -33.9403
                            ],
                            [
                                18.44945,
                                -33.94032
                            ],
                            [
                                18.44954,
                                -33.94033
                            ],
                            [
                                18.44965,
                                -33.94033
                            ],
                            [
                                18.44991,
                                -33.94031
                            ],
                            [
                                18.45001,
                                -33.9403
                            ],
                            [
                                18.45025,
                                -33.94025
                            ],
                            [
                                18.45052,
                                -33.94018
                            ],
                            [
                                18.4507,
                                -33.94014
                            ],
                            [
                                18.45091,
                                -33.94011
                            ],
                            [
                                18.45117,
                                -33.9401
                            ],
                            [
                                18.45143,
                                -33.94011
                            ],
                            [
                                18.45201,
                                -33.94016
                            ],
                            [
                                18.45226,
                                -33.94019
                            ],
                            [
                                18.45241,
                                -33.94019
                            ],
                            [
                                18.45258,
                                -33.94018
                            ],
                            [
                                18.45272,
                                -33.94016
                            ],
                            [
                                18.45299,
                                -33.94013
                            ],
                            [
                                18.45331,
                                -33.94007
                            ],
                            [
                                18.45338,
                                -33.94006
                            ],
                            [
                                18.45355,
                                -33.94003
                            ],
                            [
                                18.45376,
                                -33.94
                            ],
                            [
                                18.45396,
                                -33.93998
                            ],
                            [
                                18.45427,
                                -33.93996
                            ],
                            [
                                18.45465,
                                -33.93995
                            ],
                            [
                                18.45478,
                                -33.93995
                            ],
                            [
                                18.45492,
                                -33.93999
                            ],
                            [
                                18.45526,
                                -33.94001
                            ],
                            [
                                18.45577,
                                -33.94004
                            ],
                            [
                                18.45601,
                                -33.94005
                            ],
                            [
                                18.4563,
                                -33.94007
                            ],
                            [
                                18.45652,
                                -33.94009
                            ],
                            [
                                18.45676,
                                -33.94012
                            ],
                            [
                                18.45695,
                                -33.94015
                            ],
                            [
                                18.45714,
                                -33.94022
                            ],
                            [
                                18.45737,
                                -33.94033
                            ],
                            [
                                18.45753,
                                -33.94041
                            ],
                            [
                                18.45761,
                                -33.94047
                            ],
                            [
                                18.45782,
                                -33.94066
                            ],
                            [
                                18.45788,
                                -33.94075
                            ],
                            [
                                18.45794,
                                -33.94083
                            ],
                            [
                                18.45796,
                                -33.94088
                            ],
                            [
                                18.45803,
                                -33.94105
                            ],
                            [
                                18.45812,
                                -33.94137
                            ],
                            [
                                18.45818,
                                -33.94157
                            ],
                            [
                                18.45824,
                                -33.94173
                            ],
                            [
                                18.45829,
                                -33.9418
                            ],
                            [
                                18.4584,
                                -33.94218
                            ],
                            [
                                18.45843,
                                -33.94227
                            ],
                            [
                                18.45847,
                                -33.94235
                            ],
                            [
                                18.45856,
                                -33.94249
                            ],
                            [
                                18.4586,
                                -33.94255
                            ],
                            [
                                18.45866,
                                -33.94263
                            ],
                            [
                                18.45869,
                                -33.94266
                            ],
                            [
                                18.45878,
                                -33.94276
                            ],
                            [
                                18.45894,
                                -33.94289
                            ],
                            [
                                18.45911,
                                -33.94301
                            ],
                            [
                                18.45918,
                                -33.94305
                            ],
                            [
                                18.45932,
                                -33.9431
                            ],
                            [
                                18.45946,
                                -33.94315
                            ],
                            [
                                18.45959,
                                -33.94318
                            ],
                            [
                                18.45972,
                                -33.94319
                            ],
                            [
                                18.45985,
                                -33.9432
                            ],
                            [
                                18.46002,
                                -33.94321
                            ],
                            [
                                18.4602,
                                -33.9432
                            ],
                            [
                                18.46037,
                                -33.94318
                            ],
                            [
                                18.4606,
                                -33.94316
                            ],
                            [
                                18.46088,
                                -33.94314
                            ],
                            [
                                18.46122,
                                -33.94312
                            ],
                            [
                                18.4616,
                                -33.94309
                            ],
                            [
                                18.46189,
                                -33.94307
                            ],
                            [
                                18.46212,
                                -33.94305
                            ],
                            [
                                18.46234,
                                -33.94304
                            ],
                            [
                                18.46247,
                                -33.94304
                            ],
                            [
                                18.46258,
                                -33.94303
                            ],
                            [
                                18.46277,
                                -33.94304
                            ],
                            [
                                18.46293,
                                -33.94305
                            ],
                            [
                                18.46309,
                                -33.94305
                            ],
                            [
                                18.46317,
                                -33.94306
                            ],
                            [
                                18.46327,
                                -33.94307
                            ],
                            [
                                18.4634,
                                -33.94309
                            ],
                            [
                                18.46348,
                                -33.9431
                            ],
                            [
                                18.46365,
                                -33.94313
                            ],
                            [
                                18.46383,
                                -33.94317
                            ],
                            [
                                18.46398,
                                -33.9432
                            ],
                            [
                                18.46424,
                                -33.94325
                            ],
                            [
                                18.46461,
                                -33.94338
                            ],
                            [
                                18.46469,
                                -33.94341
                            ],
                            [
                                18.46477,
                                -33.94344
                            ],
                            [
                                18.46484,
                                -33.94347
                            ],
                            [
                                18.46491,
                                -33.94349
                            ],
                            [
                                18.4651,
                                -33.9436
                            ],
                            [
                                18.46528,
                                -33.94372
                            ],
                            [
                                18.46542,
                                -33.94382
                            ],
                            [
                                18.46546,
                                -33.94386
                            ],
                            [
                                18.46556,
                                -33.94396
                            ],
                            [
                                18.46565,
                                -33.94406
                            ],
                            [
                                18.46572,
                                -33.94417
                            ],
                            [
                                18.46578,
                                -33.94424
                            ],
                            [
                                18.46583,
                                -33.94438
                            ],
                            [
                                18.46586,
                                -33.94445
                            ],
                            [
                                18.46593,
                                -33.94458
                            ],
                            [
                                18.46619,
                                -33.94503
                            ],
                            [
                                18.46644,
                                -33.94543
                            ],
                            [
                                18.4665,
                                -33.94554
                            ],
                            [
                                18.46659,
                                -33.94569
                            ],
                            [
                                18.46668,
                                -33.94586
                            ],
                            [
                                18.46672,
                                -33.94597
                            ],
                            [
                                18.46675,
                                -33.94606
                            ],
                            [
                                18.46677,
                                -33.94616
                            ],
                            [
                                18.46678,
                                -33.94627
                            ],
                            [
                                18.46679,
                                -33.94634
                            ],
                            [
                                18.46679,
                                -33.94642
                            ],
                            [
                                18.46677,
                                -33.94667
                            ],
                            [
                                18.46676,
                                -33.9468
                            ],
                            [
                                18.46671,
                                -33.94728
                            ],
                            [
                                18.4667,
                                -33.94739
                            ],
                            [
                                18.4667,
                                -33.94749
                            ],
                            [
                                18.46671,
                                -33.94756
                            ],
                            [
                                18.46672,
                                -33.9477
                            ],
                            [
                                18.46676,
                                -33.94791
                            ],
                            [
                                18.46682,
                                -33.94819
                            ],
                            [
                                18.46698,
                                -33.94888
                            ],
                            [
                                18.46703,
                                -33.94904
                            ],
                            [
                                18.46704,
                                -33.94911
                            ],
                            [
                                18.46707,
                                -33.94921
                            ],
                            [
                                18.46708,
                                -33.94929
                            ],
                            [
                                18.46708,
                                -33.94939
                            ],
                            [
                                18.46708,
                                -33.94952
                            ],
                            [
                                18.46707,
                                -33.94962
                            ],
                            [
                                18.46705,
                                -33.94972
                            ],
                            [
                                18.46704,
                                -33.94977
                            ],
                            [
                                18.46702,
                                -33.94985
                            ],
                            [
                                18.46698,
                                -33.94995
                            ],
                            [
                                18.46693,
                                -33.95005
                            ],
                            [
                                18.46683,
                                -33.9503
                            ],
                            [
                                18.46667,
                                -33.95065
                            ],
                            [
                                18.4666,
                                -33.95079
                            ],
                            [
                                18.46651,
                                -33.95098
                            ],
                            [
                                18.46636,
                                -33.9513
                            ],
                            [
                                18.4662,
                                -33.95164
                            ],
                            [
                                18.46611,
                                -33.95182
                            ],
                            [
                                18.46602,
                                -33.95203
                            ],
                            [
                                18.4659,
                                -33.95227
                            ],
                            [
                                18.46577,
                                -33.95254
                            ],
                            [
                                18.46569,
                                -33.95266
                            ],
                            [
                                18.46561,
                                -33.95279
                            ],
                            [
                                18.46556,
                                -33.95287
                            ],
                            [
                                18.4654,
                                -33.95327
                            ],
                            [
                                18.46539,
                                -33.95339
                            ],
                            [
                                18.4654,
                                -33.95349
                            ],
                            [
                                18.46552,
                                -33.95381
                            ],
                            [
                                18.46554,
                                -33.95387
                            ],
                            [
                                18.46554,
                                -33.95394
                            ],
                            [
                                18.46551,
                                -33.95405
                            ],
                            [
                                18.4654,
                                -33.95425
                            ],
                            [
                                18.46532,
                                -33.95437
                            ],
                            [
                                18.46518,
                                -33.9546
                            ],
                            [
                                18.46513,
                                -33.95457
                            ],
                            [
                                18.46509,
                                -33.95455
                            ],
                            [
                                18.46504,
                                -33.95453
                            ],
                            [
                                18.46496,
                                -33.95449
                            ],
                            [
                                18.4649,
                                -33.95446
                            ],
                            [
                                18.46478,
                                -33.9544
                            ],
                            [
                                18.46464,
                                -33.95433
                            ],
                            [
                                18.46455,
                                -33.95429
                            ],
                            [
                                18.4645,
                                -33.95426
                            ],
                            [
                                18.46438,
                                -33.95417
                            ],
                            [
                                18.46434,
                                -33.95413
                            ],
                            [
                                18.4643,
                                -33.95407
                            ],
                            [
                                18.46428,
                                -33.95401
                            ],
                            [
                                18.46427,
                                -33.95395
                            ],
                            [
                                18.46424,
                                -33.95374
                            ],
                            [
                                18.46423,
                                -33.95365
                            ],
                            [
                                18.46418,
                                -33.95317
                            ],
                            [
                                18.46417,
                                -33.95308
                            ],
                            [
                                18.46416,
                                -33.953
                            ],
                            [
                                18.46415,
                                -33.95296
                            ],
                            [
                                18.46414,
                                -33.95293
                            ],
                            [
                                18.46411,
                                -33.95289
                            ],
                            [
                                18.46406,
                                -33.95284
                            ],
                            [
                                18.464,
                                -33.9528
                            ],
                            [
                                18.46398,
                                -33.95279
                            ],
                            [
                                18.46394,
                                -33.95277
                            ],
                            [
                                18.46386,
                                -33.95275
                            ],
                            [
                                18.46378,
                                -33.95275
                            ],
                            [
                                18.46368,
                                -33.95277
                            ],
                            [
                                18.46359,
                                -33.9528
                            ],
                            [
                                18.46352,
                                -33.95285
                            ],
                            [
                                18.46347,
                                -33.9529
                            ],
                            [
                                18.46344,
                                -33.95294
                            ],
                            [
                                18.46342,
                                -33.95301
                            ],
                            [
                                18.46337,
                                -33.95323
                            ],
                            [
                                18.46319,
                                -33.95424
                            ],
                            [
                                18.46314,
                                -33.95455
                            ],
                            [
                                18.46311,
                                -33.95469
                            ],
                            [
                                18.46302,
                                -33.95493
                            ],
                            [
                                18.46291,
                                -33.95522
                            ],
                            [
                                18.46285,
                                -33.95538
                            ],
                            [
                                18.46286,
                                -33.95544
                            ],
                            [
                                18.46284,
                                -33.95553
                            ],
                            [
                                18.46279,
                                -33.95561
                            ],
                            [
                                18.46272,
                                -33.9557
                            ],
                            [
                                18.46266,
                                -33.9557
                            ],
                            [
                                18.46258,
                                -33.95573
                            ],
                            [
                                18.4625,
                                -33.95574
                            ],
                            [
                                18.46242,
                                -33.95573
                            ],
                            [
                                18.4623,
                                -33.95568
                            ],
                            [
                                18.46223,
                                -33.95561
                            ],
                            [
                                18.46218,
                                -33.95553
                            ],
                            [
                                18.46215,
                                -33.95544
                            ],
                            [
                                18.46213,
                                -33.9553
                            ],
                            [
                                18.46212,
                                -33.95521
                            ],
                            [
                                18.46205,
                                -33.95513
                            ],
                            [
                                18.462,
                                -33.9551
                            ],
                            [
                                18.46192,
                                -33.95507
                            ],
                            [
                                18.46148,
                                -33.95496
                            ],
                            [
                                18.46132,
                                -33.95492
                            ],
                            [
                                18.46101,
                                -33.95485
                            ],
                            [
                                18.46087,
                                -33.95481
                            ],
                            [
                                18.46076,
                                -33.95479
                            ],
                            [
                                18.46046,
                                -33.95471
                            ],
                            [
                                18.46037,
                                -33.95471
                            ],
                            [
                                18.46028,
                                -33.95474
                            ],
                            [
                                18.4602,
                                -33.95479
                            ],
                            [
                                18.46014,
                                -33.95485
                            ],
                            [
                                18.4601,
                                -33.95494
                            ],
                            [
                                18.4601,
                                -33.95536
                            ],
                            [
                                18.4601,
                                -33.95556
                            ],
                            [
                                18.46008,
                                -33.95573
                            ],
                            [
                                18.46006,
                                -33.95578
                            ],
                            [
                                18.46003,
                                -33.95592
                            ],
                            [
                                18.45996,
                                -33.95611
                            ],
                            [
                                18.45982,
                                -33.95643
                            ],
                            [
                                18.45973,
                                -33.95666
                            ],
                            [
                                18.45964,
                                -33.95687
                            ],
                            [
                                18.45951,
                                -33.95722
                            ],
                            [
                                18.45934,
                                -33.95763
                            ],
                            [
                                18.45922,
                                -33.95793
                            ],
                            [
                                18.45899,
                                -33.95848
                            ],
                            [
                                18.45887,
                                -33.95877
                            ],
                            [
                                18.45884,
                                -33.95884
                            ],
                            [
                                18.45881,
                                -33.95894
                            ],
                            [
                                18.45879,
                                -33.95909
                            ],
                            [
                                18.4588,
                                -33.95918
                            ],
                            [
                                18.45881,
                                -33.95928
                            ],
                            [
                                18.45887,
                                -33.95946
                            ],
                            [
                                18.45892,
                                -33.95958
                            ],
                            [
                                18.459,
                                -33.9597
                            ],
                            [
                                18.45917,
                                -33.95994
                            ],
                            [
                                18.45945,
                                -33.96032
                            ],
                            [
                                18.45949,
                                -33.96037
                            ],
                            [
                                18.45953,
                                -33.96042
                            ]
                        ]
                    }
                },
                {
                    "href": "https://platform.whereismytransport.com/api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q/itineraries/5CxmV36Blk6n6qZxAQjrdQ/legs/2",
                    "type": "Transit",
                    "distance": {
                        "value": 3820,
                        "unit": "m"
                    },
                    "duration": 600,
                    "line": {
                        "id": "TP0onz0EikaqyZ8q3b_9gQ",
                        "href": "https://platform.whereismytransport.com/api/lines/TP0onz0EikaqyZ8q3b_9gQ",
                        "agency": {
                            "id": "2yQYhQPxpEeYUIprNP__TQ",
                            "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                            "name": "Jammie Shuttle",
                            "culture": "en"
                        },
                        "name": "Route 9B to UCT South",
                        "shortName": "Route 9B",
                        "mode": "Bus",
                        "colour": "#ff00a84e",
                        "textColour": "#ffffffff"
                    },
                    "vehicle": {
                        "direction": "OppositeDirection"
                    },
                    "waypoints": [
                        {
                            "stop": {
                                "id": "C5UPegWudUa0h8LSSsVvrg",
                                "href": "https://platform.whereismytransport.com/api/stops/C5UPegWudUa0h8LSSsVvrg",
                                "agency": {
                                    "id": "2yQYhQPxpEeYUIprNP__TQ",
                                    "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                                    "name": "Jammie Shuttle",
                                    "culture": "en"
                                },
                                "name": "UCT South",
                                "code": "1383136465",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.459573,
                                        -33.960395
                                    ]
                                },
                                "modes": [
                                    "Bus"
                                ]
                            },
                            "arrivalTime": "2016-08-29T17:30:00Z",
                            "departureTime": "2016-08-29T17:30:00Z"
                        },
                        {
                            "stop": {
                                "id": "HFL2KXjdbkuL4vn9QEW3Rw",
                                "href": "https://platform.whereismytransport.com/api/stops/HFL2KXjdbkuL4vn9QEW3Rw",
                                "agency": {
                                    "id": "2yQYhQPxpEeYUIprNP__TQ",
                                    "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                                    "name": "Jammie Shuttle",
                                    "culture": "en"
                                },
                                "name": "Tugwell",
                                "code": "1794862348",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.470761,
                                        -33.954354
                                    ]
                                },
                                "modes": [
                                    "Bus"
                                ]
                            },
                            "arrivalTime": "2016-08-29T17:35:00Z",
                            "departureTime": "2016-08-29T17:35:00Z"
                        },
                        {
                            "stop": {
                                "id": "G4Qx8shvl0qxTGCJhN1hlA",
                                "href": "https://platform.whereismytransport.com/api/stops/G4Qx8shvl0qxTGCJhN1hlA",
                                "agency": {
                                    "id": "2yQYhQPxpEeYUIprNP__TQ",
                                    "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                                    "name": "Jammie Shuttle",
                                    "culture": "en"
                                },
                                "name": "Groote Schuur",
                                "code": "1596544810",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.470524,
                                        -33.961312
                                    ]
                                },
                                "modes": [
                                    "Bus"
                                ]
                            },
                            "arrivalTime": "2016-08-29T17:40:00Z",
                            "departureTime": "2016-08-29T17:40:00Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.459539,
                                -33.960429
                            ],
                            [
                                18.45957,
                                -33.96046
                            ],
                            [
                                18.45972,
                                -33.96059
                            ],
                            [
                                18.45984,
                                -33.96068
                            ],
                            [
                                18.45995,
                                -33.96078
                            ],
                            [
                                18.45998,
                                -33.96084
                            ],
                            [
                                18.46001,
                                -33.96088
                            ],
                            [
                                18.46006,
                                -33.96098
                            ],
                            [
                                18.46008,
                                -33.96105
                            ],
                            [
                                18.4601,
                                -33.96114
                            ],
                            [
                                18.46012,
                                -33.96124
                            ],
                            [
                                18.46011,
                                -33.96133
                            ],
                            [
                                18.4601,
                                -33.96143
                            ],
                            [
                                18.46007,
                                -33.96154
                            ],
                            [
                                18.45978,
                                -33.96244
                            ],
                            [
                                18.4597,
                                -33.96265
                            ],
                            [
                                18.45955,
                                -33.96294
                            ],
                            [
                                18.45938,
                                -33.96321
                            ],
                            [
                                18.45932,
                                -33.96332
                            ],
                            [
                                18.45925,
                                -33.96346
                            ],
                            [
                                18.45921,
                                -33.96354
                            ],
                            [
                                18.45917,
                                -33.96359
                            ],
                            [
                                18.45916,
                                -33.96363
                            ],
                            [
                                18.45914,
                                -33.96368
                            ],
                            [
                                18.45909,
                                -33.96377
                            ],
                            [
                                18.45905,
                                -33.96386
                            ],
                            [
                                18.45898,
                                -33.96394
                            ],
                            [
                                18.45886,
                                -33.96405
                            ],
                            [
                                18.45881,
                                -33.96412
                            ],
                            [
                                18.45879,
                                -33.96416
                            ],
                            [
                                18.45879,
                                -33.96418
                            ],
                            [
                                18.45878,
                                -33.9642
                            ],
                            [
                                18.45879,
                                -33.96423
                            ],
                            [
                                18.45879,
                                -33.96426
                            ],
                            [
                                18.45884,
                                -33.96443
                            ],
                            [
                                18.45899,
                                -33.96433
                            ],
                            [
                                18.45918,
                                -33.9642
                            ],
                            [
                                18.45923,
                                -33.96415
                            ],
                            [
                                18.45929,
                                -33.96407
                            ],
                            [
                                18.45932,
                                -33.96401
                            ],
                            [
                                18.45937,
                                -33.96391
                            ],
                            [
                                18.45952,
                                -33.96354
                            ],
                            [
                                18.45967,
                                -33.96327
                            ],
                            [
                                18.45969,
                                -33.96321
                            ],
                            [
                                18.45987,
                                -33.96275
                            ],
                            [
                                18.46006,
                                -33.96214
                            ],
                            [
                                18.46017,
                                -33.96181
                            ],
                            [
                                18.46033,
                                -33.96134
                            ],
                            [
                                18.46039,
                                -33.96119
                            ],
                            [
                                18.46047,
                                -33.96107
                            ],
                            [
                                18.46059,
                                -33.96092
                            ],
                            [
                                18.46072,
                                -33.9608
                            ],
                            [
                                18.46086,
                                -33.96069
                            ],
                            [
                                18.46102,
                                -33.96059
                            ],
                            [
                                18.46112,
                                -33.96053
                            ],
                            [
                                18.46126,
                                -33.96047
                            ],
                            [
                                18.46133,
                                -33.96045
                            ],
                            [
                                18.46151,
                                -33.9604
                            ],
                            [
                                18.46178,
                                -33.96034
                            ],
                            [
                                18.46183,
                                -33.96033
                            ],
                            [
                                18.4621,
                                -33.96026
                            ],
                            [
                                18.46227,
                                -33.9602
                            ],
                            [
                                18.46234,
                                -33.96017
                            ],
                            [
                                18.46248,
                                -33.96011
                            ],
                            [
                                18.4626,
                                -33.96005
                            ],
                            [
                                18.4627,
                                -33.95999
                            ],
                            [
                                18.4628,
                                -33.95991
                            ],
                            [
                                18.46293,
                                -33.95979
                            ],
                            [
                                18.46301,
                                -33.9597
                            ],
                            [
                                18.46305,
                                -33.95963
                            ],
                            [
                                18.46311,
                                -33.95954
                            ],
                            [
                                18.46316,
                                -33.95943
                            ],
                            [
                                18.4632,
                                -33.95934
                            ],
                            [
                                18.46325,
                                -33.95923
                            ],
                            [
                                18.46332,
                                -33.95901
                            ],
                            [
                                18.46339,
                                -33.95879
                            ],
                            [
                                18.46351,
                                -33.95842
                            ],
                            [
                                18.46358,
                                -33.95816
                            ],
                            [
                                18.46365,
                                -33.9578
                            ],
                            [
                                18.46374,
                                -33.95725
                            ],
                            [
                                18.46376,
                                -33.95696
                            ],
                            [
                                18.46379,
                                -33.95652
                            ],
                            [
                                18.46381,
                                -33.95617
                            ],
                            [
                                18.46382,
                                -33.95605
                            ],
                            [
                                18.46382,
                                -33.95599
                            ],
                            [
                                18.46384,
                                -33.95583
                            ],
                            [
                                18.46385,
                                -33.95572
                            ],
                            [
                                18.46386,
                                -33.95562
                            ],
                            [
                                18.46392,
                                -33.95543
                            ],
                            [
                                18.46396,
                                -33.95531
                            ],
                            [
                                18.464,
                                -33.95521
                            ],
                            [
                                18.46405,
                                -33.9551
                            ],
                            [
                                18.46411,
                                -33.95499
                            ],
                            [
                                18.46447,
                                -33.95442
                            ],
                            [
                                18.46452,
                                -33.95435
                            ],
                            [
                                18.46455,
                                -33.95429
                            ],
                            [
                                18.46457,
                                -33.95426
                            ],
                            [
                                18.4647,
                                -33.95405
                            ],
                            [
                                18.46478,
                                -33.95382
                            ],
                            [
                                18.46479,
                                -33.95375
                            ],
                            [
                                18.46477,
                                -33.95369
                            ],
                            [
                                18.46475,
                                -33.95365
                            ],
                            [
                                18.46472,
                                -33.95361
                            ],
                            [
                                18.46467,
                                -33.95358
                            ],
                            [
                                18.46462,
                                -33.95355
                            ],
                            [
                                18.46455,
                                -33.95354
                            ],
                            [
                                18.4645,
                                -33.95355
                            ],
                            [
                                18.46447,
                                -33.95356
                            ],
                            [
                                18.46441,
                                -33.95358
                            ],
                            [
                                18.46435,
                                -33.95362
                            ],
                            [
                                18.4643,
                                -33.95367
                            ],
                            [
                                18.46424,
                                -33.95374
                            ],
                            [
                                18.46427,
                                -33.95395
                            ],
                            [
                                18.46428,
                                -33.95401
                            ],
                            [
                                18.4643,
                                -33.95407
                            ],
                            [
                                18.46434,
                                -33.95413
                            ],
                            [
                                18.46438,
                                -33.95417
                            ],
                            [
                                18.4645,
                                -33.95426
                            ],
                            [
                                18.46455,
                                -33.95429
                            ],
                            [
                                18.46464,
                                -33.95433
                            ],
                            [
                                18.46478,
                                -33.9544
                            ],
                            [
                                18.4649,
                                -33.95446
                            ],
                            [
                                18.46496,
                                -33.95449
                            ],
                            [
                                18.46504,
                                -33.95453
                            ],
                            [
                                18.46509,
                                -33.95455
                            ],
                            [
                                18.46513,
                                -33.95457
                            ],
                            [
                                18.46518,
                                -33.9546
                            ],
                            [
                                18.46531,
                                -33.95467
                            ],
                            [
                                18.46564,
                                -33.95483
                            ],
                            [
                                18.46589,
                                -33.95496
                            ],
                            [
                                18.46601,
                                -33.95503
                            ],
                            [
                                18.46613,
                                -33.9551
                            ],
                            [
                                18.46626,
                                -33.95515
                            ],
                            [
                                18.46636,
                                -33.95519
                            ],
                            [
                                18.46644,
                                -33.95521
                            ],
                            [
                                18.46658,
                                -33.95523
                            ],
                            [
                                18.46668,
                                -33.95525
                            ],
                            [
                                18.46679,
                                -33.95525
                            ],
                            [
                                18.46694,
                                -33.95524
                            ],
                            [
                                18.46708,
                                -33.95523
                            ],
                            [
                                18.46721,
                                -33.95522
                            ],
                            [
                                18.46731,
                                -33.95521
                            ],
                            [
                                18.4674,
                                -33.95521
                            ],
                            [
                                18.46751,
                                -33.95521
                            ],
                            [
                                18.46762,
                                -33.95521
                            ],
                            [
                                18.46775,
                                -33.95523
                            ],
                            [
                                18.46791,
                                -33.95527
                            ],
                            [
                                18.4681,
                                -33.95532
                            ],
                            [
                                18.46831,
                                -33.95537
                            ],
                            [
                                18.4684,
                                -33.95539
                            ],
                            [
                                18.46851,
                                -33.95542
                            ],
                            [
                                18.46869,
                                -33.95546
                            ],
                            [
                                18.46874,
                                -33.95547
                            ],
                            [
                                18.46876,
                                -33.95547
                            ],
                            [
                                18.46887,
                                -33.95547
                            ],
                            [
                                18.46913,
                                -33.95551
                            ],
                            [
                                18.46936,
                                -33.95553
                            ],
                            [
                                18.46962,
                                -33.95553
                            ],
                            [
                                18.46982,
                                -33.95552
                            ],
                            [
                                18.4701,
                                -33.95549
                            ],
                            [
                                18.47023,
                                -33.95548
                            ],
                            [
                                18.47035,
                                -33.95547
                            ],
                            [
                                18.47045,
                                -33.95547
                            ],
                            [
                                18.47046,
                                -33.95542
                            ],
                            [
                                18.47048,
                                -33.95535
                            ],
                            [
                                18.4705,
                                -33.95529
                            ],
                            [
                                18.47053,
                                -33.95523
                            ],
                            [
                                18.47055,
                                -33.95518
                            ],
                            [
                                18.47058,
                                -33.95512
                            ],
                            [
                                18.4706,
                                -33.95506
                            ],
                            [
                                18.47061,
                                -33.955
                            ],
                            [
                                18.47063,
                                -33.95494
                            ],
                            [
                                18.47064,
                                -33.95489
                            ],
                            [
                                18.47066,
                                -33.95481
                            ],
                            [
                                18.47069,
                                -33.95467
                            ],
                            [
                                18.47073,
                                -33.95453
                            ],
                            [
                                18.47074,
                                -33.95452
                            ],
                            [
                                18.47077,
                                -33.95446
                            ],
                            [
                                18.4708,
                                -33.95442
                            ],
                            [
                                18.47083,
                                -33.95438
                            ],
                            [
                                18.4708309,
                                -33.9543773
                            ],
                            [
                                18.4708309,
                                -33.9543773
                            ],
                            [
                                18.47084,
                                -33.95435
                            ],
                            [
                                18.47086,
                                -33.95425
                            ],
                            [
                                18.47087,
                                -33.95412
                            ],
                            [
                                18.47089,
                                -33.95403
                            ],
                            [
                                18.47093,
                                -33.95392
                            ],
                            [
                                18.47098,
                                -33.95378
                            ],
                            [
                                18.47113,
                                -33.95381
                            ],
                            [
                                18.47174,
                                -33.95393
                            ],
                            [
                                18.47202,
                                -33.95398
                            ],
                            [
                                18.472,
                                -33.95408
                            ],
                            [
                                18.47197,
                                -33.95425
                            ],
                            [
                                18.47195,
                                -33.95434
                            ],
                            [
                                18.47183,
                                -33.95484
                            ],
                            [
                                18.47172,
                                -33.95524
                            ],
                            [
                                18.47169,
                                -33.95536
                            ],
                            [
                                18.47159,
                                -33.95571
                            ],
                            [
                                18.47154,
                                -33.9559
                            ],
                            [
                                18.47151,
                                -33.95602
                            ],
                            [
                                18.47145,
                                -33.95628
                            ],
                            [
                                18.47135,
                                -33.95663
                            ],
                            [
                                18.47128,
                                -33.95686
                            ],
                            [
                                18.47126,
                                -33.95696
                            ],
                            [
                                18.47118,
                                -33.95722
                            ],
                            [
                                18.47107,
                                -33.95763
                            ],
                            [
                                18.47103,
                                -33.95777
                            ],
                            [
                                18.47097,
                                -33.95795
                            ],
                            [
                                18.47095,
                                -33.95808
                            ],
                            [
                                18.4709,
                                -33.95848
                            ],
                            [
                                18.47084,
                                -33.95889
                            ],
                            [
                                18.47082,
                                -33.959
                            ],
                            [
                                18.4708,
                                -33.95905
                            ],
                            [
                                18.47077,
                                -33.95914
                            ],
                            [
                                18.47074,
                                -33.95921
                            ],
                            [
                                18.47069,
                                -33.95929
                            ],
                            [
                                18.47046,
                                -33.95965
                            ],
                            [
                                18.4704,
                                -33.95977
                            ],
                            [
                                18.4703,
                                -33.95989
                            ],
                            [
                                18.47008,
                                -33.96024
                            ],
                            [
                                18.47005,
                                -33.96033
                            ],
                            [
                                18.47003,
                                -33.9604
                            ],
                            [
                                18.47001,
                                -33.9606
                            ],
                            [
                                18.46999,
                                -33.96079
                            ],
                            [
                                18.46998,
                                -33.96099
                            ],
                            [
                                18.46996,
                                -33.96114
                            ],
                            [
                                18.46993,
                                -33.96137
                            ],
                            [
                                18.47008,
                                -33.96137
                            ],
                            [
                                18.47034,
                                -33.96142
                            ],
                            [
                                18.47044,
                                -33.96148
                            ],
                            [
                                18.4705,
                                -33.96148
                            ],
                            [
                                18.47048,
                                -33.96143
                            ],
                            [
                                18.47048,
                                -33.9614
                            ],
                            [
                                18.47048,
                                -33.96136
                            ],
                            [
                                18.47051,
                                -33.96132
                            ],
                            [
                                18.47051,
                                -33.96131
                            ],
                            [
                                18.470513384615384,
                                -33.96130492307692
                            ]
                        ]
                    }
                },
                {
                    "href": "https://platform.whereismytransport.com/api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q/itineraries/5CxmV36Blk6n6qZxAQjrdQ/legs/3",
                    "type": "Walking",
                    "distance": {
                        "value": 889,
                        "unit": "m"
                    },
                    "duration": 640,
                    "waypoints": [
                        {
                            "stop": {
                                "id": "G4Qx8shvl0qxTGCJhN1hlA",
                                "href": "https://platform.whereismytransport.com/api/stops/G4Qx8shvl0qxTGCJhN1hlA",
                                "agency": {
                                    "id": "2yQYhQPxpEeYUIprNP__TQ",
                                    "href": "https://platform.whereismytransport.com/api/agencies/2yQYhQPxpEeYUIprNP__TQ",
                                    "name": "Jammie Shuttle",
                                    "culture": "en"
                                },
                                "name": "Groote Schuur",
                                "code": "1596544810",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.470524,
                                        -33.961312
                                    ]
                                },
                                "modes": [
                                    "Bus"
                                ]
                            },
                            "arrivalTime": "2016-08-29T17:40:00Z",
                            "departureTime": "2016-08-29T17:40:00Z"
                        },
                        {
                            "location": {
                                "address": "Albion Road, Cape Town Ward 59, Cape Town Subcouncil 20, Cape Town, City of Cape Town, Western Cape, 7700, South Africa",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.473162,
                                        -33.966573
                                    ]
                                }
                            },
                            "arrivalTime": "2016-08-29T17:50:40Z",
                            "departureTime": "2016-08-29T17:50:40Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.470513384615384,
                                -33.96130492307692
                            ],
                            [
                                18.470532,
                                -33.961314
                            ],
                            [
                                18.470503,
                                -33.96138
                            ],
                            [
                                18.470468,
                                -33.961538
                            ],
                            [
                                18.470796,
                                -33.961516
                            ],
                            [
                                18.471215,
                                -33.961435
                            ],
                            [
                                18.471522,
                                -33.961327
                            ],
                            [
                                18.471695,
                                -33.961727
                            ],
                            [
                                18.471799,
                                -33.962048
                            ],
                            [
                                18.472101,
                                -33.962758
                            ],
                            [
                                18.472137,
                                -33.962959
                            ],
                            [
                                18.472017,
                                -33.964536
                            ],
                            [
                                18.471867,
                                -33.965397
                            ],
                            [
                                18.471855,
                                -33.965467
                            ],
                            [
                                18.471944,
                                -33.965455
                            ],
                            [
                                18.471957,
                                -33.965386
                            ],
                            [
                                18.472121,
                                -33.965409
                            ],
                            [
                                18.472184,
                                -33.965395
                            ],
                            [
                                18.472202,
                                -33.965424
                            ],
                            [
                                18.472118,
                                -33.965457
                            ],
                            [
                                18.472007,
                                -33.965789
                            ],
                            [
                                18.471868,
                                -33.966313
                            ],
                            [
                                18.471809,
                                -33.966474
                            ],
                            [
                                18.4719,
                                -33.966491
                            ],
                            [
                                18.473164,
                                -33.966545
                            ]
                        ]
                    },
                    "directions": [
                        {
                            "instruction": "Continue",
                            "distance": {
                                "value": 25,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn sharp left onto Belmont Road, M92",
                            "distance": {
                                "value": 101,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right onto St. Andrews Road",
                            "distance": {
                                "value": 470,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn sharp left",
                            "distance": {
                                "value": 40,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right onto White Road",
                            "distance": {
                                "value": 125,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left onto Albion Road",
                            "distance": {
                                "value": 125,
                                "unit": "m"
                            }
                        }
                    ]
                }
            ]
        }
    ]
}
```

### Itineraries

To retrieve a specific itinerary for a previously created journey, the following resource can be requested.

`GET api/journeys/{journeyId}/itineraries/{itineraryId}`

| Parameter | Type | Description |
| :-------------- | :--- | :--- | :---- |
| journeyId | [Identifier](#identifiers) | The identifier of the journey. |
| itineraryId | [Identifier](#identifiers) | The identifier of the itinerary. |

##### Sample request

```
GET api/journeys/8GYKddjcAk6j7aVUAMV3pw/itineraries/dnCQV5Kq0kaq5KVUAMV_eQ
Accept: application/json
```

### Legs

A leg is a section of an itinerary carried out by a passenger on one mode of transport (including walking) from some departure point to an arrival point.

#### Types of legs

The API currently supports two types of legs, each with a different response model.

A _Walking_ leg is one which the passenger is to travel by foot from one waypoint to another.  Walking legs are usually accompanied with directions.

A _Transit_ leg is one which uses a public transportation service based on scheduled or absolute frequency-based stop times.

#### Leg response model {#leg-model}

| Field | Type | Description |
| :--------- | :--- | :---- |
| type | string | The [type of leg](#types-of-legs), either _Walking_ or _Transit_. |
| distance | [Distance](#distance) | If available, the total distance of leg. |
| duration | integer | If available, the total duration of the leg in seconds. |
| line | [Line](#line-response-model) | **[**[Excludable](#excludable)**]** The line that is used on this leg of the itinerary. This is only returned for _Transit_ legs. |
| vehicle | [Vehicle](#vehicle-model) | **[**[Excludable](#excludable)**]** Identifying information for the vehicle that is used on this leg of the itinerary. This is only returned for _Transit_ legs. |
| fare | [Fare](#fare-model) | If available, the fare for this leg. |
| waypoints | Array of [Waypoint](#waypoint-response-model) | **[**[Excludable](#excludable)**]** The sequence of ordered waypoints that make up this leg. |
| directions | Array of [Direction](#direction-response-model) | **[**[Excludable](#excludable)**]** If available, the directions to take in order to complete the leg. |
| geometry | [GeoJSON](#geojson) LineString | **[**[Excludable](#excludable)**]** If available, the geographic shape of the leg. |

#### Retrieving a specific leg

##### Sample request

Retrieving an itinerary's leg can be done using the index of that leg as it exists in the itinerary. The index begins counting from 1.

```
GET api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q/itineraries/5CxmV36Blk6n6qZxAQjrdQ/legs/2?exclude=stop,line,geometry,directions
```

##### Sample response

```
200 Ok
{
    "href": "https://platform.whereismytransport.com/api/journeys/PEP5VsjJ6kuo6KZxAQjq2Q/itineraries/5CxmV36Blk6n6qZxAQjrdQ/legs/2",
    "type": "Transit",
    "distance": {
        "value": 8554,
        "unit": "m"
    },
    "duration": 1200,
    "line": {
        "id": "yGZHGrc3sUOhNFLoer-Z_g",
        "href": "https://platform.whereismytransport.com/api/lines/yGZHGrc3sUOhNFLoer-Z_g"
    },
    "vehicle": {},
    "waypoints": [
        {
            "stop": {
                "id": "McWcQewKAUCZbluWHQk5kQ",
                "href": "https://platform.whereismytransport.com/api/stops/McWcQewKAUCZbluWHQk5kQ"
            },
            "arrivalTime": "2016-08-29T17:00:00Z",
            "departureTime": "2016-08-29T17:00:00Z"
        },
        {
            "stop": {
                "id": "C5UPegWudUa0h8LSSsVvrg",
                "href": "https://platform.whereismytransport.com/api/stops/C5UPegWudUa0h8LSSsVvrg"
            },
            "arrivalTime": "2016-08-29T17:20:00Z",
            "departureTime": "2016-08-29T17:20:00Z"
        }
    ]
}
```

#### Vehicle response model

Describes a single vehicle along a line so that it can be identified by passengers.

| Field | Type | Description |
| :--------- | :--- | :---- |
| designation | string | If available, an identifier for this vehicle as defined by the agency, or some other designation. |
| direction | string | If available, the direction of the vehicle, for example, _Northbound_ or _Clockwise_. |
| headsign | string | If available, identifying information (such as destination) displayed on the vehicle. |

#### Waypoint response model

A waypoint is a stopping point along an itinerary. It has either an arrival date and time or a departure date and time, or both.

| Field | Type | Description |
| :--------- | :--- | :---- |
| arrivalTime | [DateTime](#datetime) | The arrival date and time at this point of a leg. |
| departureTime | [DateTime](#datetime) | The departure date and time from this point of a leg. |
| stop | [Stop](#stop-response-model) | **[**[Excludable](#excludable)**]** The stop of the waypoint. This can be returned in either _Walking_ or _Transit_ legs. |
| location | [Location](#location-response-model) | The location of the waypoint if it is not a stop. This can be returned in only Walking legs. |

#### Direction response model

If available, the directions to follow in order to get from the start to the end of a leg.

| Field | Type | Description |
| :--------- | :--- | :---- |
| instruction | string | The instruction to follow. |
| distance | [Distance](#distance) | The distance to travel after the instruction has been followed. |

#### Location response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| address | string | The reverse geocoded address of the point, if available. |
| geometry | [GeoJSON](#geojson) Point | The geographic point of the location. |

### Fares

A fare is the cost incurred by a commuter when using a transport service.  Essentially, it is the price associated with a journey's itinerary for a particular fare product or set of fare products.

#### Fare response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| description | string | The description of the fare for this leg. |
| fareProduct | [FareProduct](#fare-product-response-model) | The fare product selected for this leg. |
| cost | [Cost](#cost) | The cost of this leg. |
| messages | Array of string | Any fare messages, such as required fare cards or special instructions. |

#### Specifying fare products

When creating a new journey, the default [fare product](#fare-products) will be used, if available. Specific fare products can be specified in order to get a more desired fare result.

##### Sample request

```
POST api/journeys
{
    "geometry": {
        "type": "MultiPoint",
        "coordinates": [
            [
                18.422326,
                -33.922843
            ],
            [
                18.473162,
                -33.966573
            ]
        ]
    },
    "time": "2016-08-30T16:00:00Z",
    "timeType": "DepartAfter",
    "fareProducts": [
        "pCawiJA73UmchaZtAKAHwg"
    ]
}
```

### Fare Products

A fare product is a fare scheme offered to passenger by an agency and will decide the total [fare](#fares) incurred when using a transport service. Note that they may be subject to eligibility restrictions. For example, a "Child Single" fare product might only be allowed to be used by children.

#### Fare Product response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the fare product. |
| href | string | The hyperlink pointing to the fare product. |
| agency | [Agency](#agency-model) | **[**[Excludable](#excludable)**]** The fare product's agency. |
| name | string | The commuter-friendly name of the fare product. |
| isDefault | bool | Flag specifying whether this is the default fare product for this agency. |
| description | string | A commuter-friendly description of the fare product. |

#### Retrieving fare products

Retrieves a collection of fare products.

`GET api/fareproducts?agencies={agencies}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| agencies | Array of [Identifier](#identifiers) | The list of agencies to filter the results by. |
| limit | int | The maximum number of entities to be returned. Default is 100. |
| offset | int | The offset of the first entity returned. Default is 0. |

##### Sample request

```
GET api/fareproducts?agencies=5kcfZkKW0ku4Uk-A6j8MFA&limit=2
```

##### Sample response

```
200 Ok
[
    {
        "id": "pCawiJA73UmchaZtAKAHwg",
        "href": "https://platform.whereismytransport.com/api/fareproducts/pCawiJA73UmchaZtAKAHwg",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
            "name": "MyCiTi",
            "culture": "en"
        },
        "name": "Monthly Pass",
        "isDefault": false,
        "description": "Unlimited travel on any day and at any time and are valid for one month from a start date of your choice excluding the Airport."
    },
    {
        "id": "1Mw8Zdr65E-eUqZtAKAH1Q",
        "href": "https://platform.whereismytransport.com/api/fareproducts/1Mw8Zdr65E-eUqZtAKAH1Q",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
            "name": "MyCiTi",
            "culture": "en"
        },
        "name": "Monthly Pass Airport",
        "isDefault": false,
        "description": "Unlimited travel on any day and at any time and are valid for one month from a start date of your choice."
    }
]
```
