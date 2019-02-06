+++
title = "Client Credentials Auth Flow"
menu = "main"
weight = 50
+++

## Overview
In order to access a **Partner** API through a machine to machine process, the client will need to implement the
**Client Credential** OAuth 2.0 grant flow. The Client Credential Grant
(defined in [RFC 6749, section 4.4](https://tools.ietf.org/html/rfc6749#section-4.4)) is a flow
where the client machine uses a **Client Secret/Password** to receive an **Access Token** from LoyaltyOne's Auth server. The client then uses this `access_token` to make calls to the underlying API.`refresh_token`s are not permitted in this authorization flow.

![Client Credentials Authorization (Sequence Diagram)](/images/client_credentials_authorization_flow.png)

{{<mermaid align="center">}}
graph LR
    A[Trusted Client-App]--1-->B[Authentication Server]
    B--2-->A
    A--3-->C[L1 Protected-API]
{{< /mermaid >}}

  1. The client machine initiates the flow by making a request for the `access_token` using its `client_secret`.

  2. The authentication server verifies the `client_secret` and `client_id` combination.

    - _If valid, authorization is granted by issuing an `access_token`._

  3. After receiving back the `access_token`, the client machine can then use it to make calls to the API.

## Prerequisites
+ Auth0 client Id. Provided by LoyaltyOne Partner Integration Team.
+ Auth0 client secret. Provided by LoyaltyOne Partner Integration Team, through a secure vault.

## 1. Get the Client's Authorization
The client makes a POST request to the URI [https://loyaltyone-sandbox-pin.auth0.com/oauth/token](https://loyaltyone-sandbox-pin.auth0.com/oauth/token) with the following json body parameters mentioned below.

```json
{
  "client_id":"YOUR_CLIENT_ID",
  "client_secret":"YOUR_CLIENT_SECRET",
  "scope":"SCOPE",
  "grant_type":"client_credentials"
}
```

**Where:**

|Parameter|Description|Value|
| ---------------:|----------------:|----------------:|
|```client_id```**(required)**|Your machine to machine application identifier. This will be provided by LoyaltyOne.||
|```client_secret```**(required)**|Your machine to machine client secret. This will be provided by LoyaltyOne.||
|```scope```**(optional)**|The scopes that you want to request authorization for. These must be separated by a space. Please note, that a refresh_token will not be granted even if offline_access is specified in the scope for this flow.||
|```grant_type```**(required)**|Denotes the kind of credential used to authenticate.|```client_credentials```|

For example:

### Sample Request:
```bash
curl --request POST \
  --url https://loyaltyone-sandbox-pin.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"YOUR_CLIENT_ID","client_secret":"YOUR_CLIENT_SECRET","grant_type":"client_credentials", "scope":"read:appointments"}'
```

### Sample Response:
```json
{
  "access_token":"eyJz93a...k4laUWw",
  "token_type":"Bearer",
  "expires_in":86400
}
```

{{% notice info %}}
The purpose of this call is to authenticate the client and grant authorization via `access_token`. If authentication is successful a JSON response is returned like the following:
{{% /notice %}}

 ## 2. Call the underlying API
Once the `access_token` has been obtained it can be used to make calls to the underlying API by including it in
the "Authorization" header when making the corresponding request.
As an example, click `**Authorize**` in the [swagger spec](http://developer.airmiles.ca/member-banner/docs/swagger.html?url=https://sandbox.api.loyalty.com/v1/api-docs#/Member), then form a `Bearer id_token` string like below and enter this as the `api_key` in the `value` field.

```html
Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlF6SkdNemd4UVRFd01qVXhNRGszUXpVelEwSkVPREEyUVVKRU16UkVPVGxCUWpORk1EaEVNQSJ9.eyJpc3MiOiJodHRwczovL2xveWFsdHlvbmUtZGV2LmF1dGgwLmNvbS8iLCJzdWIiOiJhdXRoMHx0ZXN0NTY3QGdtYWlsLmNvbSIsImF1ZCI6WyJodHRwczovL2xveWFsdHlvbmUtZGV2LmF1dGgwLmNvbS91c2VyaW5mbyJdLCJhenAiOiJjbGlHeHFXTkZPeWZmNGJINEtydUNDRkVrbUhyQ1NLUiIsImV4cCI6MTQ4NDYzODcxNCwiaWF0IjoxNDg0NjAyNzE0LCJzY29wZSI6Im9wZW5pZCJ9.LSGfN8TRQZTjeeEp_J_7rb2H2m91Pf-zU9UfHzHbd-rb_htlR3sEhZ2wP0s6l8kCiA8xuEcCR89mnDn0g52lKzrsre-TbGsJWKI1Adf7dmBzj7B_JAHw5MrrMQfPS4HmylJczsDzXWKtpRgbTr5ygg3TYBVgjTZU_Cd-NHd4F1VI6FUfYMLOfRpKm-NIlOTeHgOcsAd8ZyAyTk9SkbnWc0g9Ehh0nBaJ4hYimEYB6k26K64uIXNj4gYBhdfBmzH7W0jGBfQ4coVojh_-guP8Ii1p09AmXEQyv6abpMHNTSe7749kSa76zTu8-Sg8OsXPjbzRYFzqXWI5sAmw_8o5kQ
```

#### Leveraging the refresh token functionality

{{% notice warning %}}
'Refresh Tokens' are not permitted in this authorization flow.
{{% /notice %}}

## References:
[AirMiles Developer Portal](http://developer.airmiles.ca/authorization.html)
[Auth0 - Overview](https://auth0.com/docs/getting-started/overview)
[Auth0 - Execute a machine to machine Client Credential Flow](https://auth0.com/docs/applications/machine-to-machine)
