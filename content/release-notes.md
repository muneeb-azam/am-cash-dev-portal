+++
title = "Release Notes"
weight = 30
+++
---

##### Release v1.0.57(58) (13/12/2018)

- Improved logging capabilities, enabling message logs to be parsed as JSON.

##### Release v1.0.37 (02/10/2018)

- HTTP Status for responses now reflect the `responseCode` field in the response body.

    > Example: `HTTP 401` will be returned if the 'Collector's' number is invalid.

- Standardized response body to return...

    > `responseCode`: 4 digit, 0 left padded numeric string response code.

    > `message`: Human readable message describing the response.

- `responseCode 0035` _(i.e., bad request data)_

    > A new `responseCode` that identifies a bad request parameter.

    > The `message` field will contain detailed information about the offending request parameter.
