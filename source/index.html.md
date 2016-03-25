---
title: Internal Platform API

includes:
  - change_history

search: true
---
# Introduction

<aside class="notice">
Draft 2, 2015-09-15
</aside>

The Internal Platform API is an extension to the standard Loyalty Platform API providing functionality which exists in the platform but is either not released or not exposed to other users of the platform.

Readers must be familiar with the API Basics of the Loyalty Platform API document in order to understand the descriptions of resources herein and understand how to make well-formed API calls that use them.

Common API behaviour including descriptions of how to read lists in pages, apply search parameters and so-on are all described in the Loyalty Platform API document.



# Queue API Gateway

The Queue API Gateway allows platform requests to be sent via Amazon's SQS and processed asynchronously, facilitating simple rate-limiting.

## Messages

### PlatformRequest `::message::platform_request`

```json-doc
{
  // A unique value within the scope of this queue. This can be used to look up
  // the persisted representation of this message via QueueProcessingRequest and
  // to look up the result of processing this message via QueueProcessingResult.
  //
  "message_reference": "{string}",

  // Indicates whether this message should be processed or just persisted for
  // possible future processing.
  //
  "message_action":    "{reference[process|defer]}",

  // High precision datetime when this message was queued.
  //
  "queued_at":         "{datetime}",

  // The Outlet id of the intended Caller for these requests. This is mapped to
  // a caller_identity using an internal mapping.
  //
  "caller_outlet_id":  "{::resource::outlet.id}",

  // (Optional) Extra info attached to this message which will be persisted in
  // QueueProcessingRequest.
  //
  "info":              {
    // Arbitrary key/value pairs, where values are strings, booleans, arrays
    // or integers.
  },

  // An Array of platform_requests.
  //
  "platform_requests": [
    {::type::platform_request},
    ...
  ]
}
```

<aside class="warning">
The entire message object must be less than 64KiB in size.
</aside>
This is the message format for sending requests to the platform via an Amazon SQS queue.

# QueueProcessing API

## Data Types


### PlatformRequest `::type::platform_request`

```json-doc
{

  // Platform resource to issue this request to.
  //
  "resource":            "{string}",

  // (Optional) Version of the resource to issue this request to, defaulting to
  // 1.
  //
  "version":             {integer},

  // Platform action to use for the request.
  //
  "action":              "{reference[create|show|list|update|delete]}",

  // (Optional) Query parameters to apply to the request. This can only be
  // specified if the action is list.
  //
  "query_parameters":    {

    // The following parameters are all optional, they are defaulted to the
    // default values specified by the resource that is being listed. See the
    // 'Listing, pagination, searches and filters' section of the Platform API
    // (linked below) for further information on what these parameters mean.

    "offset":    {integer},

    "limit":     {integer},

    "sort":      "{string}",

    "direction": "{reference[asc|desc]}",

    "search":    {
      // Arbitrary key/value pairs, where values are search Strings supported by
      // the resource.
    },

    "filter":    {
      // Arbitrary key/value pairs, where values are filter Strings supported by
      // the resource.
    },

    "_embed":    [
      "{string}",
      ...
    ],

    "_reference":    [
      "{string}",
      ...
    ]

  },

  // (Optional) Extra options which affect the handling of this request.
  //
  "request_options":     {

    // (Optional) Specify the locale for this request.
    //
    "locale":        "{rfc1766_tag}",

    // (Optional) Only applies when the action is show or list. This will
    // request that resources are returned as they were at the specified
    // date-time.
    //
    "dated_at":      "{datetime}",

    // (Optional) Only applies when the action is create. This will request that
    // the resource is created as if it started to exist at a historical
    // date-time.
    //
    "dated_from":    "{datetime}",

    // (Optional) Only applies when the action is create or delete. Indicates
    // that this request may have been sent before.
    //
    "deja_vu":       "{boolean}",

    // (Optional) Only applies when the action is create. This will request
    // that the resource is created with the specified id. Use of this option
    // requires appropriate permissions on the Caller that makes this request.
    //
    "resource_uuid": "{uuid}"
  },

  // (Optional) Resource identifier to be used for the request. This must be
  // specified if the action is show, update or delete.
  //
  "resource_identifier": "{string}",

  // (Optional) The request payload. This must only be specified if the action
  // is create or update.
  //
  "body":                {
    // Arbitrary key/value pairs.
  }

}
```

