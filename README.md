
#Cloud Controller API v3 Styleguide (Proposal)

## Table of contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Overview](#overview)
  - [API Technologies](#api-technologies)
  - [API Design Inspirations](#api-design-inspirations)
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
    - [Examples:](#examples)
- [Filtering](#filtering)
    - [Example multiple value request](#example-multiple-value-request)
    - [Example single value request](#example-single-value-request)
    - [Example combined filters](#example-combined-filters)
- [Errors](#errors)
  - [Status Codes](#status-codes)
  - [Issues with v2 error format](#issues-with-v2-error-format)
  - [Proposal A](#proposal-a)
  - [Proposal B](#proposal-b)
  - [Proposal C](#proposal-c)
- [Response Codes](#response-codes)
  - [Successful Requests](#successful-requests)
  - [Redirection](#redirection)
  - [Client Errors](#client-errors)
  - [Server Errors](#server-errors)
- [Relationships](#relationships)
  - [Currently](#currently)
  - [Proposal A](#proposal-a-1)
  - [Proposal A1](#proposal-a1)
- [Nested Resources](#nested-resources)
- [Including Related Resources](#including-related-resources)
  - [Proposal A](#proposal-a-2)
  - [Proposal B](#proposal-b-1)
  - [Proposal C](#proposal-c-1)
- [Requesting Partial Resources](#requesting-partial-resources)
  - [Proposal For Sub-Resources](#proposal-for-sub-resources)
- [Asynchronicity](#asynchronicity)
  - [Currently](#currently-1)
    - [Grievances](#grievances)
  - [Proposal](#proposal)
  - [Async vs Accepts Incomplete](#async-vs-accepts-incomplete)
- [Style Guide Ideas (Style Guide Style Guide)](#style-guide-ideas-style-guide-style-guide)
  - [Interesting Ideas](#interesting-ideas)
  - [Questions](#questions)
  - [Areas of Contention](#areas-of-contention)
  - [Known SG Defects](#known-sg-defects)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
## Overview
This document serves as a styleguide for the Cloud Controller API. It is intended to act as a repository for patterns and best practices when designing and developing new API endpoints.

### API Technologies
* **HTTP:** All API requests shall be made over HTTP.
* **JSON:** All API response bodies will be JSON objects.

### API Design Inspirations
* **REST:**  https://en.wikipedia.org/wiki/Representational_state_transfer
* **JSON API:** http://jsonapi.org/
* **HAL:** http://stateless.co/hal_specification.html

List of terms:

 - Resource
 - Collection: List of resources
 - REST actions:
	 - Create
	 - Show
	 - List
	 - Update
	 - Delete

## Requests

### URL structure

All endpoints shall be prefixed with an api version
Pattern: ` /v<number>/...`
Example: `/v3/...`

Collections of resources are referenced by their resource name (plural)
Pattern: `/v3/<resource name>`
Example: `/v3/apps`

Individual resources are referenced their resource name (plural) followed by the guid
Pattern: `/v3/<resource name>/<guid>`
Example:  `/v3/apps/25fe21b8-8de2-40d0-93b0-c819101d1a11`

### GET
Used to retrieve a single resource or a list of resources.

* GET requests **may** include query parameters
* GET requests **shall NOT** include a request body

#### Examples
Show individual resource:
```
GET /v3/resource/:guid
```

##### Responses (Resource)
|Scenario|Code|Body|
|-|-|-|
| Authorized User | 200 | Resource |
| Unauthorized User | 404 | Error |

List collection of resources:
```
GET /v3/resource/:guid
```
##### Responses (Collection)
|Scenario|Code|Body|
|-|-|-|
| User With Complete Visibility | 200 | List of All Resources |
| User With Partial Visibility | 200 | List of Visible Resources |
| User With No Visibility | 200 | Empty List |

### POST
Used to create a resource or trigger an action on a resource.

* POST requests **shall NOT** include query parameters
* POST requests **may** include a request body

#### Examples

Create a resource:

```
POST /v3/resource/:guid

{
  name: "name",
  additional_parameter: "value"
}
```

##### Responses
|Scenario|Code(s)|Body|
|-|-|-|
| Authorized User (sync) | 201 | Created Resource |
| Authorized User (async) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 403 | Error |

### PUT
Used to request that an action is performed.  Sometimes PATCH will update a resource and further actions such as starting or stopping an app are required to make the change take affect within the system.  PUT is used to indicate that an change will begin taking place in the system immediately after the request is accepted.  See [Actions](#actions) for more information.

* PUT requests **shall NOT** include query parameters
* PUT requests **may** include a request body

#### Examples
Update a resource:

```
PUT /v3/resource/:guid

{
  name: "new name",
  additional_parameter: "new value"
}
```

##### Responses
|Scenario|Code(s)|Body|
|-|-|-|
| Authorized User (sync) | 201 | Updated Resource |
| Authorized User (async) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 404 | Error |

### PATCH
Used to update a portion of a resource.

* PATCH requests **shall NOT** include query parameters
* PATCH requests **may** include a request body

#### Examples
Partially update a resource:

```
PATCH /v3/resource/:guid

{
  additional_parameter: "new value"
}
```

##### Responses
|Scenario|Code(s)|Body|
|-|-|-|
| Authorized User (sync) | 201 | Updated Resource |
| Authorized User (async) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 404 | Error |

### DELETE
Used to delete a resource.

* DELETE requests **may** include query parameters
* DELETE requests **shall NOT** include a request body

#### Examples
Delete a resource:

```
DELETE /v3/resource/:guid
```

##### Responses
|Scenario|Code(s)|Body|
|-|-|-|
| Authorized User (sync) | 204 | N/A |
| Authorized User (async) | 202 | Empty w/ Location Header -> Job |
| Read-only User | 403 | Error |
| Unauthorized User | 404 | Error |
| Missing Resource | 404 | Error |

## Resources

A resource represents an individual object within the system, such as an app or a service.  It is represented as a JSON object.  

A resource **MUST** contain the following fields:

* `guid:` a unique identifier for the resource
* `created_at:` an ISO8601 compatible date and time that the resource was created
* `updated_at:` an ISO8601 compatible date and time that the resource was last updated

A resource **may** contain additional fields which are the attributes describing the resource.

A resource **should** contain a `_links:` field containing a [links](#links) object, which is used to provide URLs to relationships and actions for the resource.

A resource **should** include a `self:` link object in the `_links:` field.

### Example

```json
{
  "guid": "a-b-c",
  "created_at": "2015-07-06T23:22:56Z",
  "updated_at": "2015-07-08T23:22:56Z",

  "name": "resource1",
  "description": "an example resource",

  "_links": {
    "self": {
      "href": "/v3/resources/a-b-c"
    }
  }
}
```

## Links

Links provide URLs to relationships and actions for a resource.  Links are represented as a JSON object.

Each member of a links object is a "link".  
A link **MUST** be a JSON object.
A link **MUST** contain a `href:` field, which is a string containing the link's relative URL.
A link **may** contain a `method:` field, which is a string containing the HTTP verb that must be used to follow the URL.

### Example

```json
"_links": {
  "self": {
    "href": "/v3/apps/a-b-c"
  },
  "space": {
    "href": "/v3/apps/d-e-f/relationships/space"
  },
  "processes": {
	"href": "/v3/apps/d-e-f/relationships/processes"
  },
  "start": {
    "href": "/v3/apps/a-b-c/start",
    "method": "PUT"
  }
}
```

Note that the key is `_links` to reduce the likelihood of collisions with a hypothetical resource field named `links`.

## Collections
A collection is a list of multiple Resources.  A collection is represented as a JSON object.

A collection **MUST** contain a `resources:` field.  The resources field is an array containing multiple [Resources](#resources).

A collection **should** contain a `pagination:` field containing a [pagination](#pagination) object.

### Example

```json
{
  "resources": [
    {
      "guid": "a-b-c",
      "created_at": "2015-07-06T23:22:56Z",
      "updated_at": "2015-07-08T23:22:56Z",

      "_links": {
        "self": {
          "href": "/v3/resources/a-b-c"
        }
      }
    },
    {
      "guid": "d-e-f",
      "created_at": "2015-07-06T23:22:56Z",
      "updated_at": "2015-07-08T23:22:56Z",

      "_links": {
        "self": {
          "href": "/v3/resources/d-e-f"
        }
      }
    }
  ],
  "pagination": {
    "total_results": 2,
    "first": {
      "href": "/v3/resources?page=1&per_page=10"
    },
    "last": {
      "href": "/v3/resources?page=1&per_page=10"
    },
    "next": null,
    "previous": null
  }
}
```

## Pagination

Pagination **may** be used by [Collections](#collections) to limit the number of resources returned at a time.  Pagination is requested by a client through the use of query parameters. Pagination is represented as a JSON object.

This section will describe the preferred pagination strategy, however an individual collection **may** choose to implement a different strategy for performance reasons.  This means that the term "**MUST**", when used in this section, apply only to collections that implement the default pagination strategy.

Pagination **MUST** include a `total_results:` field with an integer value of the total number of records in the collection.

Pagination **MUST** include the following fields for pagination links:

* `first:` the first page of resources
* `last:` the last page of resources
* `previous:` the previous page of resources
* `next:` the next page of resources

Pagination links **may** be `null`.  For example, if the page currently being displayed is the first page, then it makes sense for the `previous` link to be null.

When pagination links contain a URL, they **MUST** be a JSON object with a field named `href:` containing a string with the URL for the next page.

The URL **should** include all query parameters required to maintain consistency with the original pagination request.  For example, if the client requested for the collection to be returned in a specific order direction via a query parameter, then the pagination links should include the proper query parameter to maintain the requested direction.

The following query parameters **should** be used for pagination:

* `page:` the page number of resources to return (default: 1)
* `per_page:` the number of resources to return in a paginated collection request (default: 50)
* `order_by:` a field on the resource to order the collection by; each collection may choose a subset of fields that it can be sorted by
* `order_direction:` the direction to returned the ordered collection in;  valid values **should** be `asc` or `desc`

If there are additional pagination query parameters, the parameters **MUST** have names that conform to the acceptable [query parameter](#query-parameters) names.

### Example

```json
"pagination:" {
  "total_results": 20,
  "first": {
    "href": "/v3/resources?order_by=created_at&order_direction=desc&page=1&per_page=10"
  },
  "last": {
    "href": "/v3/resources?order_by=created_at&order_direction=desc&page=2&per_page=10"
  },
  "next": {
    "href": "/v3/resources?order_by=created_at&order_direction=desc&page=2&per_page=10"
  },
  "previous": null
}
```

## Actions

Actions are API requests that are expected to initiate change within the Cloud Foundry runtime.  This is differentiated from requests which update a record, but require additional updates -- such as restarting an app -- to cause changes to a resource to take affect.  

Actions **MUST** use use PUT as their HTTP verb.

Actions **may** accept a request body.

Actions **should** be listed in the `_links:` for the related resource.

### Example
 `PUT /v2/apps/<guid>/start`

## Query Parameters

Query Parameters **MUST** include **ONLY** the following characters:

* a-z (lowercase only)
* _ (underscore)
* [
* ]

The bracket ([, ]) characters **MUST** be used **ONLY** at the end of the query parameter name, to indicate it accepts an array of values.

Query parameters that accept multiple values **should** be pluralized and use brackets to accept an array of values.

If any request receives a query parameter it does not understand, the response **MUST** be a `400 Bad Request`.

####Examples:
Single value:
`POST /v3/resources/start?async=true`

Multiple values:
 `GET /v3/resources?names[]=firstname,names[]=secondname`


## Filtering

Filtering is the use of query parameters to return a subset of resources within a [Collection](#collections).

Filter query parameters **MUST** have names that conform to the acceptable [query parameter](#query-parameters) names.

Filters **should** allow a client to request resources matching multiple values of by accepting the filter as an array.

Filter parameters **should** be able to be combined with other filters on the same collection.

#### Example multiple value request

`http://api.server.com/v3/resources?names[]=first_name,names[]=second_name`

#### Example single value request

`http://api.server.com/v3/resources?names[]=the_name`

#### Example combined filters

`http://api.server.com/v3/resources?names[]=the_name,relation_guids[]=d-e-f`

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


### Proposal A

```json
{
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

### Proposal B

This proposal includes the HTTP status code of the error.

```json
{
  "status": 400,
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

### Proposal C

This proposal includes `code` which would be an internal unique identifier of a class of error.  This could be used for support scripts for CF operators.  The method for maintaining a list of these codes and their meanings would need to be determined  

```json
{
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
|-|-|-|
|200 OK|This status **MUST** be returned for synchronous requests that complete successfully and have a response body. This should only be used if there is not a more appropriate 2XX response code. |GET, PATCH, PUT|
|201 Created|This status **MUST** be returned for synchronous requests that result in the creation of a new resource.|POST|
|202 Accepted|This status **MUST** be returned for requests that have been successfully accepted and will be asynchronously completed at a later time.|POST,PATCH,PUT,DELETE|
|204 No Content|This status **MUST** be returned for synchronous requests that complete successfully and have no response body.|DELETE


### Redirection

|Status Code|Description|Verbs|
|-|-|-|
|303 See Other| This status **MUST** be returned when an async job finishes. It should include a location header containing the resource link. See more in the [async](#asynchronicity) section. |GET, PATCH, PUT|


### Client Errors

|Status Code|Description|Verbs|
|-|-|-|
|400 Bad Request|This status **MUST** be returned for requests that provide malformed or invalid data. Examples: invalid JSON, unexpected query parameters or request fields.|GET, PATCH, POST, PUT, DELETE|
|401 Unauthenticated|This status **MUST** be returned if the requested resource requires an authenticated user but there is no OAuth token provided, or the OAuth token provided is invalid.|GET, POST, PATCH, DELETE, PUT|
|403 Forbidden|This status **MUST** be returned if the request cannot be performed by the user due to lack of permissions. Example: User with read-only permissions to a resource tries to update it. |POST, PATCH, DELETE, PUT|
|404 Not Found|This status **MUST** be returned if the requested resource does not exist or if the user requesting the resource has insufficient permissions to view the resource.|GET, POST, PATCH, PUT, DELETE|
|422 Unprocessable Entity|This status **MUST** be returned if the request is semantically valid, but performing the requested operation would result in a invalid state. Example: Attempting to start an app without assigning a droplet.|POST, PATCH, PUT|

### Server Errors

|Status Code|Description
|-|-|-|
|500 Internal Server Error|This status **MUST** be returned when an unexpected error occurs.
|502 Bad Gateway|This status **should** be returned when an upstream service failure causes a request to fail.

## Relationships

### Currently

Associations are sometimes created by setting a `relationship_guid:` field on one of the resources. For example, `POST /v3/apps { "space_guid": "abc" }` creates an app associated to a space.

Other times associations are added via PUT requests to a nested resource, For example, `PUT /v3/apps/guid/routes { route_guid: the guid }` will add a route association to an app.

An association can then be deleted via `DELETE /v3/apps/guid/routes { route_guid: the guid }`.

This currently feels a little clunky for several reasons:
* Associations are created in different ways depending on whether we consider it a required relationship or not
* The nested routes make it appear that the request is affecting a routes resource, but it is really a relationship that is being created or deleted
* Associations become a top-level concern on the resource such that the way relationships exist for that resource are difficult to modify without causing breaking changes or leaving unused fields

### Proposal A

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
  },
  "_links": {...}
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

### Proposal A1

Include the previous proposal, but include relationships on the root resource GET and allow updating relationships from the root resource.

Response body for GET /v3/apps/:guid
```json
{
  "name": "blah",
  "relationships": {
   "space": {"guid": "1234"},
   "routes": [
     {"guid": "2345"},
     {"guid": "3456"}
   ],
  },
  "_links": {...}
}
```

Request body for PATCH /v3/apps/:guid
```json
{
  "relationships": {
   "space": {"guid": "1234"},
   "routes": [
     {"guid": "2345"},
     {"guid": "3456"}
   ],
  }
}
```
In this example, `space` is a to-one relationship and `routes` is a to-many relationship.  The result is that space is changed and all routes are replaced with the values in the request. Therefore, to do a partial update of to-many resources, one would have to use the nested relationships routes listed in Proposal A.

An interesting side-effect of returning the relationships object on the resource body is that relationships and \_links have some amount of overlap.

## Nested Resources

Nested resources can optionally be accessed through their parent resource.
```
GET /v3/parent_resources/<parent_guid>/nested_resources
```
This will be equivalent to
```
GET /v3/nested_resources?parent_guid=<parent_guid>
```

These end points are optional and may not exist for all resources. Good opportunities for creating them:
* The resource commonly or exclusively lists within the context of the parent resource. E.g. droplets only exist inside apps.

## Including Related Resources

This is a mechanism for including multiple related resources in a single response.

A use case for this would be to display an HTML page that includes information about both an app and its space and would like to gather this information in one HTTP request.

### Proposal A

Included resources are in-lined at the top level of the primary resource.  If a resource within a resource -- `resource.otherresource` -- is requested, it is similarly inlined.

```
GET /v3/resource/:guid?include=subresource,other_resources,other_resources.subresource

{
  "guid": "xyz",
  "resource_field": "value",

  "subresource": {
    "guid": "abc",
    "subresource_field": "other value"
  },

  "other_resources": [
    {
      "guid": "def",
      "other_resource_field": "Zach",

      "subresource": {
        "guid": "abc",
        "subresource_field": "other value"
      }
    },
    {
      "guid": "ght",
      "other_resource_field": "Dances",

      "subresource": {
        "guid": "abc",
        "subresource_field": "other value"
      }
    },
  ]
}
```

Questions:
1. Do the sub resources include \_links and all that jazz?

### Proposal B

Included resources are in-lined in an `included:` object on the primary resource.  If a resource within a resource -- `resource.otherresource` -- is requested, it is similarly inlined.

```
GET /v3/resource/:guid?include=subresource,other_resources,other_resources.subresource

{
  "guid": "xyz",
  "resource_field": "value",

  "included": {
    "subresource": {
      "guid": "abc",
      "subresource_field": "other value"
    },
    "other_resources": [
      {
        "guid": "def",
        "other_resource_field": "Zach",

        "included": {
          "subresource": {
            "guid": "abc",
            "subresource_field": "other value"
          }
        }
      },
      {
        "guid": "ght",
        "other_resource_field": "Dances",

        "included": {
          "subresource": {
            "guid": "abc",
            "subresource_field": "other value"
          }
        }
      },
    ]
  }
}
```

### Proposal C

Included resources are in-lined under their pluralized resource name in an `included:` object on the primary resource.  If a resource within a resource -- `resource.otherresource` -- is requested, it is added in the top level `included:` object and not repeated.

Question: how do you tell which type of included resource goes with another resource (through a relationships object?)

```
GET /v3/resource/:guid?include=subresource,other_resources,other_resources.subresource

{
  "guid": "xyz",
  "resource_field": "value",

  "included": {
    "subresources": [
      {
        "guid": "abc",
        "subresource_field": "other value"
      }
    ],
    "other_resources": [
      {
      ( this resource also wanted the same subresource as above, but it was included once so we do not add it again)
        "guid": "def",  
        "other_resource_field": "Zach",
      },
      {
        "guid": "ght",
        "other_resource_field": "Dances",
      },
    ]
  }
}
```

## Requesting Partial Resources

```
GET /resource?fields=guid,name

{
  "guid": "some-guid",
  "name": "Zach"
}
```

### Proposal For Sub-Resources

If we want to be able to filter the fields of subresources, we could do something like:
```
GET /resource?fields[resource]=guid,name&fields[subresource]="color"

{
  "guid": "some-guid",
  "name": "Zach",
  "included": {
    "subresource": {
      "color": "red"
    }
  }
}
```


## Asynchronicity

### Currently

Via Mark: The original impetus for the `async` flag was
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
* `async=true` should only be available for resource operations that might take > 60 seconds (this might be impossible to determine).
* the hope is what endpoints are async should be intuitive to the user

To do a resource operation out of band, send the `async=true` flag.
`POST /v3/resource?async=true`

The CC will return a 202 with a location header pointing to the job.
```
202 Accepted
Location: /v3/jobs/123

{}
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
Location: /v3/resource/<guid>

{}
```

Note that for asynchronous deletes, the redirect location will be to a no-longer-existant resource.

### Async vs Accepts Incomplete

Currently the CC will ignore the `async` flag if `accepts_incomplete` is included. This does not have to be the case for v3.

|async|accepts_incomplete|Broker response|Result|
|-|-|-|-|
|true|true|synchronous|202 with job -> 303 to 'succeeded' instance|
|true|true|asynchronous|202 with job -> 303 to 'in progress' instance|
|false|true|synchronous|201 with 'succeeded' instance
|false|true|asynchronous|202 with 'in progress' instance

Another idea: We can assume that everyone using the v3 api accepts incomplete and have this be the only behavior.


## Style Guide Ideas (Style Guide Style Guide)

### Interesting Ideas
* Instead of `guid` everywhere, use `id`
* `accepts_incomplete` is a funky name
* `relationships` and other top level not-fields should be prefixed with '\_'  in the manner of `_links`
* We should probably have a unified example somewhere that includes everything
* Should we distinguish between `PUT` and `PATCH`? Which should we use in the docs?
* We should consider adding justifications for things like `_links` and `relationships` that express the expected use cases for API consumers.

### Questions
* Do we need to include response information about both `async` and `accepts_incomplete`
* Are the `_links` and `relationships` sections compatible? There may be duplicate/ conflicting information. Maybe these sections should be closer together.
* How should we behave when service brokers misbehave? 502? 50X?
* How should we handle resources with many links (instances)?

### Areas of Contention
* Greg thinks that actions should be POSTS instead of PUTS. http://restful-api-design.readthedocs.org/en/latest/methods.html#actions

### Known SG Defects
* The styleguide currently alternates between interpolation styles (`/v3/apps/<guid>` vs `/v3/apps/:guid`)
