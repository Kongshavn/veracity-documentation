---
Title: Service API
Author: "Brede Børhaug"
Contributors: "Rachel Hassall"
---

## Overview 

The Service API provides a consistent interface for accessing data about the user, as well as the Veracity platform in general. At its core is a REST service built up of "view-points" where you can read, write and update information. Authentication is handled through Azure B2C OAuth 2.

## Api Management

You can access documentation and try out the api through the Api Management catalogue [Veracity Api management](https://api-test-portal.veracity.com/).
Note to existing Veracity developers, we will sunset the direct access to the api and you will only be allowed to access it through  https://api.veracity.com/platform

## Versioning

We are committed to keep 2 versions alive at any given time. We will only make a new version when we need to introduce breaking changes to our service, while extentions to the api will be released without adding a new version.

### Examples of changes without introducing a new version:

- adding a new action 
- adding new properties to the existing responses

## Authentication

Authentication is performed through OAuth2 bearer tokens.

### Service api subscription

To access the service you need to register your application in our api management catalogue [Veracity Api management](https://api-test-portal.veracity.com/) and pass your subscription key through the `Ocp-Apim-Subscription-Key` header.

## View-points

The API defines three primary view-points from which you can access more detailed information. The base url for requests is:

```url
https://api.veracity.com/platform/[view-point]

e.g.:
https://api.veracity.com/platform/my/profile
```

These are:

|View-point|Path|Description|
|:---------|:-------|:----------|
|Directory|`/directory`|Information about different master data and resources with in myDNVGL. The main categories are: services, users and companies. This view point is reserved for advances services and require more priviledges|
|My|`/my`|Provides information about the currently logged in user.|
|This|`/this`|Provides information from the service or applications point of view, its capabilities and metadata.|

The view-points themselves do not provide information directly, instead you interact with them through defined **actions**. An action is just a string that you append to the end of the view-point path in order to perform a request. The actions follow normal usage of HTTP verbs with some minor exceptions.

### Responses

Each response provides all, or most, of these headers:

|Header|Type|Description|
|:-----|:--:|:----------|
|x-supportcode|string|Provides a unified way of correlating log entries accross all system components.|
|x-serviceversion|string|The api build number.|
|x-timer|int|The time spent on the server producing the response in milliseconds. |
|x-region|string|The Azure service region serving the request.|
|x-principal|string|The user the request was executed on behalf of.|
|x-view-point|string|The name of the current view-point.|
|x-actor|GUID|The user id of the actor/service account. |

The response status code describes whether or not the request succeeded. Currently,the following status codes may be returned:

|HTTP Status|Name|Description|
|:----------|:---|:----------|
|200|OK|Your request was processed correctly. View content for response.|
|204|No Content|Your request was processed correctly and no content was returned.|
|300|Ambiguous|Your request could point to multiple resources. You should augment it with additional identifying information.|
|400|Bad Request|The view-point/action exists, but the way you formatted the request was incorrect. Check `http verb`, `headers` or `body`.|
|403|Forbidden|The requester has insufficient permissions to perform the action or authorization information is missing from the request. Check that you provide a valid OAuth2 `Authorization` header.|
|406|Not Acceptable|Returned from the validate policy actions, the error body wil contain the url to the veracity accept terms page|
|404|Not Found|The requested resource/view-point/action was not found or is not known.|
|500|Internal Server Error|Something went wrong on the server when processing your request. Try to include the `x-supportcode` header content if you wish to submit a support request.|
|501|Not Implemented|The view-point or action is not currently implemented, but may be in the future.|

#### Response format

The API supports formatting the response body according to the mime type provided in your requests `Accept` header. Currently the following mime types are supported:

- `application/json`
- `text/json`
- `application/xml`
- `text/xml`

### Directory

The primary purpose of the **Directory** view-point is to provide information about your "surroundings". Using it you can get information about companies, services and users in the system. 

#### Actions

The following actions are supported on the `/directory` view-point. Parameters in urls are indicated by `{}` and should be replaced when using the action.

|Action|Method|Description|
|:-----|:----:|:----------|
|`/directory/companies/{id}`|`GET`|Get details about a specific company.|
|`/directory/companies/{id}/users`|`GET`|Get users affiliated with a specific company.|
|`/directory/services/{id}/datacontainers`|`GET`|List the service's datacontainers|
|`/directory/services/{id}/datacontainers/{containerId}`|`PUT` `DELETE`|Add or Remove a datacontainer reference for a service|
|`/directory/services/{id}`|`GET`|Get details about a specific service.|
|`/directory/services/{id}/users`|`GET`|Get users subscribed with a specific service.|
|`/directory/services/{serviceId}/administrators/{userId}`|`GET`|Check if the user is an administrator of the service.|
|`/directory/users/email?email={email}`|`GET`|Get information about all users associated with a specific email address.|
|`/directory/users/{id}`|`GET`|Get the full profile for a specific user.|
|`/directory/users`|`POST`|Get full profile information for multiple users.|
|`/directory/users/{id}/companies`|`GET`|Get companies the user is affiliated to.|
|`/directory/users/{id}/services`|`GET`|Get services the user is subscribed to.|
|`/directory/users/{userid}/services/{serviceid}`|`GET`|Get the subscription details for a given user, with regard to a specific service|

#### `/directory/companies/{id}`

Response format:

```
CompanyInfo {
  name (string, optional),
  url (string, optional),
  usersUrl (string, optional),
  addressLines (Array[string], optional),
  id (string, optional),
  city (string, optional),
  country (string, optional),
  countryCode (string, optional),
  zipCode (string, optional),
  #employee (integer, optional),
  domains (string, optional),
  email (string, optional),
  #requests (integer, optional)
}
```

#### `/directory/companies/{id}/users`

This action returns a paged result. You must provide a page number as well as a page size as query parameters to this request.

E.g.:
```url
/directory/companies/{id}/users?page=1&pageSize=15
```

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    email (string, optional),
    name (string, optional),
    id (string, optional)
  },
  ...
]
```

#### `/directory/services/{id}`

Response format:

```
ServiceInfo {
  name (string, optional),
  shortDescription (string, optional),
  description (string, optional),
  apiAudience (string, optional),
  category (string, optional),
  public (boolean, optional),
  id (string, optional),
  inherited (boolean, optional),
  selfSubscribe (boolean, optional),
  serviceOwner (string, optional),
  termsOfUse (string, optional),
  lastUpdated (string, optional),
  parentUrl (string, optional, read only),
  parentId (string, optional),
  childrenUrl (string, optional, read only),
  servicerUrl (string, optional)
}
```

#### `/directory/services/{id}/users`

This action returns a paged result. You must provide a page number as well as a page size as query parameters to this request.

E.g.:
```url
/discover/services/{id}/users?page=1&pageSize=15
```

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    email (string, optional),
    name (string, optional),
    id (string, optional)
  },
  ...
]
```

