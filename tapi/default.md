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

##### Sample request

```
GET api/agencies
```

#### Content type

The **Content-Type** header describes the format of the data being posted to the server. All POST requests must specify this header as **application/json**. If the **Content-Type** header is not specified or the type is set to a value other than **application/json** then a **415 Unsupported Media Type** [status code](#http-status-codes) will be returned.

##### Sample request

```
POST api/journeys
```

#### Compression

The API compresses response data using GZIP compression as defined by the HTTP 1.1 specification. Disabling compression can be done by setting the **Allow-Compression** request header to **false**. The response will always have the **Content-Encoding** response header set to **gzip** when the response body is compressed accordingly.

**Note:** No other methods of compression are supported. Request compression is not supported.

#### HTTPS

All API access is performed over HTTPS only. If a resource is requested using **http://** then a **403 Forbidden** [status code](#http-status-codes) will be returned.

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

A **401 Unauthorized** [status code](#http-status-codes) will be returned if the request is either missing the **Authorization** header or if the token has expired.

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

The important thing to note that is an entity could be deprecated in a future schedule. This means that any entity resource URI could return a **404 Not Found** [status code](#http-status-codes). Furthermore, new entities could be added at any point. Applications built on this API are highly encouraged to cater for this.

### Pagination 

Collection endpoints are paginated so to ensure that responses are easier to handle and that payload is kept to a manageable size.

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| limit | integer | The number of entities to be returned. The default and maximum is typically 100 unless otherwise specified. |
| offset | integer | The zero-based offset of the first entity returned. The default is always 0.  |

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

**Note:** ISO 8601 dates are timezone-agnostic and so are communicated in UTC (Coordinated Universal Time).

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

**Note: ** The ordering of these two coordinates is latitude first and then longitude.

#### BoundingBox 

In order to provide a geographic bounding box through the query string, a comma-separated SW (south west) latitude, SW longitude, NE (north east) latitude and NE longitude must be provided in that order.  These coordiantes represent the south west and north east corners of the bounding box.

##### Sample request

```
GET api/stops?bbox=-33.94,18.36,-33.89,18.43
```

#### Distance 

Distance is returned as an object consisting of the distance **value** (an integer) and the associated **unit** symbol.

##### Sample response

```
{  
    "distance": {  
        "value": 133,
        "unit": "m"
    }
}
```

**Note:** The API currently only supports the metric system and distance is always returned in metres.

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
| href | [hyperlink](#resource-linking) | The hyperlink to this resource. |
| name | string | The full name of the agency. |
| culture | string | The name of the [culture](#culture), based on RFC 4646. |

#### Retrieving agencies

Retrieves a collection of agencies.

`GET api/agencies?point={Point}&radius={int}&bbox={BoundingBox}&agencies={Identifiers}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| point | [Point](#point) | The point from where to search for nearby agencies. Agencies will be returned in order of their distance from this point (from closest to furthest). |
| radius | integer | The distance in metres from the point to search for nearby agencies. This filter is optional. |
| bbox | [BoundingBox](#boundingbox) | The bounding box from where to retrieve agencies. This will be ignored if a point is provided in the query.  |
| agencies | Array of [Identifier](#identifiers) | A string of comma-separated agency identifiers which to filter the results by. |
| exclude | string | A string of comma-separated object or collection names to [exclude](#excluding-data) from the response. |
| limit | integer | See [Pagination](#pagination). The default is 100. |
| offset | integer | See [Pagination](#pagination). The default is 0. |

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

`GET api/agencies/{id}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the agency. |

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
| href | [hyperlink](#resource-linking) | The hyperlink to this resource. |
| agency | [Agency](#agency-response-model) | **[**[Excludable](#excludable)**]** The agency. |
| name | string | The full name of the stop. |
| code | string | If available, the passenger code of the stop. |
| geometry | [GeoJSON](#geojson) Point | The geographic point of the stop. |
| modes | Array of [Mode](#modes) | The modes that are served by this stop. |
| parentStop | [Stop](#stop-response-model) | **[**[Excludable](#excludable)**]** If applicable, the parent stop. |

#### Retrieving stops

Retrieves a collection of stops.

`GET api/stops?point={Point}&radius={int}&bbox={BoundingBox}&modes={Modes}&agencies={Identifiers}&servesLines={Identifiers}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| point | [Point](#point) | The point from where to search for nearby stops. Stops will be returned in order of their distance from this point (from closest to furthest). |
| radius | integer | The distance in metres from the point to search for nearby stops. This filter is optional. |
| bbox | [BoundingBox](#boundingbox) | The bounding box from where to retrieve stops. This will be ignored if a point is provided in the query.  |
| modes | string | A string of comma-separated [transport modes](#modes) to filter the results by. |
| agencies | Array of [Identifier](#identifiers) | A string of comma-separated agency identifiers to filter the results by. |
| servesLines | Array of [Identifier](#identifiers) | A string of comma-separated line identifiers to filter the results by. |
| showChildren | bool | Specifies whether or not to also return children stops. Default is false. |
| exclude | string | A string of comma-separated object or collection names to [exclude](#excluding-data) from the response. |
| limit | integer | See [Pagination](#pagination). The default is 100. |
| offset | integer | See [Pagination](#pagination). The default is 0. |

##### Sample request

```
GET api/stops?agencies=5kcfZkKW0ku4Uk-A6j8MFA,xp_eNbqkYEaZP2YZkHwQqg&point=-33.923,18.421&radius=500
```

##### Sample response

This request will retrieve stops from either agency **5kcfZkKW0ku4Uk-A6j8MFA** or **xp_eNbqkYEaZP2YZkHwQqg** and which are within 500 meters of the point [-33.923, 18.421].

```
200 Ok
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
    },
    {
        "id": "QTtT2sMlGkKpGEBV7TlBpA",
        "href": "https://platform.whereismytransport.com/api/stops/QTtT2sMlGkKpGEBV7TlBpA",
        "agency": {
            "id": "xp_eNbqkYEaZP2YZkHwQqg",
            "href": "https://platform.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
            "name": "Metrorail Western Cape",
            "culture": "en"
        },
        "name": "Cape Town",
        "code": "1434904469",
        "geometry": {
            "type": "Point",
            "coordinates": [
                18.424846,
                -33.922993
            ]
        },
        "modes": [
            "Rail"
        ]
    }
]
```

#### Retrieving a specific stop

Retrieves a stop by its identifier.

`GET api/stops/{id}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the stop. |

##### Sample request

```
GET api/stops/eBTeYLPXOkWm5zyfjZVaZg?exclude=agency
```

##### Sample response

This request will retrieve the stop resource and exclude unneeded **agency** fields.

```
200 Ok
{
    "id": "eBTeYLPXOkWm5zyfjZVaZg",
    "href": "https://platform.whereismytransport.com/api/stops/eBTeYLPXOkWm5zyfjZVaZg",
    "agency": {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
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

`GET api/stops/{id}/stops`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the parent stop. |

##### Sample request

```
GET api/stops/E8qYuZ4nEUSLS13pskx1Qg/stops
```

##### Sample response

This request will retrieve all child stops of the stop with identifier **E8qYuZ4nEUSLS13pskx1Qg**.

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
| vehicle | [Vehicle](#vehicle-response-model) | If available, identifying information for the vehicle running at this time. |
| line | [Line](#line-response-model) | The line from which the vehicle is traveling. |

#### Retrieving a stop timetable

Retrieves a timetable for a stop, consisting of a list of occurrences of a vehicle calling at this stop in order of arrival time.

`GET api/stops/{id}/timetables?earliestArrivalTime={DateTime}&limit={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the stop. |
| earliestArrivalTime | [DateTime](#datetime) | The earliest arrival date and time to include in the timetable. |
| exclude | string | A string of comma-separated object or collection names to [exclude](#excluding-data) from the response. |
| limit | integer | The maximum number of entities to be returned. Default is 10. |

##### Sample request

```
GET api/stops/eBTeYLPXOkWm5zyfjZVaZg/timetables?limit=2
```

##### Sample response

This request will retrieve timetable information for stop with identifier **eBTeYLPXOkWm5zyfjZVaZg**, limiting it to two items.

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
| href | [hyperlink](#resource-linking) | The hyperlink to this resource. |
| agency | [Agency](#agency-response-model) | **[**[Excludable](#excludable)**]** The line's agency. |
| name | string | If available, the full name of the line, . Either **name** or **shortName** will exist. |
| shortName | string | If available, the short name of the line. Either **name** or **shortName** will exist. |
| description | string | If available, a description of the line. |
| mode | [Mode](#modes) | The transport mode of the line. |
| colour | string | The assigned colour of the line as an 8-character (ARGB) hexadecimal number. For example, #FFFF0000 (red with 100% opacity). |
| textColour | string | The colour of the text when drawn against the colour of the line so to provide sufficient contrast for legibility. Also an 8-character (ARGB) hexadecimal number. For example, #CCFFFFFF (white with 50% opacity). |

#### Retrieving lines

Retrieves a collection of lines.

`GET api/lines?agencies={Identifiers}&servesStops={Identifiers}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| agencies | Array of [Identifier](#identifiers) | A comma-separated list of agency identifiers to filter the results by. |
| servesStops | Array of [Identifier](#identifiers) | A comma-separated list of stop identifiers that represent stops which the returned lines must serve. |
| exclude | string | A string of comma-separated object or collection names to [exclude](#excluding-data) from the response. |
| limit | integer | See [Pagination](#pagination). The default is 100. |
| offset | integer | See [Pagination](#pagination). The default is 0. |

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
        "name": "A01 - Civic Centre to Airport",
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

`GET api/lines/{id}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the line. |

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
| vehicle | [Vehicle](#vehicle-response-model) | If available, identifying information for the vehicle running at this time. |
| waypoints | Array of [Waypoint](#waypoint-response-model) | The sequence of ordered way points that make up this line timetable. |

#### Retrieving a line timetable

Retrieves a timetable for a line, consisting of a list of departures on this line in order of departure time.

`GET api/lines/{id}/timetables?earliestDepartureTime={DateTime}&departureStopId={stop}&arrivalStopId={stop}&limit={int}`

| Parameter | Type | Notes |
| :-------------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the line. |
| earliestDepartureTime | [DateTime](#datetime) | Optional earliest departure time on that line to be included in the timetable. |
| departureStopId | [Identifier](#identifiers) | Optional stop identifier - bounds results to only occur after this stop. |
| arrivalStopId | [Identifier](#identifiers) | Optional stop identifier - bounds results to only occur before this stop. |
| exclude | string | A string of comma-separated object or collection names to [exclude](#excluding-data) from the response. |
| limit | integer | The maximum number of entities to be returned. Default is 10. |

##### Sample request

```
GET api/lines/rBD_j-ZRdEiiHMc9lNzQtA/timetables?departureStopId=fbJnFbrZ906L0_jC09_eJw&limit=2&exclude=stop
```

##### Sample response

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
| href | [hyperlink](#resource-linking) | The hyperlink to this resource. |
| geometry | [GeoJSON](#geojson) MultiPoint | An ordered GeoJSON MultiPoint representing the departure and arrival points for the the journey. |
| time | [DateTime](#datetime) | The requested date and time for the journey.  |
| timeType | [TimeType](#timetype) | Specifies whether this is an **ArriveBefore** or **DepartAfter** request. |
| profile | [Profile](#profile) | The profile used to calculate and order itineraries. |
| only | [Filter](#filter) | The explicit set of modes and agencies used. |
| omit | [Filter](#filter) | The explicit set of modes and agencies omitted. |
| fareProducts | Array of [Identifier](#identifiers) | An array of [fare product](#fare-products) identifiers applied when calculating the itinerarys' fare amounts. |
| maxItineraries | integer | The maximum number of itineraries to return. |
| itineraries | Array of [Itinerary](#itinerary-response-model) | **[**[Excludable](#excludable)**]** The available [itineraries](#itineraries) for this journey. |

#### Creating a journey

Creating a new journey is done by posting the journey's criteria to the resource.

`POST api/journeys`

| Field | Type | Required | Description |
| :--------- | :--- | :--- | :---- |
| geometry | [GeoJSON](#geojson) MultiPoint | Required | An ordered GeoJSON MultiPoint representing the departure and arrival points for the the journey. Exactly two points must be provided. |
| time | [DateTime](#datetime) | Optional | The requested date and time for the journey. Defaults to Now. |
| timeType | [TimeType](#timetype) | Optional | Specifies whether this is an ArriveBefore or DepartAfter request. Defaults to DepartAfter. |
| profile | [Profile](#profile) | Required | The profile used to calculate and order itineraries. |
| only | [Filter](#filter) | Optional | The explicit set of modes or agencies to use. If unset, all modes and agencies are used. |
| omit | [Filter](#filter) | Optional | The explicit set of modes or agencies to exclude. Omit will always take preference. |
| fareProducts | Array of [Identifier](#identifiers) | Optional | The list of [fare product](#fare-products) identifiers to use when calculating the journey fare. |
| maxItineraries | integer | Optional | The maximum number of itineraries to return. This must be a value between or including 1 and 5. Default is 5. |

#### TimeType

Time type can either be **DepartAfter** or **ArriveBefore**.

**DepartAfter** (the default) indicates that the journey must be calculated to depart after the specified time, at the earliest.

**ArriveBefore** indicates that the journey must be calculated to arrive before the specified time, at the latest.

#### Profile

The profile specifies how the itineraries should be prioritised.

**ClosestToTime** (the default) returns itineraries absolutely closest to the requested date; earliest for **DepartAfter**, and latest for **ArriveBefore**.

**FewestTransfers** returns itineraries with fewest connections between vehicles, and then also prioritising by closest to time.

#### Filter

A filter can be specified to explicitly use or omit certain agencies or modes from the requested journey.

| Field | Type | Description |
| :--------- | :--- | :--- |
| agencies | Array of [Identifier](#identifiers) | A list of agencies to use in or omit from the journey. |
| modes | Array of [Mode](#mode) | A list of modes to use or omit in the journey. |

##### Sample request

```
POST api/journeys?exclude=line,stop,fareProduct
{
    "geometry": {
        "type": "Multipoint",
        "coordinates": [
            [
                18.417,
                -33.916
            ],
            [
                18.412,
                -33.908
            ]
        ]
    },
    "time": "2016-08-30T10:30:00Z",
    "omit": {
        "modes": [
            "Rail", 
            "LightRail"
        ]
    },
    "maxItineraries": 1
}
```

##### Sample response

This request will exclude unneeded information on all contained stop, line and fare product resources in order to reduce the payload.

```
201 Created
{
    "id": "uvRvS486sUODiqZyAK-j2g",
    "href": "https://platform.whereismytransport.com/api/journeys/uvRvS486sUODiqZyAK-j2g",
    "geometry": {
        "type": "MultiPoint",
        "coordinates": [
            [
                18.417,
                -33.916
            ],
            [
                18.412,
                -33.908
            ]
        ]
    },
    "time": "2016-08-30T10:30:00Z",
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
            "Rail", 
            "LightRail"
        ]
    },
    "itineraries": [
        {
            "id": "fLdmWzq_h0-uoqZyAK-keQ",
            "href": "https://platform.whereismytransport.com/api/journeys/uvRvS486sUODiqZyAK-j2g/itineraries/fLdmWzq_h0-uoqZyAK-keQ",
            "departureTime": "2016-08-30T10:31:15Z",
            "arrivalTime": "2016-08-30T10:38:22Z",
            "distance": {
                "value": 1298,
                "unit": "m"
            },
            "duration": 427,
            "legs": [
                {
                    "href": "https://platform.whereismytransport.com/api/journeys/uvRvS486sUODiqZyAK-j2g/itineraries/fLdmWzq_h0-uoqZyAK-keQ/legs/0",
                    "type": "Walking",
                    "distance": {
                        "value": 228,
                        "unit": "m"
                    },
                    "duration": 164,
                    "waypoints": [
                        {
                            "location": {
                                "address": "Fiesta, Dixon Street, Cape Town Ward 77, Cape Town Subcouncil 16, Cape Town, City of Cape Town, Western Cape, 8001, South Africa",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.417,
                                        -33.916
                                    ]
                                }
                            },
                            "arrivalTime": "2016-08-30T10:31:15Z",
                            "departureTime": "2016-08-30T10:31:15Z"
                        },
                        {
                            "stop": {
                                "id": "lbR0mPfo-0e2eyhr-wlpJw",
                                "href": "https://platform.whereismytransport.com/api/stops/lbR0mPfo-0e2eyhr-wlpJw"
                            },
                            "arrivalTime": "2016-08-30T10:34:00Z",
                            "departureTime": "2016-08-30T10:34:00Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.416975,
                                -33.916008
                            ],
                            [
                                18.417094,
                                -33.916252
                            ],
                            [
                                18.417571,
                                -33.916111
                            ],
                            [
                                18.418382,
                                -33.915638
                            ],
                            [
                                18.41822,
                                -33.91529
                            ]
                        ]
                    },
                    "directions": [
                        {
                            "instruction": "Start on Waterkant Street",
                            "distance": {
                                "value": 33,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left onto Dixon Street",
                            "distance": {
                                "value": 135,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left onto Somerset Road, M61",
                            "distance": {
                                "value": 59,
                                "unit": "m"
                            }
                        }
                    ]
                },
                {
                    "href": "https://platform.whereismytransport.com/api/journeys/uvRvS486sUODiqZyAK-j2g/itineraries/fLdmWzq_h0-uoqZyAK-keQ/legs/1",
                    "type": "Transit",
                    "distance": {
                        "value": 872,
                        "unit": "m"
                    },
                    "duration": 120,
                    "fare": {
                        "description": "Default fare",
                        "fareProduct": {
                            "id": "LjPJY99Qn0iEy6ZtAKAHdA",
                            "href": "https://platform.whereismytransport.com/api/fareproducts/LjPJY99Qn0iEy6ZtAKAHdA"
                        },
                        "cost": {
                            "amount": 8.4,
                            "currencyCode": "ZAR"
                        },
                        "messages": []
                    },
                    "line": {
                        "id": "9fzL31HtwUCM3FzCVzMNAw",
                        "href": "https://platform.whereismytransport.com/api/lines/9fzL31HtwUCM3FzCVzMNAw"
                    },
                    "vehicle": {
                        "direction": "OneDirection"
                    },
                    "waypoints": [
                        {
                            "stop": {
                                "id": "lbR0mPfo-0e2eyhr-wlpJw",
                                "href": "https://platform.whereismytransport.com/api/stops/lbR0mPfo-0e2eyhr-wlpJw"
                            },
                            "arrivalTime": "2016-08-30T10:34:00Z",
                            "departureTime": "2016-08-30T10:34:00Z"
                        },
                        {
                            "stop": {
                                "id": "ai3VutsKkkitBUyMII4mBQ",
                                "href": "https://platform.whereismytransport.com/api/stops/ai3VutsKkkitBUyMII4mBQ"
                            },
                            "arrivalTime": "2016-08-30T10:35:00Z",
                            "departureTime": "2016-08-30T10:35:00Z"
                        },
                        {
                            "stop": {
                                "id": "HCvUv9zHh0adgamzIDmtoQ",
                                "href": "https://platform.whereismytransport.com/api/stops/HCvUv9zHh0adgamzIDmtoQ"
                            },
                            "arrivalTime": "2016-08-30T10:36:00Z",
                            "departureTime": "2016-08-30T10:36:00Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.41822,
                                -33.91529
                            ],
                            [
                                18.41818,
                                -33.9152
                            ],
                            [
                                18.41808,
                                -33.91501
                            ],
                            [
                                18.41802,
                                -33.9149
                            ],
                            [
                                18.41779,
                                -33.91444
                            ],
                            [
                                18.41747,
                                -33.91381
                            ],
                            [
                                18.41717,
                                -33.91325
                            ],
                            [
                                18.41676,
                                -33.91272
                            ],
                            [
                                18.41623,
                                -33.91208
                            ],
                            [
                                18.41613,
                                -33.91198
                            ],
                            [
                                18.41613,
                                -33.91198
                            ],
                            [
                                18.41568,
                                -33.9115
                            ],
                            [
                                18.41458,
                                -33.91051
                            ],
                            [
                                18.4131,
                                -33.90922
                            ]
                        ]
                    }
                },
                {
                    "href": "https://platform.whereismytransport.com/api/journeys/uvRvS486sUODiqZyAK-j2g/itineraries/fLdmWzq_h0-uoqZyAK-keQ/legs/2",
                    "type": "Walking",
                    "distance": {
                        "value": 198,
                        "unit": "m"
                    },
                    "duration": 142,
                    "waypoints": [
                        {
                            "stop": {
                                "id": "HCvUv9zHh0adgamzIDmtoQ",
                                "href": "https://platform.whereismytransport.com/api/stops/HCvUv9zHh0adgamzIDmtoQ"
                            },
                            "arrivalTime": "2016-08-30T10:36:00Z",
                            "departureTime": "2016-08-30T10:36:00Z"
                        },
                        {
                            "location": {
                                "address": "Stadium, Helen Suzman Boulevard, Green Point, Cape Town Subcouncil 16, Cape Town, City of Cape Town, Western Cape, 8005, South Africa",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.412,
                                        -33.908
                                    ]
                                }
                            },
                            "arrivalTime": "2016-08-30T10:38:22Z",
                            "departureTime": "2016-08-30T10:38:22Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.4131,
                                -33.90922
                            ],
                            [
                                18.413117,
                                -33.9092
                            ],
                            [
                                18.412888,
                                -33.909003
                            ],
                            [
                                18.413034,
                                -33.908861
                            ],
                            [
                                18.412872,
                                -33.908729
                            ],
                            [
                                18.412626,
                                -33.908619
                            ],
                            [
                                18.412706,
                                -33.908541
                            ],
                            [
                                18.411984,
                                -33.908015
                            ]
                        ]
                    },
                    "directions": [
                        {
                            "instruction": "Start on Main Road, M61",
                            "distance": {
                                "value": 30,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right",
                            "distance": {
                                "value": 78,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left onto Helen Suzman Boulevard, M6",
                            "distance": {
                                "value": 88,
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

#### Itinerary response model

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| id | [identifier](#identifiers) | The identifier of the itinerary. |
| href | [hyperlink](#resource-linking) | The hyperlink to this resource. |
| departureTime | [DateTime](#datetime) | The departure date and time for the itinerary. |
| arrivalTime | [DateTime](#datetime) | The arrival date and time for the itinerary. |
| distance | [Distance](#distance) | If available, the total distance of the itinerary. |
| duration | integer | If available, the total duration of the itinerary in seconds. |
| legs | Array of [Leg](#leg-response-model) | **[**[Excludable](#excludable)**]** The sequence of legs that make up this itinerary. |

#### Retrieving a specific itinerary

To retrieve a specific itinerary for a previously created journey, the following resource can be requested.

`GET api/journeys/{journeyId}/itineraries/{itineraryId}`

| Parameter | Type | Description |
| :-------------- | :--- | :--- | :---- |
| journeyId | [Identifier](#identifiers) | The identifier of the journey. |
| itineraryId | [Identifier](#identifiers) | The identifier of the itinerary. |

##### Sample request

```
GET api/journeys/8GYKddjcAk6j7aVUAMV3pw/itineraries/dnCQV5Kq0kaq5KVUAMV_eQ
```

### Legs

A leg is a section of an itinerary carried out by a passenger on one mode of transport (including walking) from some departure point to an arrival point.

#### Types of legs

The API currently supports two types of legs, each with a different response model.

A _Walking_ leg is one which the passenger is to travel by foot from one waypoint to another.  Walking legs are usually accompanied with directions.

A _Transit_ leg is one which uses a public transportation service based on scheduled or absolute frequency-based stop times.

#### Leg response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| type | string | The [type of leg](#types-of-legs), either _Walking_ or _Transit_. |
| distance | [Distance](#distance) | If available, the total distance of the leg. |
| duration | integer | If available, the total duration of the leg in seconds. |
| line | [Line](#line-response-model) | **[**[Excludable](#excludable)**]** The line that is used on this leg of the itinerary. This is only returned for _Transit_ legs. |
| vehicle | [Vehicle](#vehicle-response-model) | **[**[Excludable](#excludable)**]** Identifying information for the vehicle that is used on this leg of the itinerary. This is only returned for _Transit_ legs. |
| fare | [Fare](#fare-response-model) | If available, the fare for this leg. |
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

#### Waypoint response model

A waypoint is a stopping point along an itinerary. It has either an arrival date and time or a departure date and time, or both.

| Field | Type | Description |
| :--------- | :--- | :---- |
| arrivalTime | [DateTime](#datetime) | The arrival date and time at this point of a leg. |
| departureTime | [DateTime](#datetime) | The departure date and time from this point of a leg. |
| stop | [Stop](#stop-response-model) | **[**[Excludable](#excludable)**]** The stop of the waypoint. This can be returned in either _Walking_ or _Transit_ legs. |
| location | [Location](#location-response-model) | The location of the waypoint if it is not a stop. This can be returned only in  Walking legs. |

#### Vehicle response model

Describes a single vehicle along a line so that it can be identified by passengers.

| Field | Type | Description |
| :--------- | :--- | :---- |
| designation | string | If available, an identifier for this vehicle as defined by the agency, or some other designation. |
| direction | string | If available, the direction of the vehicle, for example, _Northbound_ or _Clockwise_. |
| headsign | string | If available, identifying information (such as the destination) displayed on the vehicle. |

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
| fareProduct | [FareProduct](#fare-product-response-model) | **[**[Excludable](#excludable)**]** The fare product selected for this leg. |
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
                18.422,
                -33.922
            ],
            [
                18.473,
                -33.966
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

A fare product is a fare scheme offered to passengers by an agency and will decide the total [fare](#fares) incurred when using a transport service. Note that they may be subject to eligibility restrictions. For example, a "Child Single" fare product might only be allowed to be used by children.

#### Fare Product response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the fare product. |
| href | [hyperlink](#resource-linking) | The hyperlink to this resource. |
| agency | [Agency](#agency-response-model) | **[**[Excludable](#excludable)**]** The fare product's agency. |
| name | string | The commuter-friendly name of the fare product. |
| isDefault | bool | Flag specifying whether this is the default fare product for this agency. |
| description | string | A commuter-friendly description of the fare product. |

#### Retrieving fare products

Retrieves a collection of fare products.

`GET api/fareproducts?agencies={Identifiers}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| agencies | Array of [Identifier](#identifiers) | The list of agencies to filter the results by. |
| exclude | string | A string of comma-separated object or collection names to [exclude](#excluding-data) from the response. |
| limit | integer | See [Pagination](#pagination). The default is 100. |
| offset | integer | See [Pagination](#pagination). The default is 0. |

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
