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

The **Accept** header describes the format of the content that the client can accept. All requests must specify this header as **application/json**. If the **Accept** header is not specified or the type is set to a value other than **application/json** then a **406** [HTTP status code](#status-codes) will be returned.

### Sample request

```http
GET api/agencies
Accept: application/json
```

#### Content type

The **Content-Type** header describes the format of the data being posted to the server. All POST requests must specify this header as **application/json**. If the **Content-Type** header is not specified or the type is set to a value other than **application/json** then a **415** [HTTP status code](#status-codes) will be returned.

### Sample request

```http
POST api/journeys
Accept: application/json
Content-Type: application/json
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

```http
POST https://identity.whereismytransport.com/connect/token
Content-Type: application/x-www-form-urlencoded
client_id=6a73386c-5bbf-4b4b-ae09-0ed45d87b5ad
client_secret=neph3dext3Ct+QHc5wClxTWE1Y+AKNTS28UFH39SXy4=
grant_type=client_credentials
scope=transportapi:all
```

##### Sample response

```http
201 Created
{
    "access_token": "eyJ0eXAiOiJ32aQiLCJhbGciOiJSUzI1NiIsIfg1iCI6ImEzck1VZ01Gd8d0UGNsTGE2eUYz...",
    "expires_in": 3600,
    "token_type": "Bearer"
}
```

Once this token is received, it must be added to the Authorization HTTP header in order to perform successful requests against the API.

##### Sample request

```http
GET api/agencies
Authorization: Bearer eyJ0eXAiOiJ32aQiLCJhbGciOiJSUzI1NiIsIfg1iCI6ImEzck1VZ01Gd8d0UGNsTGE2eUYz...
```

### Errors

The API uses conventional [HTTP status codes](#status-codes) to indicate the result of a request. Codes within the 200s indicate that the request was successful.  Codes within the 400s indicate that the request was somehow badly formed (such as a missing or incorrectly formatted field). 500s are typically returned when something unexpected goes wrong on the server.

#### Error response model 

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| message | string | A general message indicating the type of error. |
| fields | Dictionary of error messages | Human-readable error messages for each problematic input or query field. Only returned for a 400 (Bad Request). |

##### Sample response

```http
400 Bad Request
{
    "message":"The request has invalid or missing fields. Please visit our documentation.",
    "fields":{
        "timeType":[
            "Value 'After' is invalid. TimeType must be one of 'DepartAfter', 'ArriveBefore'."
        ],
        "maxItineraries":[
            "Value '10' is invalid. The maximum number of itineraries that may be requested is 5."
        ]
    }
}
```

### Identifiers

Almost all entities in the Transit API are identified through the use of a globally unique identifier. This identifier is specified as a 22 character long, case-sensitive string of URL-friendly characters; __a__ to __z__, __A__ to __Z__, __0__ to __9__, - (hyphen) and _ (underscore). See the `id` field in the sample response below.

#### Sample request

```http
GET api/agencies/5kcfZkKW0ku4Uk-A6j8MFA
```

##### Sample response

```http
{
    "id":"5kcfZkKW0ku4Uk-A6j8MFA",
    "href":"https://platform.whereismytransport.com/api/agencies/5kcfZkKW0ku4Uk-A6j8MFA",
    "name":"MyCiTi",
    "culture":"en"
}
```

**Note:** Identifiers are case-sensitive.

### Resource Linking

The API presents a simple and elegant approach to referencing of and linking to resources. Every available resource has a hypertext reference field, **href**. This value represents a fully qualified URL to where the resource resides. Presenting such a field for every resource aids discoverability by allowing for new resources to be consumed by just embedding a new reference link.

##### Sample response

```http
200 Ok
{
    "id": "9hKEat91jEG0z6XoAMKShw",
    "href": "https://platform.whereismytransport.com/api/stops/9hKEat91jEG0z6XoAMKShw",
    "agency": {
        "id": "edObkk6o-0WN3tNZBLqKPg",
        "href": "https://platform.whereismytransport.com/api/agencies/edObkk6o-0WN3tNZBLqKPg",
        "name": "Gautrain",
        "culture": "en"
    },
    "name": "102 Witch-Hazel Ave",
    "geometry": {
        "type": "Point",
        "coordinates": [
            28.189397,
            -25.873595
        ]
    }
}
```

### Excluding data 

In order to reduce payload, it is possible to exclude certain objects or collections from the model returned in the body of the HTTP response. This is done through the use of the `exclude` query. Although not everything can be exluded. Attributes which are _excludable_ are described in this specification with the [Excludable](#excludable) tag.

| Parameter | Type | Required | Description |
| :-------------- | :--- | :---- | :---- |
| exclude | string | Optional | Comma-separated object or collection names to exclude from the response. Unknown names will be ignored. |

When excluding resource objects (i.e. those which have their own URI), the containing object with their `id` and `href` fields will remain. This is to preserve discoverability while still reducing payload. Collections, on the other hand, will be excluded entirely.

##### Sample request

The request below will exclude **geometry** and **directions** from the resource model.

```http
GET api/journeys/8GYKddjcAk6j7aVUAMV3pw?exclude=geometry,directions
```

### Understanding Scheduled Data

Everything retrievable from the API is considered scheduled data. This means that all queries are "at" a certain scheduled date and time. An agency, for example, may schedule a line's name to change, not now, but only after a certain date.  A new stop could be scheduled to only be returned from the API at a given date. The important thing to note that is an entity could be deprecated in future schedules. This means that any entity resource URI could return a 404. Applications built on this API are highly encouraged to cater for this.

All endpoints are queried for at the current schedule.  

### Agencies Filter 

Most requests to the API can be filtered by agencies. If no filter is provided, then all agencies are considered.

| Parameter | Type | Required | Description |
| :-------------- | :--- | :--- | :---- |
| agencies | string | Optional | Comma-separated agency identifiers. |

##### Sample request

```http
GET api/stops?agencies=CUJ2ZhcOm0y7wO1KsjUgPA
```

The sample request above will only retrieve stops from the agency identified by **CUJ2ZhcOm0y7wO1KsjUgPA**.

More than one agency can be specified. This is done using a comma-separated list.

##### Sample request

```http
GET api/stops?agencies=CUJ2ZhcOm0y7wO1KsjUgPA,TQM8HCPXFEFXwm45QGvN_f
```

The sample request above will retrieve stops from the agencies identified by **CUJ2ZhcOm0y7wO1KsjUgPA** or **TQM8HCPXFEFXwm45QGvN_f**.

### Pagination 

Depending on the structure of a query, a lot of results could be returned from the API. For that reason, the results are paginated so to ensure that responses are easier to handle and that payload it kept to a manageable size.

| Parameter | Type | Required | Description |
| :-------------- | :--- | :---- | :---- |
| limit | int | Optional | The maximum number of entities to be returned. The default and maximum is typically 100. |
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
    "cost":{
        "amount":10.50,
        "currencyCode":"ZAR"
    }
}
```

#### GeoJSON 

[GeoJSON](http://geojson.org) is a JSON format for encoding geographic data structures. The Transit API uses the Point, MultiPoint and LineString geometry types.

A typical GeoJSON structure consists of a type attribute and an array of coordinates. 

**Note:**  GeoJSON represents geographic coordinates with longitude first and then latitude, `[longitude, latitude]`. i.e. `[x, y]` in the Cartesian coordinate system.

##### Sample

The following GeoJSON Point represents the coordinates for Cape Town's city centre.

```http
{
    "geometry": {
        "type":"Point",
        "coordinates":[
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

The mode of transit describes the type of vehicle that is used along a line. The following table describes the modes currently supported by the Transit API.

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

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| id | string | The identifier of the agency. |
| href | string | The hyperlink pointing to the agency. |
| name | string | The full name of the agency. |
| culture | string | The name of the [culture](#culture), based on RFC 4646. |

#### Retrieving agencies

Retrieves a collection of agencies.

`GET api/agencies?point={point}&radius={int}&bbox={bbox}&agencies={agencies}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| point | [Point](#point) | The point from where to search for nearby agencies. Agencies will be returned in order of their distance from this point (from closest to furthest). |
| radius | integer | The distance in metres from the point to search for nearby agencies. This filter is optional. |
| bbox | [Bounding Box](#bounding-box) | The bounding box from where to retrieve agencies. This will be ignored if a point is provided in the query.  |
| agencies | [Agencies Filter](#agencies-filter) | The list of agencies to filter the results by. |
| limit | int | See [Pagination](#pagination). |
| offset | int | See [Pagination](#pagination). |

##### Sample request

```http
GET api/agencies?agencies=TW5uToIpeWUEaTlYDTnK
```

##### Sample response

```http
200 Ok
[
    {
        "id": "fEZ57B5v-gz5aFhj8ueG52",
        "href": "https://transit.whereismytransport.com/api/agencies/fEZ57B5v-gz5aFhj8ueG52",
        "name": "Golden Arrow",
        "culture": "en-ZA"
    },
    {
        "id": "CUJ2ZhcOm0y7wO1KsjUgPA",
        "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
        "name": "Metrorail Cape Town",
        "culture": "en-ZA"
    }
    {
        "id": "TQM8HCPXFEFXwm45QGvN_f",
        "href": "https://transit.whereismytransport.com/api/agencies/TQM8HCPXFEFXwm45QGvN_f",
        "name": "MyCiTi",
        "culture": "en-ZA"
    }
]
```

#### Retrieving a specific agency

Retrieves an agency by its identifier.

`GET api/agencies/{agencyId}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| agencyId | string | The identifier of the agency. |

##### Sample request

```http
GET api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA
```

##### Sample response

```http
200 Ok
{
    "id": "CUJ2ZhcOm0y7wO1KsjUgPA",
    "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
    "name": "Metrorail Cape Town",
    "culture": "en-ZA"
}
```

### Stops

A location where passengers can board or alight from a transit vehicle.

#### Stop response model

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| id | string | The identifier of the stop. |
| href | string | The hyperlink pointing to the stop. |
| agency | [Agency](#agency-model) | **[**[Excludable](#excludable)**]** The agency. |
| name | string | The full name of the stop. |
| code | string | If available, the passenger code of the stop. |
| geometry | [GeoJSON](#geojson) Point | The geographic point of the stop. |
| modes | Array of [Mode](#modes) | The modes that are served by this stop. |
| parentStop | [Stop](#stop-model) | **[**[Excludable](#excludable)**]** If applicable, the parent stop. |

#### Retrieving stops

Retrieves a collection of stops.

`GET api/stops?point={point}&radius={radius}&bbox={bbox}&modes={modes}&agencies={agencies}&servesLines={lineIds}&limit={int}&offset={int}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| point | [Point](#point) | The point from where to search for nearby stops. Stops will be returned in order of their distance from this point (from closest to furthest). |
| radius | integer | The distance in metres from the point to search for nearby stops. This filter is optional. |
| bbox | [Bounding Box](#bounding-box) | The bounding box from where to retrieve stops. This will be ignored if a point is provided in the query.  |
| modes | string | A string of comma-separated [transit modes](#modes). |
| agencies | [Agencies Filter](#agencies-filter) | A string of comma-separated agency identifiers to filter the results by. |
| servesLines | string | A string of comma-separated line identifiers to filter the results by. |
| showChildren | bool | Specifies whether or not to display children stops that satisfy the query parameters. Default is false. |
| limit | int | The maximum number of entities to be returned. Default is 100. |
| offset | int | The offset of the first entity returned. Default is 0. |

##### Sample request

```http
GET api/stops?agencies=CUJ2ZhcOm0y7wO1KsjUgPA&servesLines=4ODGMWFlNUCg3a0g0AeCIw&point=-33.923400,18.421586&radius=1750&modes=rail,bus
```

##### Sample response

```http
200 Ok
Content-Type: application/json
[
    {
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
		  "Rail", "Bus"
		]
    },
    {
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
    {
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
		  "Rail", "Bus"
		]
    }
]
```

#### Retrieving a specific stop

Retrieves a stop by its identifier.

`GET api/stops/{stopId}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| stopId | string | The identifier of the stop. |
| at | [DateTime](#datetime) | The point in time from which to query. Defaults to the current date and time. |

##### Sample request

```http
GET api/stops/YhYX0avSXk6mVS-uY14zVQ
```

##### Sample response

```http
200 Ok
{
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
}
```

#### Retrieving child stops for some parent stop

Retrieves all children of the parent stop specified by its identifier.

`GET api/stops/{stopId}/stops`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| stopId | string | The identifier of the parent stop. |

##### Sample request

```http
GET api/stops/Z_usyGh-QkGjDNGeQZFEpg/stops
```

##### Sample response

```http
200 Ok
[
  {
    "id": "Z_Uz_zjFrQyXGvoWGtwqVg",
    "href": "http://localhost:26047/api/stops/Z_Uz_zjFrQyXGvoWGtwqVg",
    "agency": {
      "id": "J5kYAnLU20GNigwhB_OmAA",
      "href": "http://localhost:26047/api/agencies/J5kYAnLU20GNigwhB_OmAA",
      "name": "StopHierarchyAgency",
      "culture": "en-US"
    },
    "name": "PlatformA",
    "geometry": {
      "type": "Point",
      "coordinates": [
        18.424717,
        -33.920663
      ]
    },
    "modes": [
      "LightRail"
    ],
    "parentStop": {
      "id": "Z_usyGh-QkGjDNGeQZFEpg",
      "href": "http://localhost:26047/api/stops/Z_usyGh-QkGjDNGeQZFEpg",
      "agency": {
        "id": "J5kYAnLU20GNigwhB_OmAA",
        "href": "http://localhost:26047/api/agencies/J5kYAnLU20GNigwhB_OmAA",
        "name": "StopHierarchyAgency",
        "culture": "en-US"
      },
      "name": "CapeTownStation",
      "geometry": {
        "type": "Point",
        "coordinates": [
          18.424716,
          -33.920662
        ]
      },
      "modes": [
        "LightRail",
        "Subway"
      ]
    }
  },
  {
    "id": "a4gxy5ZqguUzZEMXGWpFoQ",
    "href": "http://localhost:26047/api/stops/a4gxy5ZqguUzZEMXGWpFoQ",
    "agency": {
      "id": "J5kYAnLU20GNigwhB_OmAA",
      "href": "http://localhost:26047/api/agencies/J5kYAnLU20GNigwhB_OmAA",
      "name": "StopHierarchyAgency",
      "culture": "en-US"
    },
    "name": "PlatformB",
    "geometry": {
      "type": "Point",
      "coordinates": [
        18.424715,
        -33.920661
      ]
    },
    "modes": [
      "Subway"
    ],
    "parentStop": {
      "id": "Z_usyGh-QkGjDNGeQZFEpg",
      "href": "http://localhost:26047/api/stops/Z_usyGh-QkGjDNGeQZFEpg",
      "agency": {
        "id": "J5kYAnLU20GNigwhB_OmAA",
        "href": "http://localhost:26047/api/agencies/J5kYAnLU20GNigwhB_OmAA",
        "name": "StopHierarchyAgency",
        "culture": "en-US"
      },
      "name": "CapeTownStation",
      "geometry": {
        "type": "Point",
        "coordinates": [
          18.424716,
          -33.920662
        ]
      },
      "modes": [
        "LightRail",
        "Subway"
      ]
    }
  },
  {
    "id": "Vvwj0Ifj0zERtBX9VnP2Fg",
    "href": "http://localhost:26047/api/stops/Vvwj0Ifj0zERtBX9VnP2Fg",
    "agency": {
      "id": "J5kYAnLU20GNigwhB_OmAA",
      "href": "http://localhost:26047/api/agencies/J5kYAnLU20GNigwhB_OmAA",
      "name": "StopHierarchyAgency",
      "culture": "en-US"
    },
    "name": "PlatformC",
    "geometry": {
      "type": "Point",
      "coordinates": [
        18.424717,
        -33.920661
      ]
    },
    "modes": [],
    "parentStop": {
      "id": "Z_usyGh-QkGjDNGeQZFEpg",
      "href": "http://localhost:26047/api/stops/Z_usyGh-QkGjDNGeQZFEpg",
      "agency": {
        "id": "J5kYAnLU20GNigwhB_OmAA",
        "href": "http://localhost:26047/api/agencies/J5kYAnLU20GNigwhB_OmAA",
        "name": "StopHierarchyAgency",
        "culture": "en-US"
      },
      "name": "CapeTownStation",
      "geometry": {
        "type": "Point",
        "coordinates": [
          18.424716,
          -33.920662
        ]
      },
      "modes": [
        "LightRail",
        "Subway"
      ]
    }
  }
]
```

### Stop Timetables

A timetable of vehicles arriving and departing from a stop along their respective routes.

#### Stop Timetable response model {#stop-timetable-model}

| Attribute | Type | Description |
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
| stopId | string | The identifier of the stop. |
| earliestArrivalTime | [DateTime](#datetime) | The earliest arrival date and time to include in the timetable. |
| limit | int | The maximum number of entities to be returned. Default is 10. |

##### Sample request

```http
GET api/stops/YhYX0avSXk6mVS-uY14zVQ/timetables?earliestArrivalTime=2015-01-15T13:00:00Z
```

##### Sample response

```http
200 Ok
[
	{
		"arrivalTime": "2015-01-15T13:25:00Z",
		"departureTime": "2015-01-15T13:27:00Z",
		"line": {
			"id": "MBATdZOrwEKu6Zd_ieFNOA",
            "href": "https://transit.whereismytransport.com/api/lines/MBATdZOrwEKu6Zd_ieFNOA",
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
		"vehicle": {
			"designation":"002",
			"direction":"Northbound",
			"headsign":"Cape Town"
		},
	},
	{
		"arrivalTime": "2015-01-15T14:00:00Z",
		"departureTime": "2015-01-15T14:02:00Z",
		"line": {
			"id": "MBATdZOrwEKu6Zd_ieFNOA",
            "href": "https://transit.whereismytransport.com/api/lines/MBATdZOrwEKu6Zd_ieFNOA",
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
		"vehicle": {
			"designation":"001",
			"direction":"Northbound",
			"headsign":"Cape Town"
		},
	}
]
```

### Lines

A grouping together of routes marketed to passengers as a single section of the transit network.

#### Line response model {#line-model}

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| id | string | The identifier of the line. |
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
| agencies | [Agencies Filter](#agencies-filter) | A comma-separated list of agency identifiers to filter the results by. |
| servesStops | string | A comma-separated list of stop identifiers that represent stops which the returned lines must serve or visit. |
| limit | int | The maximum number of entities to be returned. Default is 100. |
| offset | int | The offset of the first entity returned. Default is 0. |

##### Sample request

```http
GET api/lines?agencies=TW5uToIpeWUEaTlYDTnK&servesStops=4ODGMWFlNUCg3a0g0AeCIw
```

##### Sample response

```http
200 Ok
[
    {
        "id": "O29agx-avU-Xb1c-j_0Nvg",
        "href": "https://transit.whereismytransport.com/api/lines/O29agx-avU-Xb1c-j_0Nvg",
		"agency": {
			"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
			"name": "Metrorail Cape Town",
			"culture": "en-ZA"
		},
        "name": "Southern Line",
		"shortName": "South",
		"description": "This line goes from the central business district of Cape Town, through the Southern Suburbs, to Simonstown.",
        "mode": "Rail",
		"colour": "#ff35b5e5",
		"textColour": "#ffffffff"
    },
    {
        "id": "Vuklew1T3k6Pvt2kmLOUlw",
        "href": "https://transit.whereismytransport.com/api/lines/Vuklew1T3k6Pvt2kmLOUlw",
		"agency": {
			"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
			"name": "Metrorail Cape Town",
			"culture": "en-ZA"
		},
        "name": "Northern Line",
		"shortName": "North",
        "mode": "Rail",
		"colour": "#ff66b5e5",
		"textColour": "#ffffffff"
    },
    {
        "id": "tqMiFlDpWkOrjcnwVSOd5g",
        "href": "https://transit.whereismytransport.com/api/lines/tqMiFlDpWkOrjcnwVSOd5g",
		"agency": {
			"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
            "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
			"name": "Metrorail Cape Town",
			"culture": "en-ZA"
		},
        "name": "Central Line",
		"shortName": "Central",
        "mode": "Rail",
		"colour": "#ff0000ee",
		"textColour": "#ffffffff"
    }
]
```

#### Retrieving a specific line

Retrieves a line by its identifier.

`GET api/lines/{lineId}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| lineId | string | The identifier of the line. |

##### Sample request

```http
GET api/lines/O29agx-avU-Xb1c-j_0Nvg
```

##### Sample response

```http
200 Ok
{
    "id": "O29agx-avU-Xb1c-j_0Nvg",
    "href": "https://transit.whereismytransport.com/api/lines/O29agx-avU-Xb1c-j_0Nvg",
	"agency": {
		"id": "CUJ2ZhcOm0y7wO1KsjUgPA",
        "href": "https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
		"name": "Metrorail Cape Town",
		"culture": "en-ZA"
	},
    "name": "Southern Line",
	"shortName": "South",
    "mode": "Rail",
	"colour": "#ff35b5e5",
	"textColour": "#ffffffff"
}
```

### Line Timetables

A timetable of vehicles travelling on a line.

#### Line Timetable response model

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| vehicle | [Vehicle](#vehicle-model) | If available, identifying information for the vehicle running at this time. |
| waypoints | Array of [Waypoint](#waypoint-model) | The sequence of ordered way points that make up this line timetable. |

#### Retrieving a line timetable

Retrieves a timetable for a line, consisting of a list of departures on this line in order of departure time.

`GET api/lines/{lineId}/timetables?earliestDepartureTime={DateTime}&departureStopId={stop}&arrivalStopId={stop}&limit={int}`

| Query Parameter | Type | Notes |
| :-------------- | :--- | :---- |
| lineId | string | Required line identifier to get timetables by. |
| earliestDepartureTime | [DateTime](#datetime) | Optional earliest departure time on that line to be included in the timetable. |
| departureStopId | string | Optional stop identifier - bounds results to only occur after this stop. |
| arrivalStopId | string | Optional stop identifier - bounds results to only occur before this stop. |
| limit | int | The maximum number of entities to be returned. Default is 10. |

#####Sample request

```http
GET api/lines/abcdefghijklmn/timetables?earliestArrivalTime=2020-01-15T10:00:00&departureStopId=kSukOlW7cES5C55WRpp41Q&arrivalStopId=cuKWweFQ9UmxwAcHcht48A
```

#####Sample response

```http
200 Ok
[
    {
		"vehicle": {
			"designation":"002",
			"direction":"Northbound",
			"headsign":"Cape Town"
		},
        "waypoints":[
            {
                "stop":{
                    "id":"kSukOlW7cES5C55WRpp41Q",
                    "href":"https://transit.whereismytransport.com/api/stops/kSukOlW7cES5C55WRpp41Q",
                    "agency":{
                        "id":"CUJ2ZhcOm0y7wO1KsjUgPA",
                        "href":"https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
                        "name":"Metrorail Cape Town",
                        "culture":"en-ZA"
                    },
                    "name":"Observatory",
                    "geometry":{
                        "type":"Point",
                        "coordinates":[
                            18.471437,
                            -33.937752
                        ]
                    },
					"modes": [
						"Rail"
					]
                },
                "arrivalTime":"2020-01-15T12:58:00Z",
                "departureTime":"2020-01-15T13:00:00Z"
            },
            {
                "stop":{
                    "id":"t_pbQXOlOEOcBnzrq3iE_g",
                    "href":"https://transit.whereismytransport.com/api/stops/t_pbQXOlOEOcBnzrq3iE_g",
                    "agency":{
                        "id":"CUJ2ZhcOm0y7wO1KsjUgPA",
                        "href":"https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
                        "name":"Metrorail Cape Town",
                        "culture":"en-ZA"
                    },
                    "name":"Woodstock",
                    "geometry":{
                        "type":"Point",
                        "coordinates":[
                            18.445819,
                            -33.924728
                        ]
                    },
					"modes": [
						"Rail"
					]
                },
                "arrivalTime":"2020-01-15T13:08:00Z",
                "departureTime":"2020-01-15T13:10:00Z"
            },
            {
                "stop":{
                    "id":"HhpXNI3eakyC-d7CFtCnLw",
                    "href":"https://transit.whereismytransport.com/api/stops/HhpXNI3eakyC-d7CFtCnLw",
                    "agency":{
                        "id":"CUJ2ZhcOm0y7wO1KsjUgPA",
                        "href":"https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
                        "name":"Metrorail Cape Town",
                        "culture":"en-ZA"
                    },
                    "name":"Salt River",
                    "geometry":{
                        "type":"Point",
                        "coordinates":[
                            18.465346,
                            -33.926687
                        ]
                    },
					"modes": [
						"Rail"
					]
                },
                "arrivalTime":"2020-01-15T13:18:00Z",
                "departureTime":"2020-01-15T13:20:00Z"
            },
            {
                "stop":{
                    "id":"cuKWweFQ9UmxwAcHcht48A",
                    "href":"https://transit.whereismytransport.com/api/stops/cuKWweFQ9UmxwAcHcht48A",
                    "agency":{
                        "id":"CUJ2ZhcOm0y7wO1KsjUgPA",
                        "href":"https://transit.whereismytransport.com/api/agencies/CUJ2ZhcOm0y7wO1KsjUgPA",
                        "name":"Metrorail Cape Town",
                        "culture":"en-ZA"
                    },
                    "name":"Cape Town",
                    "code":"CPT SLT",
                    "geometry":{
                        "type":"Point",
                        "coordinates":[
                            18.425102,
                            -33.922153
                        ]
                    },
					"modes": [
						"Rail"
					]
                },
                "arrivalTime":"2020-01-15T13:25:00Z",
                "departureTime":"2020-01-15T13:27:00Z"
            }
        ]
    }
]
```

### Journeys

A journey is the traveling of a passenger from a departure point to an arrival point.  A journey can consist of zero to many possible itineraries, each a travel option in getting from A to B. An itinerary consists of one to many legs, describing the path and mode of transit, to take in order to complete the journey.

#### Journey response model {#journey-model}

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| id | string | The identifier of the journey. |
| href | string | The hyperlink pointing to the journey. |
| geometry | [GeoJSON](#geojson) MultiPoint | An ordered GeoJSON MultiPoint representing the departure and arrival points for the the journey. |
| time | [DateTime](#datetime) | The requested date and time for the journey.  |
| timeType | [TimeType](#timeType) | Specifies whether this is an ArriveBefore or DepartAfter request. |
| profile | [Profile](#profile) | The profile used to calculate and order itineraries. |
| fareProducts | Array of string | The list of [fare products](#fare-products) identifiers to try to use when calculating the journey fare. |
| maxItineraries | integer | The maximum number of itineraries to return. |
| only | [Filter](#filter) | The explicit set of modes and agencies to use. If unset, all modes and agencies are used. |
| omit | [Filter](#filter) | The explicit set of modes and agencies to exclude. If unset, all modes and agencies are used. If only one mode or agency is excluded all other modes or agencies are used. Omit takes preference over only if both are specified for the same mode or agency.|
| itineraries | Array of [Itinerary](#itinerary-model) | **[**[Excludable](#excludable)**]** The available itineraries for this journey. |




#### Creating a journey

Creating a new journey is done by posting the journey's criteria to the resource.

`POST api/journeys`

| Attribute | Type | Required | Description |
| :--------- | :--- | :--- | :---- |
| geometry | [GeoJSON](#geojson) MultiPoint | Required | An ordered GeoJSON MultiPoint representing the departure and arrival points for the the journey. Exactly two points must be provided. |
| time | [DateTime](#datetime) | Optional | The requested date and time for the journey. Defaults to Now. |
| timeType | [TimeType](#timeType) | Optional | Specifies whether this is an ArriveBefore or DepartAfter request. Defaults to DepartAfter. |
| profile | Array of [Profile](#profile) | Required | The profile used to calculate and order itineraries. |
| fareProducts | Array of string | Optional | The list of [fare products](#fare-products) identifiers to try to use when calculating the journey fare. |
| maxItineraries | integer | Optional | The maximum number of itineraries to return. This must be a value between or including 1 and 5. Default is 3. |
| only | [Filter](#filter) | Optional | The explicit set of modes and agencies to use. If unset, all modes and agencies are used. |
| omit | [Filter](#filter) | Optional | The explicit set of modes and agencies to exclude. If unset, all modes and agencies are used. If only one mode or agency is excluded all other modes or agencies are used. Omit takes preference over only if both are specified for the same mode or agency.|

##### Sample request

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
    "earliestDepartureTime": "2015-01-15T10:00:00Z",
    "maxItineraries": 1,
	"only":{    
        "agencies": ["CUJ2ZhcOm0y7wO1KsjUgPA"],
		"modes": ["Rail"]
    },
    "omit": {
        "modes": ["Air"]
    },
    "time": "2015-01-15T10:00:00Z",
	"timeType": "DepartAfter",
	"profile" : "ClosestToTime",
	"fareProducts": [ "vQYeSTuzMUWkPKZVATYFcg" ]
}
```

##### Sample response

```json
201 Created
Content-Type: application/json
{
    "id": "8GYKddjcAk6j7aVUAMV3pw",
    "href": "https://transit.whereismytransport.com/api/journeys/8GYKddjcAk6j7aVUAMV3pw",
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
	"timeType": "DepartAfter",
	"profile" : "ClosestToTime",
	"fareProducts": [ "vQYeSTuzMUWkPKZVATYFcg" ],
    "maxItineraries": 1,
	"only":{    
        "agencies": [
		  "CUJ2ZhcOm0y7wO1KsjUgPA"
		],
		"modes": [
		  "Rail"
		]
    },
    "omit": {
        "modes": [
		  "Air"
		]
    },
    "itineraries": [
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
					"vehicle": {
						"designation": "0195",
						"direction": "Outbound",
						"headsign": "Simonstown via Wynberg"
					},
					"fare": {
						"description": "Standard fare.",
						"fareProduct": {
							"id": "pTjYadjcAk6j7aVUAMV9rr",
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
                        "value": 300,
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
    ]
}
```

##### Time types

Specifies whether this is an ArriveBefore or DepartAfter request.

| Value | Description |
| :--------- | :--- | :---- |
| DepartAfter | This journey must depart after the given time, at the earliest. |
| ArriveBefore | This journey must arrive before the given time, at the latest. |

##### Profiles

The profile used to calculate the itinerary options for a journey.

| Value | Description |
| :--------- | :--- | :---- |
| ClosestToTime | Absolutely closest to the requested date - earliest for DepartAfter, and latest for ArriveBefore. |
| FewestTransfers | Fewest connections between vehicles, also prioritising closest to time. |

#### Retrieving a journey

Retrieving a previously requested journey and its itineraries can be done using the following request.

`GET api/journeys/{journeyId}`

| Parameter | Type | Description |
| :-------------- | :--- | :---- |
| journeyId | string | The identifier of the journey. |

##### Sample request

The following request will return the same journey model from the `POST api/journey` sample request above.

```
GET api/journeys/8GYKddjcAk6j7aVUAMV3pw
Accept: application/json
```

##### Sample response

```json
200 Ok
Content-Type: application/json
{
    <omitted for brevity>
}
```

### Itineraries

#### Itinerary response model {#itinerary-model}

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| id | string | The identifier of the itinerary. |
| href | string | The hyperlink pointing to the itinerary. |
| departureTime | [DateTime](#datetime) | The departure date and time for the itinerary. |
| arrivalTime | [DateTime](#datetime) | The arrival date and time for the itinerary. |
| distance | [Distance](#distance) | If available, the total distance of itinerary. |
| duration | integer | If available, the total duration of the itinerary in seconds. |
| legs | Array of [Leg](#leg-model) | **[**[Excludable](#excludable)**]** The sequence of legs that make up this itinerary. |

#### Retrieving a specific itinerary

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
    "href": "https://transit.whereismytransport.com/api/journeys/U8HA6kfSNkSDc6YCAG_paA/itineraries/dnCQV5Kq0kaq5KVUAMV_eQ",
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
			"vehicle": {
				"designation": "0195",
				"direction": "Outbound",
				"headsign": "Simonstown via Wynberg"
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

### Legs

A leg is a section of an itinerary carried out by a passenger on one mode of transit (including walking) from some departure point to an arrival point.

#### Types of legs {#types-of-legs}

The Transit API currently supports two types of legs, each with a different response model.

A _Walking_ leg is one which the passenger is to travel by foot from one waypoint to another.  Walking legs are usually accompanied with directions.

A _Transit_ leg is one which uses a public transportation service based on scheduled or absolute frequency-based stop times.

#### Leg response model {#leg-model}

| Attribute | Type | Description |
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

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| designation | string | If available, an identifier for this vehicle as defined by the transit agency, or some other designation. |
| direction | string | If available, the direction of the vehicle, for example, "Northbound" or "Clockwise" |
| headsign | string | If available, identifying information (such as destination) displayed on the vehicle. |

#### Waypoint response model

A waypoint is a stopping point along an itinerary. It has either an arrival date and time or a departure date and time, or both.

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| arrivalTime | [DateTime](#datetime) | The arrival date and time at this point of a leg. |
| departureTime | [DateTime](#datetime) | The departure date and time from this point of a leg. |
| stop | [Stop](#stop-model) | **[**[Excludable](#excludable)**]** The stop of the waypoint. This can be returned in either Walking or Transit legs. |
| location | [Location](#location-model) | The location of the waypoint if it is not a stop. This can be returned in only Walking legs. |

#### Direction response model

If available, the directions to follow in order to get from the start to the end of a leg.

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| instruction | string | The instruction to follow. |
| distance | [Distance](#distance) | The distance to travel after the instruction has been followed. |

#### Location response model

| Attribute | Type | Description |
| :--------- | :--- | :---- |
| address | string | The reverse geocoded address of the point. |
| geometry | [GeoJSON](#geojson) Point | The geographic point of the location. |

### Fares {#fares}

A fare is the cost incurred by a commuter when using a transit service.  Essentially, it is the price associated with a journey's itinerary for a particular fare product or set of fare products.

#### Fare response model

| Attribute | Type | Description |
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

| Attribute | Type | Description |
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