#### `/directory/users/email?email={email}`

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    email (string, optional),
    name (string, optional),
    id (string, optional)
  },
  ...
]
```

#### `/directory/users/{id}`

Response format:

```
UserInfo {
  name (string, optional): Contains the users formatted name: {lastName}, {firstName}. the id token contains the discrete values in the givenName and surname claims. ,
  email (string, optional): The users registered email address. if verifiedEmail is true this can be used to contact the user. ,
  id (string, optional),
  company (CompanyReference, optional): Contains the default company affiliation if any. ,
  verifiedEmail (boolean, optional): true if email is verified by the user; otherwise, false. ,
  language (string, optional): Contains the perfered language for the user. If your service support multi-language use this. ,
  identity (string, optional): The relative path to the resource details ,
  servicesUrl (string, optional, read only): Gets the relative url to the users service lits ,
  companiesUrl (string, optional, read only): Gets the relative url to the users companies lits ,
  firstName (string, optional),
  lastName (string, optional)
}
CompanyReference {
  identity (string, optional, read only): The relative path to the resource details ,
  name (string, optional),
  id (string, optional),
  description (string, optional)
}
```

#### `/directory/users`

The body of this request should be a list of user IDs. For example:

```json
[
  "eacf14df-5f5c-482c-9a12-d1444b69ae82",
  "325d5051-b68e-486b-bbf1-25416cb5035d"
]
```

The following request body types are supported:

- `application/json`
- `text/json`
- `application/xml`
- `text/xml`
- `application/x-www-form-urlencoded`

Remember to set the `Content-Type` header of your request to `application/json`.

Response format:

```
[
  {
    name (string, optional): Contains the users formatted name: {lastName}, {firstName}. the id token contains the discrete values in the givenName and surname claims. ,
    email (string, optional): The users registered email address. if verifiedEmail is true this can be used to contact the user. ,
    id (string, optional),
    company (CompanyReference, optional): Contains the default company affiliation if any. ,
    verifiedEmail (boolean, optional): true if email is verified by the user; otherwise, false. ,
    language (string, optional): Contains the perfered language for the user. If your service support multi-language use this. ,
    identity (string, optional): The relative path to the resource details ,
    servicesUrl (string, optional, read only): Gets the relative url to the users service lits ,
    companiesUrl (string, optional, read only): Gets the relative url to the users companies lits ,
    firstName (string, optional),
    lastName (string, optional)
  },
  ...
]

