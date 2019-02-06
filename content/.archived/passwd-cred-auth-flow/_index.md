+++
title = "Password Credentials Auth Flow"
menu = "main"
weight = 70
+++

## Overview
In order to access a **Member** API from a **trusted application**, the client will need to implement the
**Resource Owner Password Credentials Grant** OAuth 2.0 grant flow. The Authorization Code Grant
(defined in [RFC 6749, section 4.3](https://tools.ietf.org/html/rfc6749#section-4.3)) is a flow
where a resource owner provides a **trusted application** with their credentials (username and password).
The application then requests an `access_token` from the authorization server using the resource
owners credentials. An `id_token` and `refresh_token` can optionally be requested.
The web app can then use the `access_token` to call the underlying API on behalf of the user.

{{<mermaid align="center">}}
graph LR
    A[Resource Owner]--1-->B[Trusted Client-App]
    B--2-->C[Authorization Server]
    C--3-->B
    B--4-->D[L1 Protected-API]
{{< /mermaid >}}

  1. The user initiates the flow by providing the trusted client application their username and password.

    - _How this is done will vary depending on the implementation of the trusted application (for example, a login page)._

  2. The trusted application forwards the user credentials to the authorization server in order to obtain an `access_token`.

  3. The Authorization Server then authenticates the user credentials.

    - _After successful authentication, the authorization server responds with an `access_token`._

  4. After receiving back the `access_token`, the trusted application can then use it to call the API on behalf of the user.

## Prerequisites
+ Auth0 client Id. Provided by LoyaltyOne Partner Integration Team.

### 1. Get the Resource Owner's Credentials
The resource owner passes their credentials (username and password) to the trusted application.

### 2. Retrieve an Access Token via Resource Owner's Credentials
Makes a POST request to the URI [https://loyaltyone-sandbox-pin.auth0.com/oauth/token](https://loyaltyone-sandbox-pin.auth0.com/oauth/token) with the following json body parameters mentioned below.

```json
{
  "client_id":"YOUR_CLIENT_ID",
  "scope":"SCOPE",
  "grant_type":"password",
  "username":"USERNAME",
  "password":"PASSWORD",
}
```

**Where:**

|Parameter|Description|Value|
| ---------------:|----------------:|----------------:|
|```client_id```**(required)**|Your machine to machine application identifier. This will be provided by LoyaltyOne.||
|```scope```**(optional)**|The scopes that you want to request authorization for. These must be separated by a space. Please note, that a refresh_token will not be granted even if offline_access is specified in the scope for this flow.||
|```grant_type```**(required)**|Denotes the kind of credential used to authenticate.|```password```|
|```username```**(required)**|The username of the resource owner that is being authenticated||
|```password```**(required)**|The password of the resource owner that is being authenticated||

**For example:**

### Sample Request:
```bash
curl --request POST \
  --url https://loyaltyone-sandbox-pin.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"YOUR_CLIENT_ID","client_secret":"YOUR_CLIENT_SECRET","grant_type":"password", "scope":"read:appointments","username":"authuser","password":"authpassword"}'
```

If authentication is successful a JSON response is returned like the following:

### Sample Response:
```json
{
  "access_token":"eyJz93a...k4laUWw",
  "token_type":"Bearer",
  "expires_in":86400
}
```

{{% notice note %}}
The purpose of this call is to authenticate the client and grant authorization via `access_token`.
{{% /notice %}}

### 3. Call the underlying API
Once the `access_token` has been obtained, it can be used to make calls to the underlying API by including it in the "Authorization" header when making the corresponding request.
As an example, click **'Authorize'** in the [Swagger Spec](http://developer.airmiles.ca/member-banner/docs/swagger.html?url=https://sandbox.api.loyalty.com/v1/api-docs#/Member), then form a `Bearer id_token` string like below and enter this as the `api_key` in the `value` field.

```html
Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlF6SkdNemd4UVRFd01qVXhNRGszUXpVelEwSkVPREEyUVVKRU16UkVPVGxCUWpORk1EaEVNQSJ9.eyJpc3MiOiJodHRwczovL2xveWFsdHlvbmUtZGV2LmF1dGgwLmNvbS8iLCJzdWIiOiJhdXRoMHx0ZXN0NTY3QGdtYWlsLmNvbSIsImF1ZCI6WyJodHRwczovL2xveWFsdHlvbmUtZGV2LmF1dGgwLmNvbS91c2VyaW5mbyJdLCJhenAiOiJjbGlHeHFXTkZPeWZmNGJINEtydUNDRkVrbUhyQ1NLUiIsImV4cCI6MTQ4NDYzODcxNCwiaWF0IjoxNDg0NjAyNzE0LCJzY29wZSI6Im9wZW5pZCJ9.LSGfN8TRQZTjeeEp_J_7rb2H2m91Pf-zU9UfHzHbd-rb_htlR3sEhZ2wP0s6l8kCiA8xuEcCR89mnDn0g52lKzrsre-TbGsJWKI1Adf7dmBzj7B_JAHw5MrrMQfPS4HmylJczsDzXWKtpRgbTr5ygg3TYBVgjTZU_Cd-NHd4F1VI6FUfYMLOfRpKm-NIlOTeHgOcsAd8ZyAyTk9SkbnWc0g9Ehh0nBaJ4hYimEYB6k26K64uIXNj4gYBhdfBmzH7W0jGBfQ4coVojh_-guP8Ii1p09AmXEQyv6abpMHNTSe7749kSa76zTu8-Sg8OsXPjbzRYFzqXWI5sAmw_8o5kQ
```

#### Leveraging the refresh token functionality
When the `access_token` expires, a new one can be retrieved by the client on behalf of the collector using the `refresh_token` from the Step 3.
This can be achieved by making a HTTP POST request to the URI [https://loyaltyone-sandbox-pin.auth0.com/oauth/token](https://loyaltyone-sandbox-pin.auth0.com/oauth/token) with a JSON payload in the body as seen below.

```bash
curl --request POST \
  --url 'https://loyaltyone-sandbox-pin.auth0.com/oauth/token' \
  --header 'content-type: application/json' \
  --data '{ "grant_type": "refresh_token", "client_id": "YOUR_CLIENT_ID", "refresh_token": "YOUR_REFRESH_TOKEN" }'
```

## References:
[AirMiles Developer Portal](http://developer.airmiles.ca/authorization.html)
[Auth0 - Overview](https://auth0.com/docs/getting-started/overview)
[Auth0 - Execute a Resource Owner Password Grant Flow](https://auth0.com/docs/api-auth/tutorials/password-grant)
