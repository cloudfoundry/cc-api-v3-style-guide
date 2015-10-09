
#Cloud Controller API v3 Style Guide (Proposal)

## Table of contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Overview](#overview)
  - [Guiding Principles](#guiding-principles)
  - [API Technologies](#api-technologies)
  - [API Design Inspirations](#api-design-inspirations)
  - [Overview Example](#overview-example)
- [Requests](#requests)
  - [URL structure](#url-structure)
  - [GET](#get)
    - [Examples](#examples)
      - [Responses (Resource)](#responses-resource)
      - [Responses (Collection)](#responses-collection)
  - [POST](#post)
    - [Examples](#examples-1)
      - [Responses](#responses)
  - [PUT](#put)
    - [Examples](#examples-2)
      - [Responses](#responses-1)
  - [PATCH](#patch)
    - [Examples](#examples-3)
      - [Responses](#responses-2)
  - [DELETE](#delete)
    - [Examples](#examples-4)
      - [Responses](#responses-3)
- [Resources](#resources)
  - [Example](#example)
- [Links](#links)
  - [Example](#example-1)
- [Collections](#collections)
  - [Example](#example-2)
- [Pagination](#pagination)
  - [Example](#example-3)
- [Actions](#actions)
  - [Example](#example-4)
- [Query Parameters](#query-parameters)
    - [Example](#example-5)
- [Field Names](#field-names)
    - [Example](#example-6)
- [Filtering](#filtering)
    - [Example multiple value request](#example-multiple-value-request)
    - [Example single value request](#example-single-value-request)
    - [Example combined filters](#example-combined-filters)
- [Errors](#errors)
  - [Status Codes](#status-codes)
  - [Issues with v2 error format](#issues-with-v2-error-format)
  - [Proposal](#proposal)
- [Response Codes](#response-codes)
  - [Successful Requests](#successful-requests)
  - [Redirection](#redirection)
  - [Client Errors](#client-errors)
  - [Server Errors](#server-errors)
- [Relationships](#relationships)
  - [Currently](#currently)
  - [Proposal](#proposal-1)
- [Nested Resources](#nested-resources)
- [Including Related Resources](#including-related-resources)
  - [Proposal](#proposal-2)
  - [Pagination of Related Resources](#pagination-of-related-resources)
- [Requesting Partial Resources](#requesting-partial-resources)
  - [Proposal For Sub-Resources](#proposal-for-sub-resources)
- [Asynchronicity](#asynchronicity)
  - [Currently](#currently-1)
    - [Grievances](#grievances)
  - [Proposal](#proposal-3)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
## Overview
This document serves as a style guide for the Cloud Controller API. It is intended to act as a repository for patterns and best practices when designing and developing new API endpoints.

This is a living document; It will change over time as we learn more about our users and develop features.

### Guiding Principles

* **Consistency**: Understanding how to interact with one resource must inform how to interact with any resource.
* **Discoverability**: API responses must guide users without the need for external documentation.
* **Opinionatedness**: There must be one clear way to do something.

### API Technologies
* **HTTP:** All API requests must be made over HTTP.
* **JSON:** All API response bodies will be JSON objects.

### API Design Inspirations
* **REST:**  https://en.wikipedia.org/wiki/Representational_state_transfer
* **JSON API:** http://jsonapi.org/
* **HAL:** http://stateless.co/hal_specification.html

### Overview Example
Here is an example request to get apps filtered by query parameters:

```
GET /v3/apps?names=dora,kailan&order_by=+created_at&page=1&per_page=2
```

Note that the examples in the style guide to not encode query strings. This is to make the examples more human-readable. In actuality, all requests and responses must contain correctly encoded characters. For more information see [Query Parameters](#query-parameters).

Here is the respective response body:

```json
{
  "pagination": {
    "total_results": 3,
    "first": {
      "href": "/v3/apps?names=dora,kailan&order_by=+created_at&page=1&per_page=2"
    },
    "last": {
      "href": "/v3/apps?names=dora,kailan&order_by=+created_at&page=2&per_page=2"
    },
    "next": {
      "href": "/v3/apps?names=dora,kailan&order_by=+created_at&page=2&per_page=2"
    },
    "previous": null
  },
  "resources": [
    {
      "guid": "guid-00133700-abcd-1234-9000-3f70a011bc28",
      "name": "dora",
      "desired_state": "STOPPED",
      "total_desired_instances": 0,
      "buildpack": null,
      "created_at": "2015-08-06T00:36:20Z",
      "updated_at": null,
      "environment_variables": {

      },
      "links": {
        "self": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28"
        },
        "space": {
          "href": "/v2/spaces/ab09cd29-9420-f021-g20d-123431420768"
        },
        "processes": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28/processes"
        },
        "routes": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28/routes"
        },
        "packages": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28/packages"
        },
        "droplets": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28/droplets"
        },
        "start": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28/start",
          "method": "PUT"
        },
        "stop": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28/stop",
          "method": "PUT"
        },
        "assign_current_droplet": {
          "href": "/v3/apps/guid-00133700-abcd-1234-9000-3f70a011bc28/current_droplet",
          "method": "PUT"
        }
      }
    },
    {
      "guid": "guid-bd7369a8-deed-ff1a-2315-77410293a922",
      "name": "kailan",
      "desired_state": "STOPPED",
      "total_desired_instances": 0,
      "buildpack": null,
      "created_at": "2015-08-07T00:40:52Z",
      "updated_at": null,
      "environment_variables": {

      },
      "links": {
        "self": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922"
        },
        "space": {
          "href": "/v2/spaces/881029ab-4edd-4920-af10-6386967209d1"
        },
        "processes": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922/processes"
        },
        "routes": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922/routes"
        },
        "packages": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922/packages"
        },
        "droplets": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922/droplets"
        },
        "start": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922/start",
          "method": "PUT"
        },
        "stop": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922/stop",
          "method": "PUT"
        },
        "assign_current_droplet": {
          "href": "/v3/apps/guid-bd7369a8-deed-ff1a-2315-77410293a922/current_droplet",
          "method": "PUT"
        }
      }
    }
  ]
}
```

## Requests

### URL structure

All endpoints must be prefixed with /v3/.
Pattern: `/v3/...`  

Collections of resources are referenced by their resource name (plural)  
Pattern: `/v3/:resource name`  
Example: `/v3/apps`  

Individual resources are referenced their resource name (plural) followed by the guid  
Pattern: `/v3/:resource name/:guid`  
Example:  `/v3/apps/25fe21b8-8de2-40d0-93b0-c819101d1a11`  

### GET
Used to retrieve a single resource or a list of resources.

* GET requests **may** include query parameters
* GET requests **must NOT** include a request body

#### Examples
Show individual resource:
```
GET /v3/apps/:guid
```

##### Responses (Resource)
|Scenario|Code|Body|
|---|---|---|
| Authorized User | 200 | Resource |
| Unauthorized User | 404 | Error |

List collection of resources:
```
GET /v3/apps/
```
##### Responses (Collection)
|Scenario|Code|Body|
|---|---|---|
| User With Complete Visibility | 200 | List of All Resources |
| User With Partial Visibility | 200 | List of Visible Resources |
| User With No Visibility | 200 | Empty List |

### POST
Used to create a resource.

* POST requests **must NOT** include query parameters
* POST requests **may** include a request body

#### Examples

Create a resource:

```
POST /v3/apps/

{
  name: "cool_app",
  space_guid: "123guid"
}
```

##### Responses
|Scenario|Code(s)|Body|
|---|---|---|
| Authorized User (sync) | 201 | Created Resource |
| Authorized User [(async)](#asynchronicity) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 403 | Error |

### PUT
Used to trigger an [action](#actions). To update a resource, use [PATCH](#patch)

* PUT requests **must NOT** include query parameters
* PUT requests **may** include a request body

#### Examples
Trigger an action:

```
PUT /v3/apps/:guid/processes/web/scale

{
  "instances": 100,
  "memory_in_mb": 2048
}
```

##### Responses
|Scenario|Code(s)|Body|
|---|---|---|
| Authorized User (sync) | 200 | Empty |
| Authorized User [(async)](#asynchronicity) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 404 | Error |

### PATCH
Used to update a portion of a resource.

* PATCH requests **must NOT** include query parameters
* PATCH requests **may** include a request body
* PATCH operations **MUST** apply all requested updates or none.

#### Examples
Partially update a resource:

```
PATCH /v3/apps/:guid

{
  "name": "new_app_name"
}
```

##### Responses
|Scenario|Code(s)|Body|
|---|---|---|
| Authorized User (sync) | 200 | Updated Resource |
| Authorized User [(async)](#asynchronicity) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 404 | Error |

### DELETE
Used to delete a resource.

* DELETE requests **may** include query parameters
* DELETE requests **must NOT** include a request body

#### Examples
Delete a resource:

```
DELETE /v3/apps/:guid
```

##### Responses
|Scenario|Code(s)|Body|
|---|---|---|
| Authorized User (sync) | 204 | N/A |
| Authorized User [(async)](#asynchronicity) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 404 | Error |
| Missing Resource | 404 | Error |

## Resources

A resource represents an individual object within the system, such as an app or a service.  It is represented as a JSON object.  

A resource **MUST** contain the following fields:

* `guid`: a unique identifier for the resource
* `created_at`: an ISO8601 compatible date and time that the resource was created
* `updated_at`: an ISO8601 compatible date and time that the resource was last updated

A resource **may** contain additional fields which are the attributes describing the resource.

A resource **MUST** contain a `links` field containing a [links](#links) object, which is used to provide URLs to relationships and actions for the resource.

A resource **MUST** include a `self` link object in the `links` field.

### Example

```json
{
  "guid": "a-b-c",
  "created_at": "2015-07-06T23:22:56Z",
  "updated_at": "2015-07-08T23:22:56Z",

  "name": "dora",
  "description": "an example app",

  "links": {
    "self": {
      "href": "/v3/apps/a-b-c"
    }
  }
}
```

## Links

Links provide URLs to relationships and actions for a resource.  Links are represented as a JSON object.

Each member of a links object is a "link".  
A link **MUST** be a JSON object.
A link **MUST** contain a `href` field, which is a string containing the link's relative URL.
A link **may** contain a `method` field, which is a string containing the HTTP verb that must be used to follow the URL.  If the `method` field is not included then the link **MUST** be available using GET.

### Example

```json
"links": {
  "self": {
    "href": "/v3/apps/a-b-c"
  },
  "space": {
    "href": "/v3/apps/a-b-c/relationships/space"
  },
  "processes": {
	"href": "/v3/apps/a-b-c/relationships/processes"
  },
  "start": {
    "href": "/v3/apps/a-b-c/start",
    "method": "PUT"
  }
}
```

Note that the key is `links` to reduce the likelihood of collisions with a hypothetical resource field named `links`.

## Collections
A collection is a list of multiple Resources.  A collection is represented as a JSON object.

A collection **MUST** contain a `resources` field.  The resources field is an array containing multiple [Resources](#resources).

A collection **MUST** contain a `pagination` field containing a [pagination](#pagination) object.

### Example

```json
{
  "resources": [
    {
      "guid": "a-b-c",
      "created_at": "2015-07-06T23:22:56Z",
      "updated_at": "2015-07-08T23:22:56Z",

      "links": {
        "self": {
          "href": "/v3/apps/a-b-c"
        }
      }
    },
    {
      "guid": "d-e-f",
      "created_at": "2015-07-06T23:22:56Z",
      "updated_at": "2015-07-08T23:22:56Z",

      "links": {
        "self": {
          "href": "/v3/apps/d-e-f"
        }
      }
    }
  ],
  "pagination": {
    "total_results": 2,
    "first": {
      "href": "/v3/apps?page=1&per_page=10"
    },
    "last": {
      "href": "/v3/apps?page=1&per_page=10"
    },
    "next": null,
    "previous": null
  }
}
```

## Pagination

Pagination **may** be used by [Collections](#collections) to limit the number of resources returned at a time.  Pagination is requested by a client through the use of query parameters. Pagination is represented as a JSON object.

Pagination **MUST** include a `total_results` field with an integer value of the total number of records in the collection.

Pagination **MUST** include the following fields for pagination links:

* `first`: the first page of resources
* `last`: the last page of resources
* `previous`: the previous page of resources
* `next`: the next page of resources

Pagination links **may** be `null`.  For example, if the page currently being displayed is the first page, then  `previous` link will be null.

When pagination links contain a URL, they **MUST** be a JSON object with a field named `href` containing a string with the URL for the next page.

The URL **MUST** include all query parameters required to maintain consistency with the original pagination request.  For example, if the client requested for the collection to be returned in a specific order direction via a query parameter, then the pagination links must include the proper query parameter to maintain the requested direction.

The following query parameters **MUST** be used for pagination:

* `page`: the page number of resources to return (default: 1)
* `per_page`: the number of resources to return in a paginated collection request (default: 50)
* `order_by`: a field on the resource to order the collection by; each collection may choose a subset of fields that it can be sorted by 

When collections are ordered by a subset of fields, each field **MAY** be prepended by "+" or "-" to indicate ascending or descending order direction respectively. If the field is not prepended, the ordering will default to ascending.

If there are additional pagination query parameters, the parameters **MUST** have names that conform to the acceptable [query parameter](#query-parameters) names.

### Example

```json
"pagination:" {
  "total_results": 20,
  "first": {
    "href": "/v3/apps?order_by=-created_at&page=1&per_page=10"
  },
  "last": {
    "href": "/v3/apps?order_by=-created_at&page=2&per_page=10"
  },
  "next": {
    "href": "/v3/apps?order_by=-created_at&page=2&per_page=10"
  },
  "previous": null
}
```

## Actions

Actions are API requests that are expected to initiate change within the Cloud Foundry runtime.  This is differentiated from requests which update a record, but require additional updates — such as restarting an app — to cause changes to a resource to take affect.  

Actions **MUST** use use PUT as their HTTP verb.

Actions **may** accept a request body.

Actions **MUST** be listed in the `links` for the related resource.

### Example
 `PUT /v3/apps/:guid/start`

## Query Parameters

Query Parameters **MUST** include **ONLY** the following characters:

* a-z (lowercase only)
* _ (underscore)

Query parameters that accept multiple values **MUST** be pluralized.

If any request receives a query parameter it does not understand, the response **MUST** be a `400 Bad Request`.

All query parameters **MUST** be properly [url-encoded](https://en.wikipedia.org/wiki/Percent-encoding#Current_standard). If a query parameter value includes the comma (`,`) character, the comma **MUST** be double encoded. Note that for readability purposes, the examples throughout this document do not show encoded query strings.

####Example
Single value:
`GET /v3/apps?names=firstname`

Multiple values:
 `GET /v3/apps?names=firstname,secondname`
 
Single value with comma:
 `GET /v3/apps?names=comma%2Cname`

## Field Names

Resource Fields **MUST** include **ONLY** the following characters:

* a-z (lowercase only)
* _ (underscore)

Resource fields that accept multiple values **MUST** be pluralized.

####Example:
```json
{
  "resources": [
    {
      "guid": "guid-1",
      "environment_variables": {
        "animal": "fish",
        "color": "teal"
      }
    }
  ]
}
```

## Filtering

Filtering is the use of query parameters to return a subset of resources within a [Collection](#collections).

Filter query parameters **MUST** have names that conform to the acceptable [query parameter](#query-parameters) names.

Filters **MUST** allow a client to request resources matching multiple values of by accepting the filter as an array.

Filter parameters **MUST** be able to be combined with other filters on the same collection.

#### Example multiple value request

`http://api.server.com/v3/apps?names=first_name,second_name`

#### Example single value request

`http://api.server.com/v3/apps?names=the_name`

#### Example combined filters

`http://api.server.com/v3/apps?names=the_name&space_guids=d-e-f`

## Errors

### Status Codes

The HTTP status code returned for errors **MUST** be included in the documented [status codes](#response-codes).

### Issues with v2 error format
Currently looks like v2.

```
{
   "code": 100001,
   "description": "The app is invalid: Invalid app state provided",
   "error_code": "CF-AppInvalid"
}
```

* Multiple validation errors are difficult to show to a client
* `error_code` is often ambiguous
* `code` does not have a clear mapping to what it means or how to resolve it


### Proposal

This proposal includes `code` which would be an internal unique identifier of a class of error.  This could be used for support scripts for CF operators.  The method for maintaining a list of these codes and their meanings would need to be determined  

```json
{
  "status": 400,
  "code": 123455,
  "type": "Invalid Request",
  "description": "The request body is not valid.",
  "errors": [
    {
      "resource": "app",
      "messages": [
	    "Names must be unique",
	    "Names may not contain numbers"
      ]
    },
    {
      "resource": "lunch",
      "messages": [
        "Potatoes cannot be perfectly round"
      ]
    }
  ]
}
```

## Response Codes

### Successful Requests

|Status Code|Description|Verbs|
|---|---|---|
|200 OK|This status **MUST** be returned for synchronous requests that complete successfully and have a response body. This must only be used if there is not a more appropriate 2XX response code. |GET, PATCH, PUT|
|201 Created|This status **MUST** be returned for synchronous requests that result in the creation of a new resource.|POST|
|202 Accepted|This status **MUST** be returned for requests that have been successfully accepted and will be asynchronously completed at a later time. See more in the [async](#asynchronicity) section. |POST,PATCH,PUT,DELETE|
|204 No Content|This status **MUST** be returned for synchronous requests that complete successfully and have no response body.|DELETE


### Redirection

|Status Code|Description|Verbs|
|---|---|---|
|302 Found| This status **MUST** be returned when the cloud controller redirects to another location. Example: Downloading a package from an external blob store.  |GET|
|303 See Other| This status **MUST** be returned when an async job finishes. It must include a location header containing the resource link. See more in the [async](#asynchronicity) section. |GET|


### Client Errors

|Status Code|Description|Verbs|
|---|---|---|
|400 Bad Request|This status **MUST** be returned for requests that provide malformed or invalid data. Examples: invalid JSON, unexpected query parameters or request fields.|GET, PATCH, POST, PUT, DELETE|
|401 Unauthenticated|This status **MUST** be returned if the requested resource requires an authenticated user but there is no OAuth token provided, or the OAuth token provided is invalid.|GET, POST, PATCH, DELETE, PUT|
|403 Forbidden|This status **MUST** be returned if the request cannot be performed by the user due to lack of permissions. Example: User with read-only permissions to a resource tries to update it. |POST, PATCH, DELETE, PUT|
|404 Not Found|This status **MUST** be returned if the requested resource does not exist or if the user requesting the resource has insufficient permissions to view the resource.|GET, POST, PATCH, PUT, DELETE|
|422 Unprocessable Entity|This status **MUST** be returned if the request is semantically valid, but performing the requested operation would result in a invalid state. Example: Attempting to start an app without assigning a droplet.|POST, PATCH, PUT|

### Server Errors

|Status Code|Description
|---|---|---|
|500 Internal Server Error|This status **MUST** be returned when an unexpected error occurs.
|502 Bad Gateway|This status **MUST** be returned when an upstream service failure causes a request to fail. Example: Being unable to reach requested service broker.

## Relationships

### Currently

Associations are sometimes created by setting a `relationship_guid` field on one of the resources.

* For example, `POST /v3/apps { "space_guid": "abc" }` creates an app associated to a space.

Other times associations are added via PUT requests to a nested resource.

* For example, `PUT /v3/apps/guid/routes { route_guid: the guid }` will add a route association to an app.

An association can then be deleted via `DELETE /v3/apps/guid/routes { route_guid: the guid }`.

This currently feels a little clunky for several reasons:
* Associations are created in different ways depending on whether we consider it a required relationship or not
* The nested routes make it appear that the request is affecting a routes resource, but it is really a relationship that is being created or deleted
* Associations become a top-level concern on the resource such that the way relationships exist for that resource are difficult to modify without causing breaking changes or leaving unused fields

### Proposal

Add a relationship object to resources similar to the jsonapi spec.

Creating an app could change to allow creating relationships at create time by including the relationships object.  Some relationships could be required at creation such as space in the app case.

```
POST /v3/apps
{
  "name": "blah",
  "relationships": {
   "space": {"guid": "1234"},
   "routes": [
     {"guid": "2345"},
     {"guid": "3456"}
   ],
  }
}
```

Modifying relationships later could be done through a nested relationship resource

Setting a \*-to-one relationship:
```
PATCH /v3/apps/guid/relationships/space
{
  "data": {"guid": "some-guid"}
}
```

Clearing a \*-to-one relationship:
```
PATCH /v3/apps/guid/relationships/space
{
  "data": null
}
```

Adding to a \*-to-many relationship (not an overwrite/replace):
```
POST /v3/apps/guid/relationships/routes
{
  "data": [{"guid": "asdf" }, {"guid": "gfds" }]
}
```

Replacing all items in a \*-to-many relationship:
```
PATCH /v3/apps/guid/relationships/routes
{
  "data": [{"guid": "some-guid" }, {"guid": "whatever" }]
}
```

Clearing all items in a \*-to-many relationship:
```
PATCH /v3/apps/guid/relationships/routes
{
  "data": []
}
```

Removing some items from a \*-to-many relationship:
```
DELETE /v3/apps/guid/relationships/routes
{
  "data": [{"guid": "asdf" }, {"guid": "hgfg" }]
}
```

## Nested Resources

Nested resources can optionally be accessed through their parent resource.
```
GET /v3/apps/:guid/droplets
```
This will be equivalent to
```
GET /v3/droplets?app_guids=:app_guid
```

These end points are optional and may not exist for all resources. Good opportunities for creating them:
* The resource commonly or exclusively lists within the context of the parent resource. E.g. droplets only exist inside apps.

## Including Related Resources

This is a mechanism for including multiple related resources in a single response.

A use case for this would be to display an HTML page that includes information about both an app and its space and would like to gather this information in one HTTP request.

### Proposal

Included resources are in-lined under their pluralized resource name in an `included` object on the primary resource.  If a resource within a resource — `resource.otherresource` — is requested, it is added in the top level `included` object and not repeated. Associations between included resources and requested resources must be shown in the 'relationships' section for the requested resource. Duplicate included resources are not repeated

```
GET /v3/apps/?include=space

{
  "pagination": {
    "total_results": 3,
    "first": {
      "href": "/v3/apps?order_by=-created_at&page=1&per_page=10"
    },
    "last": {
      "href": "/v3/apps?order_by=-created_at&page=1&per_page=10"
    },
    "next": {
      "href": "/v3/apps?order_by=-created_at&page=1&per_page=10"
    },
    "previous": null
  },
  "resources": [
    {
      "guid": "643e170c-42cf-4704-840f-4fc22e95f3c5",
      "name": "my_app3",
      "desired_state": "STOPPED",
      "total_desired_instances": 0,
      "buildpack": "name-2009",
      "created_at": "1970-01-01T00:00:03Z",
      "updated_at": null,
      "environment_variables": {
        "magic": "beautiful"
      },
      "relationships": {
        "space": {
          "guid": "801d80bc-0314-48c0-bd7a-20cbb230c922"
        }    
      },
      "links": {
        "self": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5"
        },
        "space": {
          "href": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922"
        },
        "processes": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5/processes"
        },
        "routes": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5/routes"
        },
        "packages": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5/packages"
        },
        "droplets": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5/droplets"
        },
        "start": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5/start",
          "method": "PUT"
        },
        "stop": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5/stop",
          "method": "PUT"
        },
        "assign_current_droplet": {
          "href": "/v3/apps/643e170c-42cf-4704-840f-4fc22e95f3c5/current_droplet",
          "method": "PUT"
        }
      }
    },
    {
      "guid": "3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef",
      "name": "my_app2",
      "desired_state": "STOPPED",
      "total_desired_instances": 0,
      "buildpack": null,
      "created_at": "1970-01-01T00:00:02Z",
      "updated_at": null,
      "environment_variables": {

      },
      "relationships": {
        "space": {
          "guid": "8e63e218-9c12-460f-af57-f9c320657dad"
        }    
      },
      "links": {
        "self": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef"
        },
        "space": {
          "href": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad"
        },
        "processes": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef/processes"
        },
        "routes": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef/routes"
        },
        "packages": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef/packages"
        },
        "droplets": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef/droplets"
        },
        "start": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef/start",
          "method": "PUT"
        },
        "stop": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef/stop",
          "method": "PUT"
        },
        "assign_current_droplet": {
          "href": "/v3/apps/3b310427-c9f0-4e7c-ab66-5f5d9a86a4ef/current_droplet",
          "method": "PUT"
        }
      }
    },
    {
      "guid": "8c943e87-c00c-4d0b-8583-d5c49111a5ec",
      "name": "my_app2",
      "desired_state": "STOPPED",
      "total_desired_instances": 0,
      "buildpack": null,
      "created_at": "1970-01-01T00:00:02Z",
      "updated_at": null,
      "environment_variables": {

      },
      "relationships": {
        "space": {
          "guid": "8e63e218-9c12-460f-af57-f9c320657dad"
        }    
      },
      "links": {
        "self": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec"
        },
        "space": {
          "href": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad"
        },
        "processes": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec/processes"
        },
        "routes": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec/routes"
        },
        "packages": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec/packages"
        },
        "droplets": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec/droplets"
        },
        "start": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec/start",
          "method": "PUT"
        },
        "stop": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec/stop",
          "method": "PUT"
        },
        "assign_current_droplet": {
          "href": "/v3/apps/8c943e87-c00c-4d0b-8583-d5c49111a5ec/current_droplet",
          "method": "PUT"
        }
      }
    }
  ],
  "included": {
    "spaces": [
      {
        "metadata": {
          "guid": "8e63e218-9c12-460f-af57-f9c320657dad",
          "url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad",
          "created_at": "2015-10-07T00:35:20Z",
          "updated_at": null
        },
        "entity": {
          "name": "name-76",
          "organization_guid": "b35e1b19-8031-4278-86af-45dd74c58271",
          "space_quota_definition_guid": null,
          "allow_ssh": true,
          "organization_url": "/v2/organizations/b35e1b19-8031-4278-86af-45dd74c58271",
          "developers_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/developers",
          "managers_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/managers",
          "auditors_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/auditors",
          "apps_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/apps",
          "routes_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/routes",
          "domains_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/domains",
          "service_instances_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/service_instances",
          "app_events_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/app_events",
          "events_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/events",
          "security_groups_url": "/v2/spaces/8e63e218-9c12-460f-af57-f9c320657dad/security_groups" 
        }
      },
      {
        "metadata": {
          "guid": "801d80bc-0314-48c0-bd7a-20cbb230c922",
          "url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922",
          "created_at": "2015-10-07T00:35:20Z",
          "updated_at": null
        },
        "entity": {
          "name": "name-77",
          "organization_guid": "b35e1b19-8031-4278-86af-45dd74c58271",
          "space_quota_definition_guid": null,
          "allow_ssh": true,
          "organization_url": "/v2/organizations/b35e1b19-8031-4278-86af-45dd74c58271",
          "developers_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/developers",
          "managers_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/managers",
          "auditors_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/auditors",
          "apps_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/apps",
          "routes_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/routes",
          "domains_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/domains",
          "service_instances_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/service_instances",
          "app_events_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/app_events",
          "events_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/events",
          "security_groups_url": "/v2/spaces/801d80bc-0314-48c0-bd7a-20cbb230c922/security_groups" 
        }
      }
    ]
  }
}

```
###Pagination of Related Resources
Related resources are paginated in a similar style to how normal responses are paginated.  
The pagination data **may** be excluded if all results are included in the response.

GET /v3/apps/:guid?include=routes

```json
{
  "guid": "guid",
  
  "included": {
    "routes": {
      "resources": [
        {"guid": "1"},
        {"guid": "2"}
      ],
      "pagination": {
        "total_results": 20,
        "first": {
          "href": "/v3/apps/:guid/routes?order_by=-created_at&page=1&per_page=2"
        },
        "last": {
          "href": "/v3/apps/:guid/routes?order_by=-created_at&page=10&per_page=2"
        },
        "next": {
          "href": "/v3/apps/:guid/routes?order_by=-created_at&page=2&per_page=2"
        },
        "previous": null
      }
    }
  }
}
```
## Requesting Partial Resources

```
GET /apps?fields=guid,name

{
  "guid": "some-guid",
  "name": "Zach"
}
```

### Proposal For Sub-Resources

If we want to be able to filter the fields of subresources, we could do something like:
```
GET /apps?fields[apps]=guid,name&fields[droplets]=buildpack

{
  "guid": "some-guid",
  "name": "Zach",
  "included": {
    "droplet": {
      "buildpack": "ruby"
    }
  }
}
```


## Asynchronicity

### Currently

Via Mark: The original impetus for the `async` flag was:

1. The CC API could not make a major version bump, so it needed to be an additive change. The hope was to move CC to be fully async-by-default
2. Long running requests could be triggered multiple times and bring down the CC
3. Long running requests were getting clipped by network timeouts

The team later learned that not all endpoints need to be async (/v2/info for example). The overhead of background workers/ jobs is not worth it.

* By default, `async=true` causes the CLI to block and poll indefinitely waiting for job to complete (CC has to enforce a timeout)
* By default, `accepts_incomplete=true` causes the CLI to exit once a poll-able resource is returned, and CC will poll for up to a week. CC ignores `async=true`

Requests including the `async=true` query parameter will resolve asynchronously. Instead of returning whatever resource is requested, the CC will return a job.

#### Grievances
 * The CC returns a resource different than what is requested
 * Not all endpoints support `async=true`. There doesn't seem to be any indication of what supports it.
 * It doesn't play nice with `accepts_incomplete=true` (service-specific). The CC will currently ignore `async=true` if `accepts_incomplete` is included, even if the broker does not support async operations. This renders `async=true` useless for most broker operations.
 * Not all endpoints that support it need it. Sending `async=true` in many cases will slow down the request, since there is the worker/polling overhead. A 1 second synchronous operation doesn't need to happen out of band with a 5 second polling interval.

### Proposal
Remove both flags. Instead, endpoints are responsible for behaving either asynchronously (return 202) or synchronously (don't return 202).

For async endpoints:
`POST /v3/resource`

The CC will return a 202 with a location header pointing to the job. Depending on the resource, it may also return a skeletal body containing the partial resource.
```
202 Accepted
Location: /v3/jobs/123

{
  "skeletor": "YOU! You will no longer stand between me and my destiny!"
}
```

Before the job has completed,  GET requests made to the job endpoint will return 200 with information about the status of the job.
```
GET /v3/jobs/123
```
```
200 OK

{
  "status": "in progress"
}
```

When the job has completed, GET request made to the job endpoint will return 303 and a location header to the resource (assuming it still exists).
```
GET /v3/jobs/123
```
```
303 See Other
Location: /v3/resource/:guid

{}
```

Note that for asynchronous deletes, the redirect location will be to a no-longer-existent resource.