CompanyReference {
  identity (string, optional, read only): The relative path to the resource details ,
  name (string, optional),
  id (string, optional),
  description (string, optional)
}
```

#### `/directory/users/{id}/companies`

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    name (string, optional),
    id (string, optional),
    description (string, optional)
  },
  ...
]
```

#### `/directory/users/{id}/services`

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    name (string, optional),
    id (string, optional),
    description (string, optional)
  },
  ...
]
```

#### `/directory/users/{userid}/services/{serviceid}`

Response format:

```
SubscriptionDetails {
  service (ServiceReference, optional),
  user (UserReference, optional),
  role (RoleReference, optional)
}
ServiceReference {
  identity (string, optional, read only): The relative path to the resource details ,
  name (string, optional),
  id (string, optional),
  description (string, optional)
}
UserReference {
  identity (string, optional, read only): The relative path to the resource details ,
  email (string, optional),
  name (string, optional),
  id (string, optional)
}
RoleReference {
  identity (string, optional, read only): The relative path to the resource details ,
  type (string, optional),
  name (string, optional),
  id (string, optional)
}
```

### My

The `/my` view-point provides information from the point of view of the currently logged in user.

#### Actions

The following actions are supported on the `/my` view-point. Parameters in urls are indicated by `{}` and should be replaced when using the action. 

|Action|Method|Description|
|:-----|:----:|:----------|
|`/my/profile`|`GET`|Get the profile of the currently logged in user.|
|`/my/messages/count`|`GET`|Get the number of unread messages of the currently logged in user.|
|`/my/messages`|`GET`|Get all messages for the currently logged in user.|
|`/my/companies`|`GET`|Get companies the currently logged in user is affiliated with.|
|`/my/policies/{serviceId}/validate()`|`GET`|Validates all myDNVGL policies for a specific service and returns a list of the policies that need validation. |
|`/my/policies/validate()`|`GET`|Validates all myDNVGL policies and returns a list of the policies that need validation.|
|`/my/services`|`GET`|Get all services the user is subscribed to.|

#### `/my/profile`

Response format:

```
MyUserInfo {
  profilePageUrl (string, optional, read only),
  messagesUrl (string, optional, read only),
  identity (string, optional),
  servicesUrl (string, optional, read only): Gets the relative url to the users service lits ,
  companiesUrl (string, optional, read only): Gets the relative url to the users companies lits ,
  name (string, optional): Contains the users formatted name: {lastName}, {firstName}. the id token contains the discrete values in the givenName and surname claims. ,
  email (string, optional): The users registered email address. if verifiedEmail is true this can be used to contact the user. ,
  id (string, optional),
  company (CompanyReference, optional): Contains the default company affiliation if any. ,
  verifiedEmail (boolean, optional): true if email is verified by the user; otherwise, false. ,
  language (string, optional): Contains the perfered language for the user. If your service support multi-language use this. ,
  firstName (string, optional),
  lastName (string, optional)
}
CompanyReference {
  identity (string, optional, read only): The relative path to the resource details ,
  name (string, optional),
  id (string, optional),
  description (string, optional)
}
```

#### `/my/messages/count`

Response format:

```
Int32
```

#### `/my/messages`

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    name (string, optional),
    id (string, optional),
    description (string, optional)
  }
]
```

#### `/my/companies`

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    name (string, optional),
    id (string, optional),
    description (string, optional)
  }
]
```

#### `/my/policies/{serviceId}/validate()`

Validates the platform terms and services spesific terms.

Headers: 

returnUrl: the absolute url to the page in your service you want the user to be redirected to after accepting the terms

Response format:

204 - No validation errors

406 - Policy violations, redirect the user to the policy approval page.

```JSON
{
  "url": "urlToAcceptTermsPage",
  "violatedPolicies": [
    "string"
  ],
  "message": "string",
  "information": "string",
  "subCode": 0,
  "supportCode": "string"
}
```

#### `/my/policies/validate()`

Validates the platform terms.

Headers: 

returnUrl: the absolute url to the page in your service you want the user to be redirected to after accepting the terms

Response format:

204 - No validation errors

406 - Policy violations, redirect the user to the policy approval page.

```JSON
{
  "url": "urlToAcceptTermsPage",
  "violatedPolicies": [
    "List of violated policies"
  ],
  "message": "string",
  "information": "string",
  "subCode": 0,
  "supportCode": "string"
}
```

#### `/my/services`

Response format:

```
[
  {
    serviceUrl (string, optional): the location of the application. ,
    identity (string, optional, read only): The relative path to the resource details ,
    name (string, optional),
    id (string, optional),
    description (string, optional)
  }
]
```

### This

`/this` is the service/applications point of view and provides information from the context of a service or application. Authenticate using a service account before using these actions.

|Action|Method|Description|
|:-----|:----:|:----------|
|`/this/services`|`GET`|Get all services the service principal has access to.|
|`/this/subscribers`|`GET`|Get all users currently subscribed to this service.|
|`/this/services/{serviceId}/subscribers`|`GET`|Get all users currently subscribed to a specific service the current service account is associated with.|
|`/this/subscribers/{userId}`|`PUT`, `DELETE`|Add or remove service subscriptions for a specifc user.|
|`/this/services/{serviceId}/subscribers/{userId}`|`PUT`, `DELETE`|Add or remove service subscriptions for a specifc user.|
|`/this/user/resolve({email})`|`GET`|Use this to verify that a user with the specified email address exists in Veracity|
|`/this/user`|`POST`|Create a user in myDNVGL|
|`/this/users`|`POST`|Create multiple users in myDNVGL|
|`/this/services/{serviceId}/notification`|`POST`|Send a notification to the user through the Veracity notification services (web or email at the moment, but we are investigating the possibillity to add push notifications through the browser or native app)|

#### `/this/services`

This action returns a paged result. You must provide a page number as well as a page size as query parameters to this request.

E.g.:
```url
/this/services?page=1&pageSize=15
```

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    name (string, optional),
    id (string, optional),
    description (string, optional)
  }
]
```

