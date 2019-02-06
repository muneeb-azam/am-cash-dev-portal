+++
title = "Auth Code/Grant Flow"
menu = "main"
weight = 30
+++

## Overview
In order to utilize Air Miles' library of API calls, used to access everything from specific customer (i.e., 'Member') details,
to application banners and other integration resources, the partner (i.e., 'You!'), will need to generate and use a valid `Authorization Code`,
which can be obtained by visiting our `Auth Server` and requesting access via the OAuth 2.0 framework.

The 'Auth Code/Grant Flow' for 'OAuth 2.0' code OAuth 2.0 grant flow.

The Authorization Code Grant (defined in [RFC 6749, section 4.1](https://tools.ietf.org/html/rfc6749#section-4.1)) is a flow
where the browser receives an **Authorization Code** from LoyaltyOne's Auth server and sends this
to the web application (client-side server). The web app will then interact with the auth server
and exchange the Authorization Code for an `access_token`, and optionally an `id_token`
and a `refresh_token`. The web app can then use this `access_token` to call the
underlying API on behalf of the user.

![Authorization Code Grant Flow (Sequence Diagram)](/images/authorization_code_grant_flow.png)



{{<mermaid align="center">}}
graph LR
    linkStyle default interpolate basis
    A[Web Browser]--1-->B[Auth Server]
    B--2-->A
    A--3.1-->C[Web App]
    C--3.2-->B
    C--4-->D[L1 Protected API]
{{< /mermaid >}}

  1. The Authorization Server authenticates the user (via the browser). The first time, a user goes through this
  flow, a consent page may be shown where the permissions are listed that would be given to the client.

  2. After successful authentication, the authorization server redirects the user back to the web application
  (specifically to the redirect_uri) with an Authorization Code in the query string (`code`).

  3. The web app then sends the Authorization Code to the Loyalty One server in a POST request and asks to exchange it for
   an access_token, id_token and a refresh_token (optional) via a prescribed endpoint.

  4. Finally, after receiving back the access_token the web application can then use it to call the API on behalf of the user.

## Prerequisites
+ Auth0 client Id. Provided by LoyaltyOne Partner Integration Team.
+ Auth0 client secret. Provided by LoyaltyOne Partner Integration Team, through a secure vault.

### 1. Get the User's Authorization
The browser makes a GET request on behalf of the user to this URI ([https://sandbox.loyalty.com/v1/authorize](https://sandbox.loyalty.com/v1/authorize)) with the following query parameters mentioned below.

{{% notice note %}}
This request should **not** be embedded in the client application (via iframe or any other means) to ensure that the collector's credentials are not exposed to parties outside the authorization server.
{{% /notice %}}

```json
https://sandbox.loyalty.com/v1/authorize?
    scope=SCOPE&
    response_type=code&
    client_id=YOUR_CLIENT_ID&
    redirect_uri=https://myclientapp.com/callback
```

**Where:**

|Parameter|Description|Value|
| ---------------:|----------------:|----------------:|
|```scope```**(optional)**|The scopes that you want to request authorization for. These must be separated by a space. Include value ```offline_access``` to get a refresh token.||
|```response_type```**(required)**|Denotes the kind of credential that the Authorization server will return.|```code```|
|```client_id```**(required)**|Your web application's identifier. This will be provided by LoyaltyOne.||
|```redirect_uri```**(required)**|The URL to which the authorization server will redirect the browser after authorization has been granted to the user. The Authorization code (used in step 2) will be available in the ```code``` URL parameter. This URL must be specified as a valid callback URL and communicated to LoyaltyOne when the ```client_id``` is being generated for your web app.||

**For example:**

#### Sample Request:
```html
<a href="https://sandbox.loyalty.com/v1/authorize?scope=appointments%20contacts&response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=https://myclientapp.com/callback">
  Sign In
</a>
```

{{% notice tip %}}
The purpose of this call is to obtain consent from the user to invoke the API to do certain things (if any specified in `scope`) on behalf of the user. The LoyaltyOne Auth server then authenticates the user.
{{% /notice %}}

### 2. Exchange the Authorization Code for an Access Token
After extracting the Authorization Code `code` out of the redirect URL, the web app must exchange it for an
`access_token` that can later be used to call the underlying API. Use the Authorization Code `code` from the
previous step to make a `POST` request to this *https://sandbox.api.loyalty.com/v1/oauth/token* URI where:


|Parameter|Description|Value|
| ---------------:|----------------:|----------------:|
|```grant_type```**(required)**|This must have the value as ```authorization_code```.||
|```client_id```**(required)**|Your web application's identifier. This will be provided by LoyaltyOne.|
|```code```**(required)**|The value of the ```code``` extracted out from the response returned in Step 3.||
|```redirect_uri```**(required)**|The URL to which the authorization server will redirect the browser after authorization has been granted to the user. The value of this URL must exactly match the ```redirect_uri``` passed in Step 3.||

##### cURL
```bash
curl --request POST \
  --url 'https://sandbox.loyalty.com/v1/oauth/token' \
  --header 'content-type: application/json' \
  --data '{"grant_type":"authorization_code","client_id": "YOUR_CLIENT_ID","code": "YOUR_AUTHORIZATION_CODE","redirect_uri": "x`x://myclientapp.com/callback", }'
```

##### Java
```java
HttpResponse<String> response = Unirest.post("https://sandbox.loyalty.com/v1/oauth/token")
  .header("content-type", "application/json")
  .body("{\"grant_type\":\"authorization_code\",\"client_id\": \"YOUR_CLIENT_ID\",\"code\": \"YOUR_AUTHORIZATION_CODE\",\"redirect_uri\": \"https://myclientapp.com/callback\", }")
  .asString();
```

##### Swift 3
```js
import Foundation

let headers = ["content-type": "application/json"]

let postData = NSData(data: "{"grant_type":"authorization_code","client_id": "YOUR_CLIENT_ID","code": "YOUR_AUTHORIZATION_CODE","redirect_uri": "https://myclientapp.com/callback", }".dataUsingEncoding(NSUTF8StringEncoding)!)

var request = NSMutableURLRequest(URL: NSURL(string: "https://sandbox.loyalty.com/v1/oauth/token")!,
                                        cachePolicy: .UseProtocolCachePolicy,
                                    timeoutInterval: 10.0)
request.HTTPMethod = "POST"
request.allHTTPHeaderFields = headers
request.HTTPBody = postData

let session = NSURLSession.sharedSession()
let dataTask = session.dataTaskWithRequest(request, completionHandler: { (data, response, error) -> Void in
  if (error != nil) {
    println(error)
  } else {
    let httpResponse = response as? NSHTTPURLResponse
    println(httpResponse)
  }
})

dataTask.resume()
```

##### Objective-C
```objc
#import <Foundation/Foundation.h>

NSDictionary *headers = @{ @"content-type": @"application/json" };

NSData *postData = [[NSData alloc] initWithData:[@"{"grant_type":"authorization_code","client_id": "YOUR_CLIENT_ID","code": "YOUR_AUTHORIZATION_CODE","redirect_uri": "https://myclientapp.com/callback", }" dataUsingEncoding:NSUTF8StringEncoding]];

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://sandbox.loyalty.com/v1/oauth/token"]
                                                       cachePolicy:NSURLRequestUseProtocolCachePolicy
                                                   timeoutInterval:10.0];
[request setHTTPMethod:@"POST"];
[request setAllHTTPHeaderFields:headers];
[request setHTTPBody:postData];

NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request
                                            completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
                                                if (error) {
                                                    NSLog(@"%@", error);
                                                } else {
                                                    NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *) response;
                                                    NSLog(@"%@", httpResponse);
                                                }
                                            }];
[dataTask resume];
```

The response contains `access_token`, `refresh_token`, `id_token`, and `token_type` values, for example:

```json
{
  "access_token": "eyJz93a...k4laUWw",
  "refresh_token": "GEbRxBN...edjnXbL",
  "id_token": "eyJ0XAi...4faeEoQ",
  "token_type": "Bearer"
}
```

{{% notice note %}}
The `refresh_token` will only be present in the response if `scope=offline_access` was included in the request as part of _"Step 3"_.
{{% /notice %}}

## 3. Call the underlying API

{{% notice tip %}}
Once the `access_token` has been obtained it can be used to make calls to the underlying API by including it in
the 'Authorization' header when making the corresponding request.
As an example, click **"Authorize"** in the [swagger spec](http://developer.airmiles.ca/member-banner/docs/swagger.html?url=https://sandbox.api.loyalty.com/v1/api-docs#/Member)
, then form a `Bearer id_token` string like below and enter this as the `api_key` in the `value` field.
{{% /notice %}}

```html
Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IlF6SkdNemd4UVRFd01qVXhNRGszUXpVelEwSkVPREEyUVVKRU16UkVPVGxCUWpORk1EaEVNQSJ9.eyJpc3MiOiJodHRwczovL2xveWFsdHlvbmUtZGV2LmF1dGgwLmNvbS8iLCJzdWIiOiJhdXRoMHx0ZXN0NTY3QGdtYWlsLmNvbSIsImF1ZCI6WyJodHRwczovL2xveWFsdHlvbmUtZGV2LmF1dGgwLmNvbS91c2VyaW5mbyJdLCJhenAiOiJjbGlHeHFXTkZPeWZmNGJINEtydUNDRkVrbUhyQ1NLUiIsImV4cCI6MTQ4NDYzODcxNCwiaWF0IjoxNDg0NjAyNzE0LCJzY29wZSI6Im9wZW5pZCJ9.LSGfN8TRQZTjeeEp_J_7rb2H2m91Pf-zU9UfHzHbd-rb_htlR3sEhZ2wP0s6l8kCiA8xuEcCR89mnDn0g52lKzrsre-TbGsJWKI1Adf7dmBzj7B_JAHw5MrrMQfPS4HmylJczsDzXWKtpRgbTr5ygg3TYBVgjTZU_Cd-NHd4F1VI6FUfYMLOfRpKm-NIlOTeHgOcsAd8ZyAyTk9SkbnWc0g9Ehh0nBaJ4hYimEYB6k26K64uIXNj4gYBhdfBmzH7W0jGBfQ4coVojh_-guP8Ii1p09AmXEQyv6abpMHNTSe7749kSa76zTu8-Sg8OsXPjbzRYFzqXWI5sAmw_8o5kQ
```

#### Leveraging the refresh token functionality
When the ```access_token``` expires, a new one can be retrieved by the client on behalf of the collector using
the ```refresh_token``` from the Step 3. This can be achieved by making a HTTP POST request to *https://loyaltyone-sandbox-pin.auth0.com/oauth/token* URI with a JSON payload in the body as seen below.

```bash
curl --request POST \
  --url 'https://loyaltyone-sandbox-pin.auth0.com/oauth/token' \
  --header 'content-type: application/json' \
  --data '{ "grant_type": "refresh_token", "client_id": "YOUR_CLIENT_ID", "client_secret": "YOUR_CLIENT_SECRET", "refresh_token": "YOUR_REFRESH_TOKEN" }'
```

## References:

[AirMiles Developer Portal](http://developer.airmiles.ca/authorization.html)
[Auth0 - Overview](https://auth0.com/docs/getting-started/overview)
[Auth0 - Execute an Authorization Code Grant Flow](https://auth0.com/docs/api-auth/tutorials/authorization-code-grant)
