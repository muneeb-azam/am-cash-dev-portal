+++
title = "Implicit Grant Auth Flow"
menu = "main"
weight = 60
+++

{{% notice note %}}
Please note, the Implicit Grant Flow may become a deprecated flow in the future.
{{% /notice %}}

## Overview
In order to access a **Member** API from a web application, the client will need to implement the
**Implicit Grant Flow** OAuth 2.0 grant flow. The Implicit Grant
(defined in [RFC 6749, section 4.2](https://tools.ietf.org/html/rfc6749#section-4.2)) is a flow
where the browser receives an **Access Token** and optionally an **Id Token** from LoyaltyOne's Auth server and sends this to the web application (client-side server). The web app can then use this ```access_token``` to call the underlying API on behalf of the user. ```refresh_token```s are not permitted in this authorization flow.

![Implicit Grant Auth Flow (Sequence Diagram)](/images/implicit_grant_flow.png)

  1. The Authorization Server authenticates the user (via the browser).
    - _The first time, a user goes through this flow, a consent page may be shown where the permissions are listed that would be given to the client._

  2. After successful authentication, the authorization server redirects the user back to the web application (specifically to the redirect_uri) with an Access Token as a hash parameter (`access_token`).

  3. After receiving the access_token, the web application can then use it to call the API on behalf of the user.

## Prerequisites
+ Auth0 client Id. Provided by LoyaltyOne Partner Integration Team.

## 1. Get the User's Authorization
The browser makes a GET request on behalf of the user to the URI [https://loyaltyone-sandbox-pin.auth0.com/authorize](https://loyaltyone-sandbox-pin.auth0.com/authorize) with the following query parameters mentioned below.
The user will then get redirected to the AirMiles Single Sign-On page where they will be asked to enter their credentials.

{{% notice note %}}
This request should **NOT** be embedded in the client application (via iframe or any other means) to ensure that the collector's credentials are not exposed to parties outside the authorization server.
{{% /notice %}}

```html
<a href="
  https://loyaltyone-sandbox-pin.auth0.com/authorize?
  connection=CONNECTION&
  scope=SCOPE&
  response_type=code&
  nonce=NONCE&
  client_id=YOUR_CLIENT_ID&
  redirect_uri=https://myclientwebapp.com/callback
">
  Sign In
</a>
```

**Where:**

|Parameter|Description|Value|
| ---------------:|----------------:|----------------:|
|```connection```**(required)**|The name of the identity provider configured to your application.|```member-pin-idp-recaptcha```|
|```scope```**(optional)**|The scopes that you want to request authorization for. These must be separated by a space. Please note, that a ```refresh_token``` will not be granted even if ```offline_access``` is specified in the scope for this flow.||
|```response_type```**(required)**|Denotes the kind of credential that the Authorization server will return.|```token id_token```|
|```nonce```**(required)**|A cryptographically random nonce which allows your application to verify that the ID Token response matches your authentication request. For more information: https://auth0.com/docs/api-auth/tutorials/nonce||
|```state```**(optional)**|A random string from your to make sure that the response belongs to a request that was initiated by the application. Mitigates CSRF attacks. For more information: https://auth0.com/docs/protocols/oauth2/oauth-state||
|```client_id```**(required)**|Your web application's identifier. This will be provided by LoyaltyOne.||
|```redirect_uri```**(required)**|The URL to which the authorization server will redirect the browser after  authorization has been granted to the user. The Access Token (used in step 2) will be available in the ```access_token``` hash parameter. This URL must be specified as a valid callback URL and communicated to LoyaltyOne when the ```client_id``` is being generated for your web app.||

**For example:**

### Sample Request:
```html
<a href="https://loyaltyone-sandbox-pin.auth0.com/authorize?client_id=YOUR_CLIENT_ID&response_type=token%20id_token&redirect_uri=http://myclientapp.com/callback&scope=openid&connection=member-pin-idp-recaptcha&nonce=4aEVKxXZ-YrkKrfX&state=jYUaLiLrPYpqbCUa">
  Sign In
</a>
```

If authentication is successful a JSON response (like the one shown below) is returned.

### Sample Response:
```json
{
  "access_token":"eyJz93a...k4laUWw",
  "token_type":"Bearer",
  "expires_in":86400
}
```

{{% notice tip %}}
The purpose of this call is to obtain consent from the user to invoke the API to do certain things (if any specified in `scope`) on behalf of the user.
{{% /notice %}}

 ## 2. Call the underlying API
Once the `access_token` has been obtained it can be used to make calls to the underlying API by including it in
the "Authorization" header when making the corresponding request.
As an example, click **'Authorize'** in the [Swagger Spec](http://developer.airmiles.ca/member-banner/docs/swagger.html?url=https://sandbox.api.loyalty.com/v1/api-docs#/Member), then form a `Bearer id_token` string like below and enter this as the `api_key` in the `value` field.

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
[Auth0 - How to implement the Implicit Grant](https://auth0.com/docs/api-auth/tutorials/implicit-grant)