#### `/this/subscribers`

**warning**: This action may return `300 Ambiguous` if the service account you are authenticated with is linked to multiple services.

This action returns a paged result. You must provide a page number as well as a page size as query parameters to this request.

E.g.:
```url
/this/subscribers?page=1&pageSize=15
```

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    email (string, optional),
    name (string, optional),
    id (string, optional)
  }
]
```

#### `/this/services/{serviceId}/subscribers`

This action returns a paged result. You must provide a page number as well as a page size as query parameters to this request.

E.g.:
```url
/this/services/{serviceId}/subscribers?page=1&pageSize=15
```

Response format:

```
[
  {
    identity (string, optional, read only): The relative path to the resource details ,
    email (string, optional),
    name (string, optional),
    id (string, optional)
  }
]
```

#### `/this/subscribers/{userId}`

**warning**: This action may return `300 Ambiguous` if the service account you are authenticated with is linked to multiple services.

Adds or removes subscribers to the service. Add by using the HTTP verb `PUT` and remove by using `DELETE`.

The `PUT` request must provide a body of the following format:

```
SubscriptionOptions {
  role (string, optional)
}
```

The following request body types are supported:

- `application/json`
- `text/json`
- `application/xml`
- `text/xml`
- `application/x-www-form-urlencoded`

#### `/this/services/{serviceId}/subscribers/{userId}`

Adds or removes subscribers to the service. Add by using the HTTP verb `PUT` and remove by using `DELETE`.

The `PUT` request must provide a body of the following format:

```
SubscriptionOptions {
  role (string, optional)
}
```

The following request body types are supported:

- `application/json`
- `text/json`
- `application/xml`
- `text/xml`
- `application/x-www-form-urlencoded`

#### `/this/user`

Request format:
```
UserRegistration {
  firstName (string, optional),
  lastName (string, optional),
  email (string, optional),
  options (RegistrationOptions, optional): Specify additional creation control options, this is not mandatory
}

RegistrationOptions {
  sendMail (boolean, optional): Set this to false to take responsibility of sending the registration email to the user. ,
  createSubscription (boolean, optional): Make the service create a default subscription for the newly created user ,
  serviceId (string, optional): The service id to create subscription for ,
  role (string, optional): Specify the accessLevel/role the user should have with the new subscription. Optional
}
```

The following request body types are supported:

- `application/json`
- `text/json`
- `application/xml`
- `text/xml`
- `application/x-www-form-urlencoded`

#### `/this/users`

Request format:
```
[
  UserRegistration {
    firstName (string, optional),
    lastName (string, optional),
    email (string, optional),
    options (RegistrationOptions, optional): Specify additional creation control options, this is not mandatory
  }
]

RegistrationOptions {
  sendMail (boolean, optional): Set this to false to take responsibility of sending the registration email to the user. ,
  createSubscription (boolean, optional): Make the service create a default subscription for the newly created user ,
  serviceId (string, optional): The service id to create subscription for ,
  role (string, optional): Specify the accessLevel/role the user should have with the new subscription. Optional
}
```

The following request body types are supported:

- `application/json`
- `text/json`
- `application/xml`
- `text/xml`
- `application/x-www-form-urlencoded`

## GitHub  
Follow our open projects related to Veracity Services API on https://github.com/veracity

## Stack Overflow
Stack Overflow is the largest, most trusted online community for developers to learn, share their programming knowledge. The Veracity developer team monitor Stack Overflow forum posts that include the tag Veracity Platform.

[Visit Stack Overflow](https://stackoverflow.com/questions/tagged/veracity+platform?mode=all)