<br>
More information on `query_parameters` can be found in the [Listing, pagination, searches and filters](https://github.com/LoyaltyNZ/awg/blob/master/reference/platform_api.md#lppsf) section.



### PlatformResult `::type::platform_result`

```json-doc
{

  // Interaction ID of the result which can be used to find the result via the
  // Log resource.
  //
  "interaction_id": "{uuid}",

  // Whether the result succeeded or led to an error.
  //
  "status": "{reference[success|error]}",

  // The id of the resulting Error if one occurs. This will be null if no error
  // occurred or if the Error did not have an id.
  //
  "error_id": "{::resource::error.id}",

  // Array of errors that occurred in processing the platform result.
  //
  "errors": [
    {::type::error_primitive},
    ...
  ]

}
```



## Resources

### QueueProcessingRequest `::resource::queue_processing_request`

```json-doc
{
  "kind":         "QueueProcessingRequest",
  "id":           "{uuid}",
  "created_at":   "{datetime}",
  "secured_with": {
    "participant_id": "{uuid}"
  },

  // A unique value within the scope of this queue. This can be used to look up
  // the persisted representation of this message via QueueProcessingRequest and
  // to look up the result of processing this message via QueueProcessingResult.
  //
  "message_reference": "{string}",

  // Indicates whether this message has been processed or just persisted for
  // possible future processing. The state is derived from the message_action
  // given when creating a QueueProcessingRequest.
  //
  "state":             "{reference[processed|deferred]}",

  // High precision datetime when this message was queued.
  //
  "queued_at":         "{datetime}",

  // (Optional) Extra info attached to the message.
  //
  "info":              {
    // Arbitrary key/value pairs, where values are strings, booleans, arrays
    // or integers.
  },

  // An Array of platform_requests.
  //
  "platform_requests": [
    {::type::platform_request},
    ...
  ],

  // An Array of errors resulting from the creation of this
  // QueueProcessingRequest. This does not include errors that arise from
  // processing the platform_requests.
  //
  "payload_errors": [
    {::type::error_primitive),
    ...
  ]
}
```



### Interface

| HTTP method | Endpoint                          | Result |
|-------------|-----------------------------------|--------|
| `POST`      | /queue_processing_requests/       | Create new QueueProcessingRequest instance |
| `GET`       | /queue_processing_requests/       | Obtain list of QueueProcessingRequest representations |
| `GET`       | /queue_processing_requests/{uuid} | Obtain representation of identified QueueProcessingRequest instance |

* The `participant_id` session identity is recorded against resource instances when created as well as the `participant_id` in the `caller_identity` if present. Sessions can only access data where one of these `participant_id`s matches an ID in the session's `authorised_participant_ids`.
* The `GET` 'list' call accepts [common query string parameters](#lppsf).
* No additional sort fields are defined.
* The following search fields are defined, all case sensitive with no wildcards unless stated:
  * `message_reference={string}` - return all QueueProcessingRequest instances containing a matching `message_reference`
  * `state={string}` - return all QueueProcessingRequest instances with a matching `state`
  * `participant_id={uuid}` - return all QueueProcessingRequests created under a [Session](#session.resource) that has an `identity` section containing a matching [Participant](#participant.resource) UUID
  * `participant_ids={uuid}[,{uuid},...]` - return all QueueProcessingRequests created under a [Session](#session.resource) that has an `identity` section containing any items in a comma-separated list of [Participant](#participant.resource) UUIDs
* Allowed filter strings are the same as allowed search strings, except they exclude, rather than include, matching resource instances.
* No resources are embeddable.

For the `POST` 'create' call, send the following JSON payload:

```json-doc
{
  // A caller-defined reference which can be used to trace a queue message to
  // its persisted result.
  //
  "message_reference": "{string}",

  // Indicates whether this message should be processed or just persisted for
  // possible future processing. The message_action will dictate the value
  // of the state in the returned representation.
  //
  "message_action":    "{reference[process|defer]}",

  // High precision datetime when this message was queued.
  //
  "queued_at":         "{datetime}",

  // (Optional) Extra info attached to the message.
  //
  "info":              {
    // Arbitrary key/value pairs, where values are strings, booleans, arrays
    // or integers.
  },

  // An Array of platform_requests.
  //
  "platform_requests": [
    {::type::platform_request},
    ...
  ]
}
```



### QueueProcessingResult `::resource::queue_processing_result`

```json-doc
{
  "kind":         "QueueProcessingResult",
  "id":           "{uuid}",
  "created_at":   "{datetime}",
  "secured_with": {
    "participant_id": "{uuid}"
  },

  // Indicates whether the associated QueueProcessingRequest was successfully
  // parsed and executed. If parsing fails or any errors occur in processing
  // the platform_requests this field will be set to error, otherwise it will
  // be success.
  //
  "status":                      "{reference[success|error]}",

  // The id of the QueueProcessingRequest which led to the creation of this
  // QueueProcessingResult.
  //
  "queue_processing_request_id": "{::resource::queue_processing_request.id}",

  // An Array of results from executing the platform_requests in the associated
  // QueueProcessingRequest. The platform_requests are executed in the order
  // they are given and the platform_results are returned in that same order. If
  // executing a platform_request results in an error the subsequent requests
  // will not be executed and there will be no platform_results for those
  // requests.
  //
  "platform_results":            [
    // Fields are inline
    {::type::platform_result}
  ]
}
```



### Interface

| HTTP method | Endpoint                         | Result |
|-------------|----------------------------------|--------|
| `GET`       | /queue_processing_results/       | Obtain list of QueueProcessingResult representations |
| `GET`       | /queue_processing_results/{uuid} | Obtain representation of identified QueueProcessingResult instance |

* The `participant_id` session identity is recorded against resource instances when created as well as the `participant_id` in the `caller_identity` if present. Sessions can only access data where one of these `participant_id`s matches an ID in the session's `authorised_participant_ids`.
* The `GET` 'list' call accepts [common query string parameters](#lppsf).
* No additional sort fields are defined.
* The following search fields are defined, all case sensitive with no wildcards unless stated:
  * `status={string}` - return all QueueProcessingResult instances containing a matching `status`
  * `queue_processing_request_id={string}` - return all QueueProcessingResult instances with a matching `queue_processing_request_id`
  * `participant_id={uuid}` - return all QueueProcessingResults created under a [Session](#session.resource) that has an `identity` section containing a matching [Participant](#participant.resource) UUID
  * `participant_ids={uuid}[,{uuid},...]` - return all QueueProcessingResults created under a [Session](#session.resource) that has an `identity` section containing any items in a comma-separated list of [Participant](#participant.resource) UUIDs
* Allowed filter strings are the same as allowed search strings, except they exclude, rather than include, matching resource instances.
* No resources are embeddable.



# Metadata API

## Data Types

### TransactionData `::type::transaction_data`

This Type encapsulates data related to transactions originating from Points Pro and Loyalty Host transaction files.

```json-doc
{
  // The NZD amount for a transaction.
  "transaction_amount": "{string}"
}
```

The `transaction_amount` is a string, to avoid rounding error issues. The amount described may be positive or negative, integer or fractional. Only digits and zero or one "." are allowed, to separate the integer and fractional components. An optional "+" or "-" at the start of the string is allowed to indicate positive or negative values, respectively; if this is absent, a positive value is assumed.
