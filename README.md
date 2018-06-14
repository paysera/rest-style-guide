# Table of contents

1. [Versioning](#versioning)

    1.1. [Providing Version](#providing-version)
    
    1.2. [Changes in API](#changes-in-api)
    
    1.2.1. [Backwards Incompatible Changes](#backwards-incompatible-changes)
    
    1.2.2. [Bumping the Version](#bumping-the-version)
    
    1.2.3. [Changing Workflow and/or Resources Completely](#changing-workflow-and/or-Resources-completely)
    
2. [Size of API](#size-of-api)

3. [HTTP Methods](#http-methods)

4. [URLs](#urls)

    4.1. [Dashes](#dashes)

    4.2. [URLs as Resources](#urls-as-resources)
    
    4.2.1. [Actions with Resources](#actions-with-resources)
    
    4.3. [Plural for Resource Names](#plural-for-resource-names)
    
    4.3.1. [Exception for Singletons](#exception-for-singletons)
    
    4.4. [Relations after Resource URL](#relations-after-resource-url)
    
    4.5. [Query String for Filtering](#query-string-for-filtering)
    
    4.5.1. [No filtering by ID](#no-filtering-by-id)
    
5. [Defining Resources](#defining-resources)

    5.1. [Types of Resources](#types-of-resources)
    
    5.2. [Main Resources](#main-resources)
    
    5.3. [Embedded Resources](#embedded-resources)
    
    5.4. [Related Resources](#related-resources)
    
6. [Naming](#naming)

    6.1. [Nouns for Resources and Relations](#nouns-for-resources-and-relations)
    
    6.2. [Verbs for Resource Actions](#verbs-for-resource-actions)
    
    6.3. [Unique Names](#unique-names)
    
    6.4. [Naming Relations](#naming-relations)
    
7. [Request and Response Contents](#request-and-response-contents)

    7.1. [Creating Resources](#creating-resources)
    
    7.2. [Editing Resources](#editing-resources)
    
    7.3. [Deleting Resources](#deleting-resources)
    
    7.4. [Performing Actions on Resources](#performing-actions-on-resources)
    
    7.5. [Getting Resource by ID](#getting-resource-by-id)
    
    7.6. [Getting List of Resources](#getting-list-of-resources)
    
    7.7. [Embedded Resources](#embedded-resources)
    
    7.8. [Errors](#errors)
    
8. [Encoding and Structures](#encoding-and-structures)

    8.1. [JSON](#json)
    
    8.2. [JSON Types](#json-types)
    
    8.2.1. [No Floats](#no-floats)
    
    8.2.2. [Integers Only for Numeric Fields](#integers-only-for-numeric-fields)
    
    8.3. [Snake Case for Properties](#snake-case-for-properties)
    
    8.4. [Object as Response](#object-as-response)
    
    8.5. [Data Types](#data-types)
    
    8.5.1. [Date Time](#date-time)
    
    8.5.2. [Date](#date)
    
    8.6. [IDs](#ids)
    
    8.6.1. [IDs as Strings](#ids-as-strings)
    
    8.6.2. [Global identifiers](#global-identifiers)
    
    8.6.3. [No duplicate IDs](#no-duplicate-ids)
    
    8.7. [Binary Responses](#binary-responses)
    
    8.7.1. [Public Binary Files](#public-binary-files)
    
9. [Changing state of object](#changing-state-of-object)

10. [Cache Response Headers](#cache-response-headers)

    10.1. [ETag](#etag)
    
    10.2. [Vary](#vary)
    
11. [Same Resource Structure](#same-resource-structure)

    11.1. [A/B, Alpha and Beta Testing](#a/b,-alpha-and-beta-testing)
    
    11.1.1. [A/B Testing](#a/b-testing)
    
    11.1.2. [Feature Flags](#feature-flags)
    
12. [I18N](#i18n)

13. [Pagination](#pagination)

    13.1. [Always Paginated Lists](#always-paginated-lists)
    
    13.2. [Pagination Strategies](#pagination-strategies)
    
    13.2.1. [Offset and Limit](#offset-and-limit)
    
    13.2.2. [After and Next After](#after-and-next-after)
    
    13.2.3. [Calculating Next After](#calculating-next-after)
    
14. [Authentication](#authentication)

    14.1. [No Cookies](#no-cookies)
    
    14.1.1. [If we need to use cookies](#if-we-need-to-use-cookies)
    
    14.2. [Auth2 for getting permissions](#auth2-for-getting-permissions)
    
    14.3. [JWT tokens](#jwt-tokens)
    

# Versioning

## Providing Version

We use URL prefix with version in REST endpoints.

**Why not Accepts header with version?** There might be cases where the whole structure and/or workflow of API changes and not only the resource representation. Such changes cannot be gracefully handled by Access header with different version. Also in some cases we might get resource that is (dynamically) embedded in another resource. This would require very complicated Access headers as we would need to provide version for each resource separately. For example, if we have event about any object in the system and the object is embedded in the event resource, we should provide version for each and every resource in the system to provide needed resource representation.

## Changes in API

### Backwards Incompatible Changes

We never make backwards incompatible changes inside the API without bumping the version.

### Bumping the Version

We try to keep the version the same as long as possible (and viable). We make additions to the API without bumping the version - these are backwards compatible changes. If we need to change something, we tend to add new functionality and deprecate the old one.

When seems practical or needed, we bump the version removing all deprecated functionality.

### Changing Workflow and/or Resources Completely

We can make new REST interface with new version separately, then deprecate old version and remove it completely when no one uses it anymore.

In this case we should migrate / route all needed functionality that was left the same from previous to the next version.

In other words, we try to change version for the whole API, not for separate resources. We can add new resources and functionality for the current version without bumping it.

# Size of API

We try to make small APIs with related functionality. Even if all resources are in the same system, we try to group functionality into separate APIs.

This allows to make version bumps for each API separately.

Every API should have it's own endpoint (dedicated domain and/or path prefix).

# HTTP Methods

We use `GET`, `POST`, `PUT` and `DELETE` methods in APIs as defined in HTTP specification:
- `GET` only for getting information. `GET` requests must never modify resources or change state of the system;
- `POST` for creating resources. If we repeat same `POST` request, new resources are added;
- `DELETE` for deleting resources;
- `PUT` for modifying resources and changing their state. If we repeat same `PUT` request, it must not create new resources or make additional changes in the system. `PUT` requests are repeatable.
 
When editing resource with `PUT`, we provide all structure of resource. If some field is not provided, then it should be deleted if possible from the resource.

Currently we do not use `PATCH` requests, which would have different behaviour.

# URLs

## Dashes

We use lower-cased URLs with dashes (`-`) to split words.

## URLs as Resources

URL (without predefined endpoint prefix) indicates a resource.

For example `/payments` to indicate payment collection, `/payments/{id}` to indicate single payment. Full URI can be `https://checkout.paysera.com/payments/rest/v3/payments/{id}` or similar (`/payments/rest/v3` being a prefix for this API).

### Actions with Resources

Last part of URL can be an action, which will be performed on provided resource. This action is not used, if it’s one of CRUD actions.

For example, to create payment we use `POST /payments`, to edit payment we use `PUT /payments/{id}`, to confirm payment we use `PUT /payments/{id}/confirm`.

We also provide action when getting resources in some custom way in the same pattern. For example, if we need to get contacts filtered by business and country we use `GET /contacts?business_id=1&country=LT`; this should provide only the resources that match the filter provided. If we want to resolve some concrete contact from given parameters, we use `GET /contacts/resolve?business_id=1&country=LT`; this can return either collection or single resource (we should document this clearly) and can make some resolving decitions on server side - in this case, choose the best available contact for that business in provided country.

Action is always a verb - this way it is clear that it is not a relation to other resource (as relations are always nouns).

## Plural for Resource Names

We always use plural for naming resources.

For example `GET /businesses`, `GET /businesses/{id}`, `GET /businesses/{id}/money-receivers`.

Keep in mind that plural ending is added only for last word in resource name, for example `email-receivers`, not `emails-receivers`.

### Exception for Singletons

Exception: singletons in system or *-to-one relation inside a resource.

For example `GET /status`, `GET /accounts/{id}/owner`.

In these cases we always return single resource, not a list of resources.

## Relations after Resource URL

To indicate some related resource, first we indicate the resource itself, then add relation name to the end of URI. For example we use `/payments/{id}/items` to get items in the specific payment, but not `/payments/items/{id}` or similar.

This allows to understand which relations belong to which resources and that specific ID provided belongs to some concrete resource and not to the other. Also this helps to avoid clashes for routing, as "items" can be a valid ID.

## Query String for Filtering

When we query a collection of some resource, we provide filtering by query string.

For example, `/payments?beneficiary_id=beww2124j`.

### No filtering by ID

We do not put resource ID into collection filter - we provide endpoint to get the resource by the ID if needed.

# Defining Resources

## Types of Resources

There are 3 types of data:
- **main resource**. Resource has its own life cycle and can be created, edited and deleted separately;
- data, included in the resource (**embedded resource**). It belongs to the main resource and always comes with it;
- **related resource**. It can have its own life cycle or be completely dependent on another resource, but it is not created by it's own - it's always related to some other main resource. See below for distinction between embedded resources.

We choose the type by these rules:
- if we can create resource by itself and it does make sense, then it's a **main resource**;
- if information is related strictly to single resource, cannot contain large collections and directly relates to the main functionality of a resource, then it's **embedded resource**;
- otherwise it's **related resource**. Usually it has at least one of these properties:
  - can be related to several other resources (has relation, not dependency, or has dependency on few different resources);
  - any number of resources can be created (or large number);
  - main resource should not include this one, as it's not directly related, needed only in some cases and/or should be provided by different API (possibly in same domain).

## Main Resources

Main resources always have ID in their structure. They have their own endpoints if needed (`POST /main-resources`, `GET /main-resources`, `GET /main-resources/{id}` etc.) from root of API endpoint.

They are never included inside any other resources (unless asked explicitly to be embedded) - only their IDs are provided.

Relations between the resources can be included inside the resource content or not, depending on situation. This is really similar to distinction between embedded and related resources: if there can be many relations (than could be paginated), or relation is not directly related to the resource itself, we should not include relations inside the main resource. In such case we get them by separate endpoint.

For example `GET /main-resources/{id}/relations`.

To identify relation and not the related resource itself, we can either assign ID to the relation (the relation itself becomes *related resource*) or identify it by ID of that resource.

For example `DELETE /main-resources/{id}/another-main-resources/{id2}` would delete relation from main resource with {id} to another main resource with {id2}.

## Embedded Resources

Embedded resources always comes inside the main resource. They never have ID in their structure. They do not have their own endpoints from root of API endpoint. They usually have no endpoints at all (if really needed, we can provide `GET /main-resources/{id}/embedded-resources` or similar).

We edit them together with the main resource. We edit main resource instead of creating or removing embedded resources.

## Related Resources

Related resources always have ID in their structure. 

They are usually created by endpoints of main resources to provide direct dependency/relation.

For example `POST /main-resources/{id}/related-resources`.

As main resources, related resources are not included in other structures.

Related resources usually have endpoint for getting the resources related to some concrete main resource. This collection can be paginated (if resource could be paginated, then it cannot be embedded resource, as this would not allow it).

For example `GET /main-resources/{id}/related-resources?limit=100`.

They usually have their own endpoints for editing, deleting and filtering globally (this is usually used by other systems or administrators, not resource owners).

For example `GET /related-resources?field=value`, `DELETE /related-resources/{id}`, `PUT /related-resources/{id}`.

We do not put both IDs of main resource and related resource inside the URL as this makes some tasks much more difficult. It also allows to represent single resource with multiple different URLs if a resource is related to several different main resources. For example, this would be wrong: `DELETE /main-resources/{id}/related-resources/{id}`.

# Naming

## Nouns for Resources and Relations

We use nouns for both resource names and relations.

## Verbs for Resource Actions

We always use verb to identify custom action with a resource.

## Unique Names

We give unique names for different resources in the API. Structure might be different with same resource name in different APIs, but we should avoid it.

We should try to name relations uniquely, so that relation with the same name would not point to two different resources (for example `business.owner_id` and `account.owner_id` should both be compatible - these should be relations to the same resource, for example `User`).

## Naming Relations

We name relations by their type, not by the resource type they're pointing to.

For example, we use `owner_id` to identify owner of some resource, not `user_id`, even if it points to `User` resource.

# Request and Response Contents 

## Creating Resources

We provide structure of a resource when creating it.

We return structure of a resource in response.

We can accept properties that are read-only (only returned in the response) and ignore them when creating the resource - this allows to clone (possibly editing before that) some other resource much more easily for API client.

## Editing Resources

We provide same resource structure as when creating the resource to edit it. This can allow us to pass more or less properties than when creating, but the main structure must be the same.

We can accept properties that are read-only, just like when creating. This allows to get the resource, change needed parameters and send it back to the API - no need to filter out unmodifiable properties like `id`.

We return modified resource as a response.

## Deleting Resources

We do not provide request body when deleting resources.

We return either structure of deleted resource (only if resource can be soft-deleted and retrieved later, for example if deletion just changes status to `deleted` or similar) or `204 No Content` response. Such response has no content (duhh).

## Performing Actions on Resources

When performing actions, we provide no request content or we provide structure for action parameters (not the resource itself).

We return modified resource as a response.

## Getting Resource by ID

We return resource structure as a response.

## Getting List of Resources

We always return paginated list object as a response.

The object has two elements: `items` which holds array of resource structures and `_metadata` which has one of `total`, `next_after` or both of them. See "Pagination" below.

## Embedded Resources

We can include whole structures of related resources if explicitly asked with `fields` query parameter.

This parameter can let embed "hidden" elements and filter unneeded elements that would be provided by default.

For example `GET /payments/{id}?fields=id,beneficiary.*` would include only `id` and `beneficiary` fields in response structure; `beneficiary` would have it's default structure and would be embedded in `payment` resource even if only ID is provided by default.

## Errors

We provide error structure in the response if error has occured.

We always indicate error by response code: `500` for server error (we've f***ed up, we need to fix something), `4xx` for client-side error (wrong data passed, resource not found, state does not allow making some action etc).

Error structure has the following fields:
- `error`. Required. Indicates code of the error (string). Should be unique in that endpoint to identify the problem in client side;
- `error_description`. Description of the error for developer on the client side. Always in English. Should not be displayed directly to the end user, ever;
- `error_uri`. URI with more information about the error, meant for developer on the client side;
- `error_properties`. Localized error messages meant for end user. Provided only when `error` is `invalid_properties`. It's object, key is property path in resource structure, value is array of errors (strings);
- `error_data`. Additional data passed with error. Object, depends on concrete `error` and endpoint. For example, if `error` is `code_already_sent`, `error_data` could include time when that code was sent or when user could repeat sending it.

# Encoding and Structures

## JSON

We use JSON (optionally with some other encoding, like XML) to provide responses for resources.

## JSON Types

### No Floats

We do not use floats in JSON responses. We either use integers or strings, as floats loose precision.

### Integers Only for Numeric Fields

We use integers only when data can be added together, subtracted or at least compared.

For example, we can use integers for counts of different sorts.

We do not use integers for enumeration or IDs. These are not comparable (IDs could represent time created, but usually does not). We use strings for these.

## Snake Case for Properties

We use lower-case underscored keys in JSON objects.

## Object as Response

We always return JSON object as a response. We do not use arrays and/or scalar types as a response.

**Why not scalars?** If we return scalar type, we cannot extend response if needed in any way. Also, just scalar JSON (in top level) is not a valid JSON. In Java and some libraries it cannot be parsed at all.

**Why not arrays?** If we return array, we cannot extend response if needed in any way (provide metadata etc.)

## Data Types

### Date Time

We use integer UNIX timestamp to represent date with time.

### Date

We use string in format `yyyy-mm-dd` to represent date.

## IDs

### IDs as Strings

We use string to represent any ID.

**Why?** This gives much more possibilities - we can include server ID for some scaling solutions, we can generate UUID, we can include object’s type to give unique ID for any of resources returned from API (not only between same resources).

### Global identifiers

If object has global identifier which can be used instead of the local one, we always provide the global one.

For example, we use `user_id` (used globally in several systems) instead of `client_id` (used in single system) if possible, we also use account number instead of account ID, which has meaning only internally in the system.

### No duplicate IDs

If we have some field that already uniquely identifies the resource, we do not use second ID in the same resource.

For example, if we have resource hash for public access, we do not provide both hash and ID in resource structure. We just provide the hash as resource's ID as it must be unique in either case. We use it's ID only internally. From API client's point of view, hash **is** the ID.

Another example is when we use predefined (configured) keys for resources. These are also resource IDs. For example `method_key` for `PaymentMethod` resource - `GET /payment-methods/{method_key}` would return information about some specific method.

## Binary Responses

We can return binary responses. These should be indicated by `Accept` request header and/or extension in URL.

### Public Binary Files

We should not return public binary files directly in API responses. We should provide public URL to get those resources.

This allows to store them separately in dedicated server(s) use CDN and effective cache headers for distribution. Also this allows to have different URL for different version of binary file of the same resource, which allows to add very long caching for the responses (for example, 9001 years, or at least six months).

# Changing state of object

We give separate endpoints to complete different actions with resource.

This allows to give separate parameters for each of the actions, also to see the available actions more clearly.

We prefer this over modifying resource while changing it's status. All status changes should be handled by `DELETE` request and/or actions with the resource.

Fo example:

```php
PUT /request/{id}/accept HTTP/1.0
```

```php
PUT /request/{id}/deny HTTP/1.0

{"comment": "some text"}
```

# Cache Response Headers

## ETag

We always provide `ETag` response header and return `304 Not Modified` if it would match the one provided in `If-Match` request header.

**Why not Last-Modified-At?** We can provide this header, too, but ETag can be generated automatically (hash of response content) if date could not be resolved, it could also include milliseconds (which Last-Modified-At cannot) and we can easily change how or what we encode in it at any time.

## Vary

We always provide `Vary: Accept-Encoding, Accept, Accept-Language`, as all these three request headers can be used to form different responses for requests with same URL.

# Same Resource Structure

Structure of the resource should be the same for any request with same URL (URL and request headers that are provided in `Vary` response header, like `Accept-Encoding`).

In other words, we should not filter out any structure elements in the resource depending on context (like permissions).

For example, if resource has some public and some private properties, we cannot give different resource representation using same URL - we should explicitly ask only for public properties if we do not have permissions to view private ones. Preferrably we would split resource into separate parts so that properties would be divided (and possibly repeated) into public and private ones.

Another example would be filtering collection of resources. We should not filter by the permissions - we should explicitly ask only for resources that we have permission to access. If list would include resources that we do not have access to, we should return `403 Forbidden`. We should do this even if no resources were found (check permissions on filter, not on results themselves).

**Why?** This gives strict responsibilities and allows to structure application - first we check permissions, then we provide resource for given URL (and possibly other parameters, like `Accept` header and similar). This way we could even split responsibilities between separate servers - check permissions in gateway server which would proxy request to resource servers (that would not check permissions or know context at all). This also allows simpler caching mechanisms.

## A/B, Alpha and Beta Testing

### A/B Testing

A/B testing can usually be decided by frontend - we select one of A or B randomly and save selected variant in local storage.

In case there is a need for A/B testing in backend, we use same strategy as for feature flags.

### Feature Flags

Sometimes we need to get extra features that should not yet be available publicly.

These can be enabled only by some parameter publicly or require some access permissions.

For example, when adding new language, some translations can be not ready yet for public audience, but translators and other personnel needs to see the progress and check if all translations are correct inside the system itself.

Another example would be to turn on new feature only for some portion of our visitors.

As there is a need to enable feature this without "asking" the client (from server side), we cannot use any request headers or parameters for this.

In backends, we look for enabled features inside authentication token. See Authentication for more details.

We configure which features are enabled for which clients in Auth API, when issuing the token. We can choose to enable feature automatically for specific users or to only enable if some feature is explicitly asked when getting the token.

Cache should not be the problem, as Authentication header is used in this case, but make sure that any response formed with feature enabled would not be cached and returned to client without that specific feature.

# I18N

`Accept-Language` request header for language list and preferred language. Preferred language is first having highest quality. For language list quality is ignored, if more than zero. All codes that are not ISO-639 codes are ignored. `*` can be used to indicate all available languages - this should be used to provide preferred language without filtering languages in the content. For example `Accept-Language: en, *`.

`Content-Language` response header is provided with list of languages used in the resource representation.

For preferred language, if no language in the list can be chosen, system default is given (usually `en`).

Preferred language is used for system content translations, like error messages.

Language list is used for filtering content that is language-related. For example, getting titles for categories.

In resource representation, we prefer providing list with all possible translations and filtering it, not by just giving content in preferred language.

Use-cases:
- we have single language that interests us, there are dozens of languages available. We provide single language in `Accept-Language` header. Where structure gives list of translatable content, we would usually get single item (or zero - see below for considerations);
- we want to provide all information that was given by the user, if user created these translated resources. We either don't provide `Accept-Language` header at all, or add `, *` in the end of the header value;
- we want to synchronize and cache resource with all languages so that we would not need to cache them for each language separately. We do the same as in previous use-case.
 
Client should always provide at least one language that is guaranteed to be provided to avoid getting empty lists for translatable content.

Server should not provide any extra structures depending on value of the header - only filtering of already generated (fully translated) resource should be made.

Server should provide translated content for each of globally available languages in API (which should be documented or even provided by API call). If some translation is unavailable, server should choose most appropriate translation in another language (for example, fallback to English translations). This makes returned structure more predictable for the client - there is always at least one translatable resource as long as at least one supported language is passed in request.

We must provide `Accept-Language` in `Vary` response header for translatable resources (even if only error messages can be translated) even if one has not been passed in the request.

**Why?** If we return cacheable response without `Vary` header for request without `Accept-Language` header, same request with this header provided would still hit the cache, as cache layer would not know to use `Accept-Language` as part of a caching key.

# Pagination

## Always Paginated Lists

If we return list of resources, it should always be paginated.

## Pagination Strategies

We use one or both of these pagination strategies:
- `offset` and `limit` query parameters with `_metadata.total` provided in response structure;
- `after` and `limit` query parameters with `_metadata.next_after` provided in response structure.

### Offset and Limit

We do not use `page` parameter as `offset` lets more granular control over page sizes. For example, we can get first 5 resources and show them to the user and only then get rest of resources in pages of 100. This would show user the most relevant information fast and optimize page sizes later on. Also this would allow to see if rest of resource list is cached (if we order by date in descending order).

If `limit` is not provided, we use default one. If `offset` is not provided, we use `0`.

We provide both `limit` and `offset` used in the `_metadata` object inside response structure.

### After and Next After

For resources that can be created frequently and it is important in client side to get all resources (each of them once and only once), we use this strategy.

It applies for concrete resource, `order_by` and `order_direction` combinations.

If no `after` parameter is provided, we give first page of results (limited by `limit` or default page size).

If there is next page in resource list, we provide `next_after` parameter (as string) in `_metadata` object inside response structure. If value of this parameter is provided as `after` parameter in query string, we must return next page of resources.

**Why is this needed?** If resources are constantly created, they can be inserted into already accessed pages - resources are shifted and we get same results twice or do not get some results at all.

### Calculating Next After

We can use different strategies, depending on how resources can be added, deleted and modified and what ordering we use for resource list.

For example, if we order by date created, we can provide creation date for last item in the page, followed by last item's ID. We must also order not only by date created, but also by ID when getting the results, as creation date can be not unique for resources. When getting results, we parse creation date and ID of last given result and modify database query to give only the following resource.

We always use 0 as an offset in database queries when using this strategy.

# Authentication

## No Cookies

We do not use cookies in REST APIs.

This means that there should be no endpoint in the same domain as the API which would set the cookie in the response.

### If we need to use cookies

In some cases, we must use cookies to implement integration with partners. For example, if partner uses POST (or GET) request to return all users to same preconfigured URL, we might have no other way to relate which user any request belongs to.

In these cases we should put these endpoints into separate domains and make abstraction layer over it, so we could integrate with that partner without use of cookies.

## Auth2 for getting permissions

We use Auth2 protocol with Auth API to get tokens for use in REST APIs, when needed permissions are assigned at run-time using some sort of user authorization.

## JWT tokens

We use JWT tokens for communication between browser and resource servers, our partners and resource servers etc.

Tokens are issued by Auth API and validated by the resource server.

We can use custom tokens when they are long-term and related to some concrete resource server. For example, for communication between our servers, for communication from our partner servers to our servers (if use-case does not include getting user-related data and that user is not the owner of partner system).

