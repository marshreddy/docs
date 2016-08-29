# Our API Specification

## Overview

Central to the WhereIsMyTransport platform is our transport API. It is based on [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer), [JSON](http://www.json.org/), [OAuth 2.0](http://oauth.net/2/) and [OpenID Connect](http://openid.net/connect/). These are standards which are broadly supported in the industry.  Note that [JSON](http://www.json.org/) is the only supported data format.

### Introduction

Read the following sections to get you started using the API.

#### API Endpoint

The following address is the standard URL endpoint to be used to access the various resources of the API.

`https://platform.whereismytransport.com/api`

For example, the agencies endpoint would be queried at **GET https://platform.whereismytransport.com/api/agencies**.

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

The **Accept** header describes the format of the content that the client can accept. All requests must specify this header as **application/json**. If the **Accept** header is not specified or the type is set to a value other than **application/json** then a **406** [HTTP status code](#http-status-codes) will be returned.

### Sample request

```
GET api/agencies
Accept: application/json
```

#### Content type

The **Content-Type** header describes the format of the data being posted to the server. All POST requests must specify this header as **application/json**. If the **Content-Type** header is not specified or the type is set to a value other than **application/json** then a **415** [HTTP status code](#status-codes) will be returned.

### Sample request

```
POST api/journeys
```

#### Compression

The API compresses response data using GZIP compression as defined by the HTTP 1.1 specification. Disabling compression can be done by setting the **Allow-Compression** request header to **false**. The response will always have the **Content-Encoding** header set to **gzip** when the response body is compressed accordingly.

**Note:** No other methods of compression are supported. Request compression is not supported.

#### HTTPS

All API access is performed over HTTPS only. If a resource is requested using **http://** then a **403 Forbidden** status code will be returned.

### Authorisation

The API uses OpenID Connect and OAuth 2.0 protocols for federated access and security. WhereIsMyTransport provides its own security token service which issues tokens to applications so that they can authenticate themselves against our transport API. To authorise an application, one must first acquire a **client_id** and **client_secret** from the WhereIsMyTransport [Developer Portal](https://developer.whereismytransport.com).

Using client credentials one can make requests against the security token service to retrieve a token.  When requesting a token, the scopes that are required must also be specified. Currently, the only scope available is `transportapi:all` which provides full access to the API.

**Note:** The content type of **application/x-www-form-urlencoded** must be used for this request.

#### Token endpoint

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

Once this token is received, it must be added to the **Authorization** HTTP header in order to perform successful requests against the API.

##### Sample request

```
GET api/agencies
Authorization: Bearer eyJ0eXAiOiJ32aQiLCJhbGciOiJSUzI1NiIsIfg1iCI6ImEzck1VZ01Gd8d0UGNsTGE2eUYz...
```

### Errors

The API uses conventional [HTTP status codes](#status-codes) to indicate the result of a request. Codes within the 200s indicate that the request was successful.  Codes within the 400s indicate that the request was somehow badly formed (such as a missing or incorrectly formatted field). 500s are typically returned when something unexpected goes wrong on the server.

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

Almost all entities in the Transit API are identified through the use of a globally unique identifier. This identifier is specified as a 22 character long, case-sensitive string of URL-friendly characters; __a__ to __z__, __A__ to __Z__, __0__ to __9__, - (hyphen) and _ (underscore). See the **id** field in the sample response below.

#### Sample request

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

The API presents a simple and elegant approach to referencing of and linking to resources. Every available resource has a hypertext reference field, **href**. This value represents a fully qualified URL to where the resource resides. Presenting such a field for every resource aids discoverability by allowing for new resources to be consumed by just embedding a new reference link.

For example, the sample below demonstrates a stop with its agency sub-resource, both having **id** and **href** fields.

##### Sample request

```
GET api/stops/eBTeYLPXOkWm5zyfjZVaZg
```

##### Sample response

```http
200 Ok
{
    "id": "eBTeYLPXOkWm5zyfjZVaZg",
    "href": "https://transit.whereismytransport.com/api/stops/eBTeYLPXOkWm5zyfjZVaZg",
    "agency": {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://transit.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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

In order to reduce payload, it is possible to exclude certain objects or collections from the model returned in the body of the HTTP response. This is done through the use of the **exclude** query. Although not everything can be exluded. Fields which are _excludable_ are described in this specification with the [Excludable](#excludable) tag.

| Parameter | Type | Required | Description |
| :-------------- | :--- | :---- | :---- |
| exclude | string | Optional | Comma-separated object or collection names to be excluded from the response. |

When excluding resource objects, the containing object with their **id** and **href** fields will remain. This is to preserve discoverability while still reducing payload. Collections, on the other hand, will be excluded entirely.

##### Sample request

The request below will exclude **geometry** and **directions** from the resource model.

```
GET api/journeys/8GYKddjcAk6j7aVUAMV3pw?exclude=geometry,directions
```

### Understanding Scheduled Data

Everything retrievable from the API is considered scheduled data. This means that all queries are "at" a certain scheduled date and time. An agency, for example, may schedule a line's name to change, not now, but only after a certain date.  A new stop could be scheduled to only be returned from the API at a given date. The important thing to note that is an entity could be deprecated in future schedules. This means that any entity resource URI could return a 404. Applications built on this API are highly encouraged to cater for this.

### Pagination 

Depending on the structure of a query, a lot of results could be returned from the API. For that reason, the results are paginated so to ensure that responses are easier to handle and that payload it kept to a manageable size.

| Parameter | Type | Required | Description |
| :-------------- | :--- | :---- | :---- |
| limit | int | Optional | The number of entities to be returned. The default and maximum is typically 100. |
| offset | int | Optional | The zero-based offset of the first entity returned. The default is 0.  |

##### Sample request

The request below will retrieve 10 stops from the 50th stop onwards.

```http
GET api/stops?limit=10&offset=50
```

### Formatting Standards 

#### DateTime 

A typical format for encoding of date and time in JSON is to use the ISO 8601 standard. This is a well-established specification which is both human readable and widely supported by many web-based frameworks.

ISO 8601 date and time strings can be represented as "2016-11-19T07:22Z" (7:22 AM on the 19th November 2016).

More information can be found [here](https://en.wikipedia.org/wiki/ISO_8601).

**Note:** ISO 8601 dates are timzone-agnostic and so are communicated in UTC (Coordinated Universal Time).

#### Culture 

The appropriate culture format used in the Transit API is based on RFC 4646. This unique name is a combination of an ISO 639 two-letter lowercase culture code associated with a language and an ISO 3166 two-letter uppercase subculture code associated with a country or region.

For example, _en-US_ refers to English (United States) and _en-ZA_ to English (South Africa).

The detailed specification can be found [here](https://www.ietf.org/rfc/rfc4646.txt).

#### Cost 

Monetary amounts are represented by the cost object, which is made up of an amount, as a decimal value, and the applicable currency code. The currency code is such as defined in ISO 4217. For example, **ZAR** represents the South African Rand. More information and a full list of currency codes can be found [here](https://en.wikipedia.org/wiki/ISO_4217).

##### Sample

The following cost object represents the value of R10,50.

```http
{
    "cost": {
        "amount": 10.5,
        "currencyCode": "ZAR"
    }
}
```

#### GeoJSON 

[GeoJSON](http://geojson.org) is a JSON format for encoding geographic data structures. The Transit API uses the Point, MultiPoint and LineString geometry types.

A typical GeoJSON structure consists of a **type** field and an array of **coordinates**. 

**Note:**  GeoJSON represents geographic coordinates with longitude first and then latitude, `[longitude, latitude]`. i.e. `[x, y]` in the Cartesian coordinate system.

##### Sample

The following GeoJSON Point represents the coordinates for Cape Town's city centre.

```http
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

```http
GET api/stops?point=-33.925430,18.436443&radius=1750
```

**Note: ** The order is latitude then longitude.

#### Bounding Box 

In order to provide a geographic bounding box through the query string, a comma-separated SW (south west) latitude, SW longitude, NE (north east) latitude and NE longitude must be provided in that order.  These coordiantes represent the south west and north east corners of the box.

```http
GET api/stops?bbox=-33.944,18.36,-33.895,18.43
```

#### Distance 

Distance is returned as an object consisting of the distance value (an integer) and the associated unit symbol.

##### Sample response

```http
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

The mode of transit describes the type of vehicle that is used along a line. The following table describes the modes currently supported by the API.

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

A transit agency, or operator, is an organisation which provides and governs a transport service which is available for use by either the general public (in most cases) or through some private arrangement.

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

```http
GET api/agencies?bbox=-33.94,18.36,-33.89,18.43
```

##### Sample response

```http
200 Ok
[
    {
        "id": "xp_eNbqkYEaZP2YZkHwQqg",
        "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
        "name": "Metrorail Western Cape",
        "culture": "en"
    },
    {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://transit.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
        "name": "MyCiTi",
        "culture": "en"
    },
    {
        "id": "PO83DTm4oEuJ19prwicxHw",
        "href": "https://transit.whereismytransport.com/api/agencies/PO83DTm4oEuJ19prwicxHw",
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

```http
GET api/agencies/5kcfZkKW0ku4Uk-A6j8MFA
```

##### Sample response

```http
200 Ok
{
    "id": "5kcfZkKW0ku4Uk-A6j8MFA",
    "href": "https://transit.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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
| modes | string | A string of comma-separated [transit modes](#modes). |
| agencies | Array of [Identifier](#identifiers) | A string of comma-separated agency identifiers to filter the results by. |
| servesLines | Array of [Identifier](#identifiers) | A string of comma-separated line identifiers to filter the results by. |
| showChildren | bool | Specifies whether or not to display children stops that satisfy the query parameters. Default is false. |
| limit | int | See [Pagination](#pagination). |
| offset | int | See [Pagination](#pagination). |

##### Sample request

```http
GET api/stops?agencies=5kcfZkKW0ku4Uk-A6j8MFA&servicesLines=kYdaW1_dKUe7WZegmV1bFw&point=-33.923400,18.421586&radius=200
```

##### Sample response

```http
200 Ok
Content-Type: application/json
[
    {
        "id": "k42lx2toK0yJiVGvsM4WSA",
        "href": "https://transit.whereismytransport.com/api/stops/k42lx2toK0yJiVGvsM4WSA",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://transit.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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
        "href": "https://transit.whereismytransport.com/api/stops/-rMw7Fg3GEqwJIsiDa4xMA",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://transit.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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

```http
GET api/stops/eBTeYLPXOkWm5zyfjZVaZg
```

##### Sample response

```
200 Ok
{
    "id": "eBTeYLPXOkWm5zyfjZVaZg",
    "href": "https://transit.whereismytransport.com/api/stops/eBTeYLPXOkWm5zyfjZVaZg",
    "agency": {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://transit.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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
        "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/qiCODz6ky0qZ9agqLgF44Q",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
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
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/E8qYuZ4nEUSLS13pskx1Qg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
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
        "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/N_hUhxwX-EaNn5SRMk2VJg",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
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
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/E8qYuZ4nEUSLS13pskx1Qg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA"
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
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/lines/vBk_jw2saU-gfZCgo_JvLg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/lines/vBk_jw2saU-gfZCgo_JvLg",
            "agency": {
                "id": "5kcfZkKW0ku4Uk-A6j8MFA",
                "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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

A grouping together of routes marketed to passengers as a single section of the transit network.

#### Line response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | [Identifier](#identifiers) | The identifier of the line. |
| href | string | The hyperlink pointing to the line. |
| agency | [Agency](#agency-model) | **[**[Excludable](#excludable)**]** The line's agency. |
| name | string | If available, the full name of the line, . Either name or shortName will exist. |
| shortName | string | If available, the short name of the line. Either name or shortName will exist. |
| description | string | If available, a description of the line. |
| mode | [Mode](#modes) | The transit mode of the line. |
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

```http
GET api/lines?agencies=5kcfZkKW0ku4Uk-A6j8MFA&limit=2
```

##### Sample response

```http
200 Ok
[
    {
        "id": "-29sdEyVLkel5ouTHN5Bsg",
        "href": "https://transitapi-dev-webapp.azurewebsites.net/api/lines/-29sdEyVLkel5ouTHN5Bsg",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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
        "href": "https://transitapi-dev-webapp.azurewebsites.net/api/lines/kYdaW1_dKUe7WZegmV1bFw",
        "agency": {
            "id": "5kcfZkKW0ku4Uk-A6j8MFA",
            "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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

```http
GET api/lines/rBD_j-ZRdEiiHMc9lNzQtA
```

##### Sample response

```http
200 Ok
{
    "id": "rBD_j-ZRdEiiHMc9lNzQtA",
    "href": "https://transitapi-dev-webapp.azurewebsites.net/api/lines/rBD_j-ZRdEiiHMc9lNzQtA",
    "agency": {
        "id": "5kcfZkKW0ku4Uk-A6j8MFA",
        "href": "https://transitapi-dev-webapp.azurewebsites.net/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
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
                    "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/fbJnFbrZ906L0_jC09_eJw"
                },
                "arrivalTime": "2016-08-29T16:17:00Z",
                "departureTime": "2016-08-29T16:17:00Z"
            },
            {
                "stop": {
                    "id": "eBTeYLPXOkWm5zyfjZVaZg",
                    "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/eBTeYLPXOkWm5zyfjZVaZg"
                },
                "arrivalTime": "2016-08-29T16:22:00Z",
                "departureTime": "2016-08-29T16:22:00Z"
            },
            {
                "stop": {
                    "id": "84AqtsFm-0yAwX67o9-2vg",
                    "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/84AqtsFm-0yAwX67o9-2vg"
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
                    "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/fbJnFbrZ906L0_jC09_eJw"
                },
                "arrivalTime": "2016-08-30T16:17:00Z",
                "departureTime": "2016-08-30T16:17:00Z"
            },
            {
                "stop": {
                    "id": "eBTeYLPXOkWm5zyfjZVaZg",
                    "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/eBTeYLPXOkWm5zyfjZVaZg"
                },
                "arrivalTime": "2016-08-30T16:22:00Z",
                "departureTime": "2016-08-30T16:22:00Z"
            },
            {
                "stop": {
                    "id": "84AqtsFm-0yAwX67o9-2vg",
                    "href": "https://transitapi-dev-webapp.azurewebsites.net/api/stops/84AqtsFm-0yAwX67o9-2vg"
                },
                "arrivalTime": "2016-08-30T16:26:00Z",
                "departureTime": "2016-08-30T16:26:00Z"
            }
        ]
    }
]
```

### Journeys

A journey is the traveling of a passenger from a departure point to an arrival point.  A journey can consist of zero to many possible itineraries, each a travel option in getting from A to B. An itinerary consists of one to many legs, describing the path and mode of transit, to take in order to complete the journey.

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
| profile | Array of [Profile](#profile) | Required | The profile used to calculate and order itineraries. |
| fareProducts | Array of [Identifier](#identifiers) | Optional | The list of [fare products](#fare-products) identifiers to try to use when calculating the journey fare. |
| maxItineraries | integer | Optional | The maximum number of itineraries to return. This must be a value between or including 1 and 5. Default is 3. |
| only | [Filter](#filter) | Optional | The explicit set of modes or agencies to use. If unset, all modes and agencies are used. |
| omit | [Filter](#filter) | Optional | The explicit set of modes or agencies to exclude. Omit will always take preference. |

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
    }
}
```

##### Sample response

```json
201 Created
Content-Type: application/json
{
    "id": "FvIwnLmLy0uauqZxAQXaFA",
    "href": "https://transit.whereismytransport.com/api/journeys/FvIwnLmLy0uauqZxAQXaFA",
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
    "time": "2016-08-29T15:53:22Z",
    "timeType": "DepartAfter",
    "profile": "ClosestToTime",
    "fareProducts": [],
    "maxItineraries": 3,
    "only": {
        "agencies": [],
        "modes": []
    },
    "omit": {
        "agencies": [],
        "modes": []
    },
    "itineraries": [
        {
            "id": "UZyXS7BFlUyXBaZxAQXa7A",
            "href": "https://transit.whereismytransport.com/api/journeys/FvIwnLmLy0uauqZxAQXaFA/itineraries/UZyXS7BFlUyXBaZxAQXa7A",
            "departureTime": "2016-08-29T15:55:08Z",
            "arrivalTime": "2016-08-29T16:22:00Z",
            "distance": {
                "value": 8942,
                "unit": "m"
            },
            "duration": 1612,
            "legs": [
                {
                    "href": "https://transit.whereismytransport.com/api/journeys/FvIwnLmLy0uauqZxAQXaFA/itineraries/UZyXS7BFlUyXBaZxAQXa7A/legs/0",
                    "type": "Walking",
                    "distance": {
                        "value": 321,
                        "unit": "m"
                    },
                    "duration": 231,
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
                            "arrivalTime": "2016-08-29T15:55:08Z",
                            "departureTime": "2016-08-29T15:55:08Z"
                        },
                        {
                            "stop": {
                                "id": "QTtT2sMlGkKpGEBV7TlBpA",
                                "href": "https://transit.whereismytransport.com/api/stops/QTtT2sMlGkKpGEBV7TlBpA",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
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
                            },
                            "arrivalTime": "2016-08-29T15:59:00Z",
                            "departureTime": "2016-08-29T15:59:00Z"
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
                                18.422533,
                                -33.922561
                            ],
                            [
                                18.423072,
                                -33.922212
                            ],
                            [
                                18.423177,
                                -33.922146
                            ],
                            [
                                18.423266,
                                -33.922225
                            ],
                            [
                                18.424242,
                                -33.922944
                            ],
                            [
                                18.424456,
                                -33.922744
                            ],
                            [
                                18.424778,
                                -33.922977
                            ],
                            [
                                18.424826,
                                -33.923012
                            ],
                            [
                                18.42522,
                                -33.92288
                            ]
                        ]
                    },
                    "directions": [
                        {
                            "instruction": "Continue onto Adderley Street",
                            "distance": {
                                "value": 111,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right onto Strand Street, R102",
                            "distance": {
                                "value": 132,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn left",
                            "distance": {
                                "value": 76,
                                "unit": "m"
                            }
                        }
                    ]
                },
                {
                    "href": "https://transit.whereismytransport.com/api/journeys/FvIwnLmLy0uauqZxAQXaFA/itineraries/UZyXS7BFlUyXBaZxAQXa7A/legs/1",
                    "type": "Transit",
                    "distance": {
                        "value": 7955,
                        "unit": "m"
                    },
                    "duration": 900,
                    "fare": {
                        "description": "Default fare",
                        "fareProduct": {
                            "id": "j4aUOTtRuUC8KANPs9VRkA",
                            "href": "https://transit.whereismytransport.com/api/fareproducts/j4aUOTtRuUC8KANPs9VRkA",
                            "agency": {
                                "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                "name": "Metrorail Western Cape",
                                "culture": "en"
                            },
                            "name": "Metro Plus",
                            "isDefault": true,
                            "description": "Standard First Class fare with no discount."
                        },
                        "cost": {
                            "amount": 10.5,
                            "currencyCode": "ZAR"
                        },
                        "messages": []
                    },
                    "line": {
                        "id": "bxjXkT56-kaJsy3RURC1fg",
                        "href": "https://transit.whereismytransport.com/api/lines/bxjXkT56-kaJsy3RURC1fg",
                        "agency": {
                            "id": "xp_eNbqkYEaZP2YZkHwQqg",
                            "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                            "name": "Metrorail Western Cape",
                            "culture": "en"
                        },
                        "name": "Southern Line to Retreat",
                        "shortName": "Southern Line",
                        "mode": "Rail",
                        "colour": "#ffed1c24",
                        "textColour": "#ffffffff"
                    },
                    "vehicle": {
                        "designation": "0219",
                        "direction": "OneDirection"
                    },
                    "waypoints": [
                        {
                            "stop": {
                                "id": "QTtT2sMlGkKpGEBV7TlBpA",
                                "href": "https://transit.whereismytransport.com/api/stops/QTtT2sMlGkKpGEBV7TlBpA",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
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
                            },
                            "arrivalTime": "2016-08-29T15:59:00Z",
                            "departureTime": "2016-08-29T15:59:00Z"
                        },
                        {
                            "stop": {
                                "id": "Zuoq-0ebDkqCXXF57rYEeA",
                                "href": "https://transit.whereismytransport.com/api/stops/Zuoq-0ebDkqCXXF57rYEeA",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                    "name": "Metrorail Western Cape",
                                    "culture": "en"
                                },
                                "name": "Woodstock",
                                "code": "1588952223",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.446022,
                                        -33.925818
                                    ]
                                },
                                "modes": [
                                    "Rail"
                                ]
                            },
                            "arrivalTime": "2016-08-29T16:02:00Z",
                            "departureTime": "2016-08-29T16:02:00Z"
                        },
                        {
                            "stop": {
                                "id": "yuw4438WcEeB_5ofin89pA",
                                "href": "https://transit.whereismytransport.com/api/stops/yuw4438WcEeB_5ofin89pA",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                    "name": "Metrorail Western Cape",
                                    "culture": "en"
                                },
                                "name": "Salt River",
                                "code": "1864220716",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.465278,
                                        -33.927222
                                    ]
                                },
                                "modes": [
                                    "Rail"
                                ]
                            },
                            "arrivalTime": "2016-08-29T16:05:00Z",
                            "departureTime": "2016-08-29T16:05:00Z"
                        },
                        {
                            "stop": {
                                "id": "8DJV2nsQf0i63A2w2r8oHQ",
                                "href": "https://transit.whereismytransport.com/api/stops/8DJV2nsQf0i63A2w2r8oHQ",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                    "name": "Metrorail Western Cape",
                                    "culture": "en"
                                },
                                "name": "Observatory",
                                "code": "1495645232",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.471526,
                                        -33.938067
                                    ]
                                },
                                "modes": [
                                    "Rail"
                                ]
                            },
                            "arrivalTime": "2016-08-29T16:08:00Z",
                            "departureTime": "2016-08-29T16:08:00Z"
                        },
                        {
                            "stop": {
                                "id": "dLehMUV95EOIji5vJSND1A",
                                "href": "https://transit.whereismytransport.com/api/stops/dLehMUV95EOIji5vJSND1A",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                    "name": "Metrorail Western Cape",
                                    "culture": "en"
                                },
                                "name": "Mowbray",
                                "code": "1917047691",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.473603,
                                        -33.946733
                                    ]
                                },
                                "modes": [
                                    "Rail"
                                ]
                            },
                            "arrivalTime": "2016-08-29T16:10:00Z",
                            "departureTime": "2016-08-29T16:10:00Z"
                        },
                        {
                            "stop": {
                                "id": "FHAmVfZFW0O40TawMeo4lQ",
                                "href": "https://transit.whereismytransport.com/api/stops/FHAmVfZFW0O40TawMeo4lQ",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                    "name": "Metrorail Western Cape",
                                    "culture": "en"
                                },
                                "name": "Rosebank",
                                "code": "860637881",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.472998,
                                        -33.95457
                                    ]
                                },
                                "modes": [
                                    "Rail"
                                ]
                            },
                            "arrivalTime": "2016-08-29T16:12:00Z",
                            "departureTime": "2016-08-29T16:12:00Z"
                        },
                        {
                            "stop": {
                                "id": "GAW_yYTHwkSTn_6RjZ5v9g",
                                "href": "https://transit.whereismytransport.com/api/stops/GAW_yYTHwkSTn_6RjZ5v9g",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                    "name": "Metrorail Western Cape",
                                    "culture": "en"
                                },
                                "name": "Rondebosch",
                                "code": "1964415770",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.472641,
                                        -33.962226
                                    ]
                                },
                                "modes": [
                                    "Rail"
                                ]
                            },
                            "arrivalTime": "2016-08-29T16:14:00Z",
                            "departureTime": "2016-08-29T16:14:00Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.42522,
                                -33.92288
                            ],
                            [
                                18.42677,
                                -33.92394
                            ],
                            [
                                18.42715,
                                -33.92416
                            ],
                            [
                                18.4274,
                                -33.92428
                            ],
                            [
                                18.428,
                                -33.92454
                            ],
                            [
                                18.42901,
                                -33.92485
                            ],
                            [
                                18.43039,
                                -33.92532
                            ],
                            [
                                18.43052,
                                -33.92537
                            ],
                            [
                                18.43084,
                                -33.92548
                            ],
                            [
                                18.43145,
                                -33.92566
                            ],
                            [
                                18.43175,
                                -33.92573
                            ],
                            [
                                18.43211,
                                -33.92581
                            ],
                            [
                                18.43249,
                                -33.92586
                            ],
                            [
                                18.43286,
                                -33.9259
                            ],
                            [
                                18.43317,
                                -33.92593
                            ],
                            [
                                18.4333,
                                -33.92594
                            ],
                            [
                                18.43369,
                                -33.92594
                            ],
                            [
                                18.43534,
                                -33.92601
                            ],
                            [
                                18.43682,
                                -33.92611
                            ],
                            [
                                18.43707,
                                -33.92612
                            ],
                            [
                                18.43797,
                                -33.92616
                            ],
                            [
                                18.44029,
                                -33.92619
                            ],
                            [
                                18.44098,
                                -33.92617
                            ],
                            [
                                18.4412,
                                -33.92616
                            ],
                            [
                                18.44141,
                                -33.92614
                            ],
                            [
                                18.44171,
                                -33.92608
                            ],
                            [
                                18.44185,
                                -33.92607
                            ],
                            [
                                18.44216,
                                -33.92601
                            ],
                            [
                                18.44229,
                                -33.92599
                            ],
                            [
                                18.44269,
                                -33.92589
                            ],
                            [
                                18.44277,
                                -33.92586
                            ],
                            [
                                18.44316,
                                -33.92577
                            ],
                            [
                                18.44387,
                                -33.9256
                            ],
                            [
                                18.44406,
                                -33.92556
                            ],
                            [
                                18.44427,
                                -33.92553
                            ],
                            [
                                18.44449,
                                -33.92551
                            ],
                            [
                                18.44501,
                                -33.92549
                            ],
                            [
                                18.44601865359733,
                                -33.92548001333072
                            ],
                            [
                                18.44601865359733,
                                -33.92548001333072
                            ],
                            [
                                18.44602,
                                -33.92548
                            ],
                            [
                                18.44698,
                                -33.92547
                            ],
                            [
                                18.44757,
                                -33.92546
                            ],
                            [
                                18.44791,
                                -33.92546
                            ],
                            [
                                18.44831,
                                -33.92545
                            ],
                            [
                                18.44868,
                                -33.92543
                            ],
                            [
                                18.44876,
                                -33.92544
                            ],
                            [
                                18.4499,
                                -33.92556
                            ],
                            [
                                18.45254,
                                -33.92588
                            ],
                            [
                                18.45311,
                                -33.92594
                            ],
                            [
                                18.45431,
                                -33.92609
                            ],
                            [
                                18.45893,
                                -33.92667
                            ],
                            [
                                18.4611,
                                -33.92692
                            ],
                            [
                                18.46232,
                                -33.92708
                            ],
                            [
                                18.46303,
                                -33.92716
                            ],
                            [
                                18.46333,
                                -33.9272
                            ],
                            [
                                18.46382,
                                -33.92725
                            ],
                            [
                                18.46508,
                                -33.92747
                            ],
                            [
                                18.465222692307687,
                                -33.92749853846154
                            ],
                            [
                                18.465222692307687,
                                -33.92749853846154
                            ],
                            [
                                18.46523,
                                -33.9275
                            ],
                            [
                                18.46561,
                                -33.92758
                            ],
                            [
                                18.46626,
                                -33.92773
                            ],
                            [
                                18.46665,
                                -33.92787
                            ],
                            [
                                18.46698,
                                -33.92803
                            ],
                            [
                                18.46733,
                                -33.92822
                            ],
                            [
                                18.46784,
                                -33.92855
                            ],
                            [
                                18.46811,
                                -33.92877
                            ],
                            [
                                18.46837,
                                -33.92903
                            ],
                            [
                                18.46865,
                                -33.92935
                            ],
                            [
                                18.46887,
                                -33.92968
                            ],
                            [
                                18.46906,
                                -33.93002
                            ],
                            [
                                18.46925,
                                -33.93058
                            ],
                            [
                                18.46928,
                                -33.9307
                            ],
                            [
                                18.46954,
                                -33.93141
                            ],
                            [
                                18.46982,
                                -33.93231
                            ],
                            [
                                18.47,
                                -33.93291
                            ],
                            [
                                18.47024,
                                -33.93365
                            ],
                            [
                                18.47063,
                                -33.93494
                            ],
                            [
                                18.47096,
                                -33.93603
                            ],
                            [
                                18.47113,
                                -33.93663
                            ],
                            [
                                18.47126,
                                -33.93713
                            ],
                            [
                                18.47136,
                                -33.93748
                            ],
                            [
                                18.471528895225465,
                                -33.93806616578249
                            ],
                            [
                                18.471528895225465,
                                -33.93806616578249
                            ],
                            [
                                18.47153,
                                -33.93807
                            ],
                            [
                                18.4716,
                                -33.93835
                            ],
                            [
                                18.4725,
                                -33.94122
                            ],
                            [
                                18.47276,
                                -33.94208
                            ],
                            [
                                18.47303,
                                -33.94296
                            ],
                            [
                                18.47336,
                                -33.94407
                            ],
                            [
                                18.47343,
                                -33.94431
                            ],
                            [
                                18.47347,
                                -33.94444
                            ],
                            [
                                18.47357,
                                -33.94479
                            ],
                            [
                                18.47364,
                                -33.94503
                            ],
                            [
                                18.4737,
                                -33.94528
                            ],
                            [
                                18.47383,
                                -33.94589
                            ],
                            [
                                18.47389,
                                -33.94626
                            ],
                            [
                                18.47391,
                                -33.94646
                            ],
                            [
                                18.47393,
                                -33.94668
                            ],
                            [
                                18.47393,
                                -33.94673
                            ],
                            [
                                18.47393,
                                -33.946733
                            ],
                            [
                                18.47393,
                                -33.946733
                            ],
                            [
                                18.47393,
                                -33.94702
                            ],
                            [
                                18.47394,
                                -33.94736
                            ],
                            [
                                18.47392,
                                -33.94777
                            ],
                            [
                                18.47393,
                                -33.94801
                            ],
                            [
                                18.4739,
                                -33.94823
                            ],
                            [
                                18.47383,
                                -33.94871
                            ],
                            [
                                18.4738,
                                -33.94913
                            ],
                            [
                                18.47371,
                                -33.94986
                            ],
                            [
                                18.47366,
                                -33.95035
                            ],
                            [
                                18.47356,
                                -33.95115
                            ],
                            [
                                18.47347,
                                -33.95191
                            ],
                            [
                                18.47343,
                                -33.95221
                            ],
                            [
                                18.47338,
                                -33.95282
                            ],
                            [
                                18.47329,
                                -33.95362
                            ],
                            [
                                18.4732,
                                -33.95428
                            ],
                            [
                                18.47316,
                                -33.95458
                            ],
                            [
                                18.47316,
                                -33.95458
                            ],
                            [
                                18.47316,
                                -33.95458
                            ],
                            [
                                18.47315,
                                -33.95472
                            ],
                            [
                                18.47296,
                                -33.95631
                            ],
                            [
                                18.47285,
                                -33.95726
                            ],
                            [
                                18.47281,
                                -33.95754
                            ],
                            [
                                18.47272,
                                -33.95833
                            ],
                            [
                                18.47268,
                                -33.95873
                            ],
                            [
                                18.4726,
                                -33.95944
                            ],
                            [
                                18.47261,
                                -33.95954
                            ],
                            [
                                18.4726,
                                -33.95955
                            ],
                            [
                                18.47258,
                                -33.95974
                            ],
                            [
                                18.47253,
                                -33.96027
                            ],
                            [
                                18.47246,
                                -33.96092
                            ],
                            [
                                18.47243,
                                -33.96102
                            ],
                            [
                                18.47233,
                                -33.96221
                            ],
                            [
                                18.47233,
                                -33.96221
                            ]
                        ]
                    }
                },
                {
                    "href": "https://transit.whereismytransport.com/api/journeys/FvIwnLmLy0uauqZxAQXaFA/itineraries/UZyXS7BFlUyXBaZxAQXa7A/legs/2",
                    "type": "Walking",
                    "distance": {
                        "value": 666,
                        "unit": "m"
                    },
                    "duration": 480,
                    "waypoints": [
                        {
                            "stop": {
                                "id": "GAW_yYTHwkSTn_6RjZ5v9g",
                                "href": "https://transit.whereismytransport.com/api/stops/GAW_yYTHwkSTn_6RjZ5v9g",
                                "agency": {
                                    "id": "xp_eNbqkYEaZP2YZkHwQqg",
                                    "href": "https://transit.whereismytransport.com/api/agencies/xp_eNbqkYEaZP2YZkHwQqg",
                                    "name": "Metrorail Western Cape",
                                    "culture": "en"
                                },
                                "name": "Rondebosch",
                                "code": "1964415770",
                                "geometry": {
                                    "type": "Point",
                                    "coordinates": [
                                        18.472641,
                                        -33.962226
                                    ]
                                },
                                "modes": [
                                    "Rail"
                                ]
                            },
                            "arrivalTime": "2016-08-29T16:14:00Z",
                            "departureTime": "2016-08-29T16:14:00Z"
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
                            "arrivalTime": "2016-08-29T16:22:00Z",
                            "departureTime": "2016-08-29T16:22:00Z"
                        }
                    ],
                    "geometry": {
                        "type": "LineString",
                        "coordinates": [
                            [
                                18.47233,
                                -33.96221
                            ],
                            [
                                18.472633,
                                -33.962238
                            ],
                            [
                                18.472653,
                                -33.962247
                            ],
                            [
                                18.472797,
                                -33.962298
                            ],
                            [
                                18.472815,
                                -33.962349
                            ],
                            [
                                18.473148,
                                -33.963152
                            ],
                            [
                                18.472842,
                                -33.963243
                            ],
                            [
                                18.472767,
                                -33.963299
                            ],
                            [
                                18.472722,
                                -33.963357
                            ],
                            [
                                18.472606,
                                -33.964223
                            ],
                            [
                                18.472452,
                                -33.96498
                            ],
                            [
                                18.472441,
                                -33.965114
                            ],
                            [
                                18.472458,
                                -33.96521
                            ],
                            [
                                18.472504,
                                -33.965332
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
                                "value": 16,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right onto Station Road",
                            "distance": {
                                "value": 100,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right onto Myrtle Road",
                            "distance": {
                                "value": 269,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Turn right onto Rouwkoop Road",
                            "distance": {
                                "value": 29,
                                "unit": "m"
                            }
                        },
                        {
                            "instruction": "Continue onto White Road",
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

### Legs

A leg is a section of an itinerary carried out by a passenger on one mode of transit (including walking) from some departure point to an arrival point.

#### Types of legs {#types-of-legs}

The Transit API currently supports two types of legs, each with a different response model.

A _Walking_ leg is one which the passenger is to travel by foot from one waypoint to another.  Walking legs are usually accompanied with directions.

A _Transit_ leg is one which uses a public transportation service based on scheduled or absolute frequency-based stop times.

#### Leg response model {#leg-model}

| Field | Type | Description |
| :--------- | :--- | :---- |
| type | string | The [type of leg](#types-of-legs), either _Walking_ or _Transit_. |
| distance | [Distance](#distance) | If available, the total distance of leg. |
| duration | integer | If available, the total duration of the leg in seconds. |
| line | [Line](#line-model) | **[**[Excludable](#excludable)**]** The line that is used on this leg of the itinerary. This is only returned for Transit legs. |
| vehicle | [Vehicle](#vehicle-model) | **[**[Excludable](#excludable)**]** Identifying information for the vehicle that is used on this leg of the itinerary. This is only returned for Transit legs. |
| fare | [Fare](#fare-model) | If available, the fare for this leg. |
| waypoints | Array of [Waypoint](#waypoint-model) | **[**[Excludable](#excludable)**]** The sequence of ordered waypoints that make up this leg. |
| directions | Array of [Direction](#direction-model) | **[**[Excludable](#excludable)**]** If available, the directions to take in order to complete the leg. |
| geometry | [GeoJSON](#geojson) LineString | **[**[Excludable](#excludable)**]** If available, the geographic shape of the leg. |

#### Retrieving a specific leg

##### Sample request

Retrieving an itinerary's leg can be done using the index of that leg as it exists in the itinerary. The index begins counting from 1.

```
GET api/journeys/8GYKddjcAk6j7aVUAMV3pw/itineraries/dnCQV5Kq0kaq5KVUAMV_eQ/legs/1
Accept: application/json
```

##### Sample response

```json
200 Ok
Content-Type: application/json
{
    "type": "Walking",
    "distance": {
        "value": 163,
        "unit": "m"
    },
    "duration": 91,
    "waypoints": [
        {
            "location": {
                "address": "34 Strand Street, Cape Town, 7925",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        18.422997,
                        -33.922033
                    ]
                }
            },
            "arrivalTime": "2015-01-15T12:00:32Z",
            "departureTime": "2015-01-15T12:00:32Z"
        },
        {
            "stop": {
                "id": "YhYX0avSXk6mVS-uY14zVQ",
                "href": "https://transit.whereismytransport.com/api/stops/YhYX0avSXk6mVS-uY14zVQ",
				"agency": {
					"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
                    "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
					"name": "Metrorail Cape Town",
					"culture": "en-ZA"
				},
                "name": "Cape Town",
                "code": "CPT SLT",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        18.425102,
                        -33.922153
                    ]
                },
				"modes": [
					"Rail"
				]
            },
            "arrivalTime": "2015-01-15T12:02:00Z",
            "departureTime": "2015-01-15T12:02:00Z"
        }
    ],
	"directions": [
		{
			"instruction": "Continue onto Strand Street",
			"distance": {
				"value": 18,
				"unit": "m"
			}
		},
		{
			"instruction": "Turn slight left onto Strand Street",
			"distance": {
				"value": 145,
				"unit": "m"
			}
		}
	],
    "geometry": {
        "type": "LineString",
        "coordinates": [
			[
				18.423071,
				-33.92186
			],
			[
				18.423218,
				-33.921973
			],
			[
				18.423407,
				-33.922046
			],
			[
				18.424113,
				-33.922576
			],
			[
				18.424427,
				-33.922801
			]
        ]
    }
}
```

#### Vehicle response model

Describes a single transit vehicle along a line so that it can be identified by passengers.

| Field | Type | Description |
| :--------- | :--- | :---- |
| designation | string | If available, an identifier for this vehicle as defined by the transit agency, or some other designation. |
| direction | string | If available, the direction of the vehicle, for example, "Northbound" or "Clockwise" |
| headsign | string | If available, identifying information (such as destination) displayed on the vehicle. |

#### Waypoint response model

A waypoint is a stopping point along an itinerary. It has either an arrival date and time or a departure date and time, or both.

| Field | Type | Description |
| :--------- | :--- | :---- |
| arrivalTime | [DateTime](#datetime) | The arrival date and time at this point of a leg. |
| departureTime | [DateTime](#datetime) | The departure date and time from this point of a leg. |
| stop | [Stop](#stop-model) | **[**[Excludable](#excludable)**]** The stop of the waypoint. This can be returned in either Walking or Transit legs. |
| location | [Location](#location-model) | The location of the waypoint if it is not a stop. This can be returned in only Walking legs. |

#### Direction response model

If available, the directions to follow in order to get from the start to the end of a leg.

| Field | Type | Description |
| :--------- | :--- | :---- |
| instruction | string | The instruction to follow. |
| distance | [Distance](#distance) | The distance to travel after the instruction has been followed. |

#### Location response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| address | string | The reverse geocoded address of the point. |
| geometry | [GeoJSON](#geojson) Point | The geographic point of the location. |

### Fares {#fares}

A fare is the cost incurred by a commuter when using a transit service.  Essentially, it is the price associated with a journey's itinerary for a particular fare product or set of fare products.

#### Fare response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| description | string | The description of the fare for this leg. |
| fareProduct | [FareProduct](#fare-products) | The fare product selected for this leg. |
| cost | [Cost](#cost) | The cost of this leg. |
| messages | Array of string | Any fare messages, such as required fare cards or special instructions. |

#### Specifying fare products

When creating or retrieving a journey, or when retrieving a journey's itineraries or legs, the default [fare products](#fare-products) will be used, if available. Specific fare products can be supplied in order to get a more specific fare result.

##### Sample request

When creating a new journey:

```json
POST api/journeys
Accept: application/json
Content-Type: application/json
{
    "geometry": {
        "type": "MultiPoint",
        "coordinates": [
			[
                18.422997,
                -33.922033
            ],
			[
                18.470095,
                -33.937826
            ]
        ]
    },
    "time": "2015-01-15T10:00:00Z",
	"timeType": "LatestArrival",
	"fareProducts": [ "vQYeSTuzMUWkPKZVATYFcg" ]
}
```

To retrieve a specific itinerary for a previously created journey, the following resource can be requested.

`GET api/journeys/{journeyId}/itineraries/{itineraryId}`

| Parameter | Type | Required | Description |
| :-------------- | :--- | :--- | :---- |
| journeyId | string | Required | The identifier of the journey. |
| itineraryId | string | Required | The identifier of the itinerary. |

##### Sample request

```
GET api/journeys/8GYKddjcAk6j7aVUAMV3pw/itineraries/dnCQV5Kq0kaq5KVUAMV_eQ
Accept: application/json
```

##### Sample response

```json
200 Ok
Content-Type: application/json
{
    "id": "dnCQV5Kq0kaq5KVUAMV_eQ",
    "href": "https://transit.whereismytransport.com/api/journeys/8GYKddjcAk6j7aVUAMV3pw/itineraries/dnCQV5Kq0kaq5KVUAMV_eQ",
    "departureTime": "2015-01-15T12:00:32Z",
    "arrivalTime": "2015-01-15T13:31:34Z",
    "duration": 4110,
    "distance": {
        "value": 5357,
        "unit": "m"
    },
    "legs": [
        {
            "type": "Walking",
            "distance": {
                "value": 163,
                "unit": "m"
            },
            "duration": 91,
            "waypoints": [
                {
                    "location": {
                        "address": "34 Strand Street, Cape Town, 7925",
                        "geometry": {
                            "type": "Point",
                            "coordinates": [
                                18.422997,
                                -33.922033
                            ]
                        }
                    },
                    "arrivalTime": "2015-01-15T12:00:32Z",
                    "departureTime": "2015-01-15T12:00:32Z"
                },
                {
                    "stop": {
                        "id": "YhYX0avSXk6mVS-uY14zVQ",
                        "href": "https://transit.whereismytransport.com/api/stops/YhYX0avSXk6mVS-uY14zVQ",
                        "agency": {
							"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
                            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
							"name": "Metrorail Cape Town",
							"culture": "en-ZA"
						},
                        "name": "Cape Town",
                        "code": "CPT SLT",
                        "geometry": {
                            "type": "Point",
                            "coordinates": [
                                18.425102,
                                -33.922153
                            ]
                        },
						"modes": [
							"Rail"
						]
                    },
                    "arrivalTime": "2015-01-15T12:02:00Z",
                    "departureTime": "2015-01-15T12:02:00Z"
                }
            ],
			"directions": [
				{
					"instruction": "Continue onto Strand Street",
					"distance": {
						"value": 18,
						"unit": "m"
					}
				},
				{
					"instruction": "Turn slight left onto Strand Street",
					"distance": {
						"value": 145,
						"unit": "m"
					}
				}
			],
            "geometry": {
                "type": "LineString",
                "coordinates": [
					[
						18.423071,
						-33.92186
					],
					[
						18.423218,
						-33.921973
					],
					[
						18.423407,
						-33.922046
					],
					[
						18.424113,
						-33.922576
					],
					[
						18.424427,
						-33.922801
					]
                ]
            }
        },
        {
            "type": "Transit",
            "distance": {
                "value": 4890,
                "unit": "m"
            },
            "duration": 3600,
            "line": {
                "id": "O29agx-avU-Xb1c-j_0Nvg",
                "href": "https://transit.whereismytransport.com/api/lines/O29agx-avU-Xb1c-j_0Nvg",
                "agency": {
					"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
                    "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
					"name": "Metrorail Cape Town",
					"culture": "en-ZA"
				},
                "name": "Southern Line",
                "mode": "Rail",
				"colour": "#ff35b5e5",
				"textColour": "#ffffffff"
            },
			"fare": {
				"description": "Standard fare.",
				"fareProduct": {
					"id": "EREREREREREREREREREREQ",
                    "href": "https://transit.whereismytransport.com/api/fareproducts/EREREREREREREREREREREQ",
					"agency": {
						"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
						"href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
						"name": "Metrorail Cape Town",
						"culture": "en-ZA"
					},
					"name": "Adult Single MetroPlus",
					"isDefault": true,
					"description": "First class ticket price for an adult making a one-way journey."
				},
				"cost": {
					"amount": 10.50,
					"currencyCode": "ZAR"
				},
				"messages": []
			},
            "waypoints": [
                {
                    "stop": {
                        "id": "YhYX0avSXk6mVS-uY14zVQ",
                        "href": "https://transit.whereismytransport.com/api/stops/YhYX0avSXk6mVS-uY14zVQ",
                        "agency": {
							"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
                            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
							"name": "Metrorail Cape Town",
							"culture": "en-ZA"
						},
                        "name": "Cape Town",
                        "code": "CPT SLT",
                        "geometry": {
                            "type": "Point",
                            "coordinates": [
                                18.425102,
                                -33.922153
                            ]
                        },
						"modes": [
							"Rail"
						]
                    },
                    "arrivalTime": "2015-01-15T12:02:00Z",
                    "departureTime": "2015-01-15T12:02:00Z"
                },
                {
                    "stop": {
                        "id": "n7O8Px_REk21cMelcP6Odw",
                        "href": "https://transit.whereismytransport.com/api/stops/n7O8Px_REk21cMelcP6Odw",
                        "agency": {
							"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
                            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
							"name": "Metrorail Cape Town",
							"culture": "en-ZA"
						},
                        "name": "Salt River",
                        "geometry": {
                            "type": "Point",
                            "coordinates": [
                                18.465346,
                                -33.926687
                            ]
                        },
						"modes": [
							"Rail"
						]
                    },
                    "arrivalTime": "2015-01-15T12:30:00Z",
                    "departureTime": "2015-01-15T12:32:00Z"
                },
                {
                    "stop": {
                        "id": "t_pbQXOlOEOcBnzrq3iE_g",
                        "href": "https://transit.whereismytransport.com/api/stops/t_pbQXOlOEOcBnzrq3iE_g",
                        "agency": {
							"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
                            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
							"name": "Metrorail Cape Town",
							"culture": "en-ZA"
						},
                        "name": "Woodstock",
                        "geometry": {
                            "type": "Point",
                            "coordinates": [
                                18.445819,
                                -33.924728
                            ]
                        },
						"modes": [
							"Rail"
						]
                    },
                    "arrivalTime": "2015-01-15T13:00:00Z",
                    "departureTime": "2015-01-15T13:02:00Z"
                }
            ],
            "geometry": {
                "type": "LineString",
                "coordinates": [
					[
						18.424775004386902,
						-33.92254795325198
					],
					[
						18.42703878879547,
						-33.92414597473452
					],
					[
						18.431464433670044,
						-33.92569055196698
					],
					[
						18.43801975250244,
						-33.926144572224985
					],
					[
						18.44130277633667,
						-33.92616237689167
					],
					[
						18.444993495941162,
						-33.92532555353428
					],
					[
						18.44797611236572,
						-33.92530774869268
					],
					[
						18.46121549606323,
						-33.92696358304087
					],
					[
						18.46503496170044,
						-33.92721284563916
					]
                ]
            }
        },
        {
            "type": "Walking",
            "distance": {
                "value": 304,
                "unit": "m"
            },
            "duration": 419,
            "waypoints": [
                {
                    "stop": {
                        "id": "kSukOlW7cES5C55WRpp41Q",
                        "href": "https://transit.whereismytransport.com/api/stops/kSukOlW7cES5C55WRpp41Q",
                        "agency": {
							"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
                            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
							"name": "Metrorail Cape Town",
							"culture": "en-ZA"
						},
                        "name": "Observatory",
                        "geometry": {
                            "type": "Point",
                            "coordinates": [
                                18.471437,
                                -33.937752
                            ]
                        },
						"modes": [
							"Rail"
						]
                    },
                    "arrivalTime": "2015-01-15T13:30:00Z",
                    "departureTime": "2015-01-15T13:30:00Z"
                },
                {
                    "location": {
						"address": "79 Station Road, Observatory, Cape Town, 7925",
                        "geometry": {
                            "type": "Point",
                            "coordinates": [
                                18.470095,
								-33.937826
                            ]
                        }
                    },
                    "arrivalTime": "2015-01-15T13:31:34Z",
                    "departureTime": "2015-01-15T13:30:00Z"
                }
            ],
			"directions": [
				{
					"instruction": "Continue",
					"distance": {
						"value": 91,
						"unit": "m"
					}
				},
				{
					"instruction": "Turn right onto Trill Road",
					"distance": {
						"value": 10,
						"unit": "m"
					}
				},
				{
					"instruction": "Turn right onto Disa Avenue",
					"distance": {
						"value": 54,
						"unit": "m"
					}
				},
				{
					"instruction": "Turn left onto Fairfield Road",
					"distance": {
						"value": 84,
						"unit": "m"
					}
				},
				{
					"instruction": "Turn right onto Herschel Road",
					"distance": {
						"value": 63,
						"unit": "m"
					}
				},
				{
					"instruction": "Turn left onto Station Road",
					"distance": {
						"value": 1,
						"unit": "m"
					}
				}
			],
            "geometry": {
                "type": "LineString",
                "coordinates": [
					[
						18.471225,
						-33.937796
					],
					[
						18.471458,
						-33.938543
					],
					[
						18.471473,
						-33.93859
					],
					[
						18.471365,
						-33.938622
					],
					[
						18.471212,
						-33.938148
					],
					[
						18.470335,
						-33.938359
					],
					[
						18.470296,
						-33.938299
					],
					[
						18.470212,
						-33.938092
					],
					[
						18.470105,
						-33.937827
					],
					[
						18.470096,
						-33.937829
					]
                ]
            }
        }
    ]
}
```

### Fare Products {#fare-products}

A fare product is a fare scheme offered to passenger by an agency and will decide the total [fare](#fares) incurred when using a transit service. Note that they may be subject to eligibility restrictions. For example, a "Child Single" fare product might only be allowed to be used by children.

#### Fare Product response model

| Field | Type | Description |
| :--------- | :--- | :---- |
| id | string | The identifier of the fare product. |
| href | string | The hyperlink pointing to the fare product. |
| agency | [Agency](#agency-model) | **[**[Excludable](#excludable)**]** The fare product's agency. |
| name | string | The commuter-friendly name of the fare product. |
| isDefault | bool | Flag specifying whether this is the default fare product for this agency. |
| description | string | A commuter-friendly description of the fare product. |

#### Retrieving fare products

Retrieves a collection of fare products.

`GET api/fareproducts?agencies={agencies}&limit={limit}&offset={offset}&at={at}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| agencies | [Agencies Filter](#agencies-filter) | The list of agencies to filter the results by. |
| limit | int | The maximum number of entities to be returned. Default is 100. |
| offset | int | The offset of the first entity returned. Default is 0. |
| at | [DateTime](#datetime) | The point in time from which to query. Defaults to the current date and time. |

##### Sample request

```json
GET api/fareproducts?agencies=CUJ2ZhcOm0y7wO1KsjUgPA
Accept: application/json
```

##### Sample response

```json
200 Ok
Content-Type: application/json
[
    {
        "id": "q5d1eTAVa87aG6Ki2lRqmw",
        "href": "https://transit.whereismytransport.com/api/fareproducts/q5d1eTAVa87aG6Ki2lRqmw",
		"agency": {
			"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
			"name": "Metrorail Cape Town",
			"culture": "en-ZA"
		},
        "name": "Adult Single",
		"isDefault": true,
        "description": "Default ticket price for an adult making a one-way journey."
    },
    {
        "id": "UKQrSP1DLpVh5Gj8V-AhmL",
        "href": "https://transit.whereismytransport.com/api/fareproducts/UKQrSP1DLpVh5Gj8V-AhmL",
		"agency": {
			"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
			"name": "Metrorail Cape Town",
			"culture": "en-ZA"
		},
        "name": "Child Single",
		"isDefault": false,
        "description": "Ticket price for an child making a one-way journey."
    }
]
```
