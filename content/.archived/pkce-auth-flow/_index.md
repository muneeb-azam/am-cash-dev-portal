+++
title = "Auth Code/Grant Flow (w/PKCE)"
menu = "main"
weight = 40
+++

## Overview
In order to access a **Member** API from a native mobile app, the client will need to implement the
**Authorization Code** OAuth 2.0 grant flow using PKCE. The Authorization Code Grant
(defined in [RFC 6749, section 4.1](https://tools.ietf.org/html/rfc6749#section-4.1)) is a flow
where the browser receives an **Authorization Code** from LoyaltyOne's Auth server and sends this
to the mobile application. The mobile app will then interact with the auth server and exchange the Authorization Code and a ```code_verifier``` for an ```access_token```, and optionally an ```id_token```
and/or a `refresh_token`. The mobile app can then use this `access_token` to call the
underlying API on behalf of the user.

![Authorized Code Grant Flow with PKCE (Sequence Diagram)](/images/authorization_code_grant_flow_with_pkce.png)

1. The user initiates the flow (by clicking login). The client application generates a `code_verifier` using a cryptographically random key.
2. The client application generates a `code_challenge` with the `code_verifier` using `SHA-256`.
The digest is base 64 encoded and sent as a parameter, along with the `code_challenge_method` *(`S256` to
indicate SHA-256)* and redirects the browser to the Authorization Server hosted page (AirMiles Single Sign-On),
so they can authenticate.
3. The Authorization Server then authenticates the user (via the browser). The first time, a user goes through this
flow, a consent page may be shown where the permissions are listed that would be given to the client.
4. After successful authentication, the authorization server redirects the user back to the mobile application
(specifically to the redirect_uri) with an Authorization Code in the query string (`code`).
5. The app then sends the Authorization Code to the Loyalty One server in a POST request and asks to exchange it for
 an access_token, id_token and a refresh_token (optional) via a prescribed endpoint. The `code_verifier` is also sent for native mobile applications.
6. Finally, after receiving back the access_token the mobile application can then use it to call the API on behalf of the user.

## Prerequisites
+ Auth0 client Id. Provided by LoyaltyOne Partner Integration Team.

## 1. Create a Code Verifier

{{% notice info %}}
Generate a `code_verifier`. A `code_verifier` is a cryptographically random key that must be generated for
every authorization request. For more information, click [here](https://auth0.com/docs/api-auth/grant/authorization-code-pkce#overview-of-the-flow).
{{% /notice %}}

##### Java
```java
SecureRandom sr = new SecureRandom();
byte[] code = new byte[32];
sr.nextBytes(code);
String verifier = Base64.encodeToString(code, Base64.URL_SAFE | Base64.NO_WRAP | Base64.NO_PADDING);
```

##### Swift 3
```js
var buffer = [UInt8](repeating: 0, count: 32)
_ = SecRandomCopyBytes(kSecRandomDefault, buffer.count, &buffer)
let verifier = Data(bytes: buffer).base64EncodedString()
.replacingOccurrences(of: "+", with: "-")
.replacingOccurrences(of: "/", with: "_")
.replacingOccurrences(of: "=", with: "")
.trimmingCharacters(in: .whitespaces)
```

##### Objective-C
```objc
NSMutableData *data = [NSMutableData dataWithLength:32];
int result attribute((unused)) = SecRandomCopyBytes(kSecRandomDefault, 32, data.mutableBytes);
NSString *verifier = [[[[data base64EncodedStringWithOptions:0]
stringByReplacingOccurrencesOfString:@"+" withString:@"-"]
stringByReplacingOccurrencesOfString:@"/" withString:@"_"]
stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"="]];
```

## 2. Create a Code Challenge
A digest called `code__challenge` is generated from the `code_verifier`.
The `code_challenge` is sent as part of the authorization request.

##### Java
```java
byte[] bytes = verifier.getBytes("US-ASCII");
MessageDigest md = MessageDigest.getInstance("SHA-256");
md.update(bytes, 0, bytes.length);
byte[] digest = md.digest();
//Use Apache "Commons Codec" dependency. Import the Base64 class
//import org.apache.commons.codec.binary.Base64;
String challenge = Base64.encodeBase64URLSafeString(digest);
```

##### Swift 3
```js
// You need to import CommonCrypto
guard let data = verifier.data(using: .utf8) else { return nil }
var buffer = [UInt8](repeating: 0,  count: Int(CC_SHA256_DIGEST_LENGTH))
data.withUnsafeBytes {
_ = CC_SHA256($0, CC_LONG(data.count), &buffer)
}
let hash = Data(bytes: buffer)
let challenge = hash.base64EncodedString()
.replacingOccurrences(of: "+", with: "-")
.replacingOccurrences(of: "/", with: "_")
.replacingOccurrences(of: "=", with: "")
.trimmingCharacters(in: .whitespaces)
```

##### Objective-C
```objc
// You need to import CommonCrypto
u_int8_t buffer[CC_SHA256_DIGEST_LENGTH * sizeof(u_int8_t)];
memset(buffer, 0x0, CC_SHA256_DIGEST_LENGTH);
NSData *data = [verifier dataUsingEncoding:NSUTF8StringEncoding];
CC_SHA256([data bytes], (CC_LONG)[data length], buffer);
NSData *hash = [NSData dataWithBytes:buffer length:CC_SHA256_DIGEST_LENGTH];
NSString *challenge = [[[[hash base64EncodedStringWithOptions:0]
stringByReplacingOccurrencesOfString:@"+" withString:@"-"]
stringByReplacingOccurrencesOfString:@"/" withString:@"_"]
stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"="]];
```

## 3. Get the User's Authorization
The external user agent makes a GET request on behalf of the user to this URI [https://sandbox.loyalty.com/v1/authorize](https://sandbox.loyalty.com/v1/authorize) with the query parameters mentioned below. The `code_challenge` and `code_challenge_method` are also required for the PKCE grant flow. The user will then get redirected to the AirMiles Single Sign-On page where they will be asked to enter their credentials. If the user agent already has a session to the auth server, the authentication step is skipped and the authorization code is passed to the `redirect_uri` immediately.

{{% notice note %}}
The native app **must not** use an embedded user agent to request make authorization requests. Requests should only be made through an external user agent to ensure user credentials remains hidden from the app. This also allows SSO capability since the user's auth session is only visible via the external browser.
{{% /notice %}}

```js
https://sandbox.loyalty.com/v1/authorize?
    scope=SCOPE&
    response_type=code&
    client_id=YOUR_CLIENT_ID&
    code_challenge=CODE_CHALLENGE&
    code_challenge_method=S256&
    redirect_uri=com.myclientapp://myclientapp.com/callback
```

**Where:**

|Parameter|Description|Value|
| ---------------:|----------------:|----------------:|
|```scope```**(optional)**|The scopes that you want to request authorization for. These must be separated by a space. Include value ```offline_access``` to get a refresh token.||
|```response_type```**(required)**|Denotes the kind of credential that the Authorization server will return.|```code```|
|```client_id```**(required)**|Your mobile application's identifier. This will be provided by LoyaltyOne.||
|```code_challenge```**(required)**|The generated challenge from the ```code_verifier```.||
|```code_challenge_method```**(required)**|The method used to generate the challenge.|```S256```|
|```redirect_uri```**(required)**|The URL to which the authorization server will redirect the browser after authorization has been granted to the user. The Authorization code (used in step 2) will be available in the ```code``` URL parameter. This URL must be specified as a valid callback URL and communicated to LoyaltyOne when the ```client_id``` is being generated for your mobile app.||

**For example:**

### Sample Request:
```html
<a href="https://sandbox.loyalty.com/v1/authorize?scope=appointments%20contacts&response_type=code&client_id=YOUR_CLIENT_ID&code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM&code_challenge_method=S256&redirect_uri=com.myclientapp://myclientapp.com/callback">
  Sign In
</a>
```

{{% notice tip %}}
The purpose of this call is to obtain consent from the user to invoke the API to do certain things (if any specified in `scope`) on behalf of the user. The LoyaltyOne Auth server then authenticates the user.
{{% /notice %}}

## 4. Exchange the Authorization Code for an Access Token
After extracting the Authorization Code `code` out of the redirect URL, the mobile app must exchange it for an `access_token` that can later be used to call the underlying API. Use the Authorization Code `code` from the previous step to make a `POST` request to this URI [https://sandbox.api.loyalty.com/v1/oauth/token](https://sandbox.api.loyalty.com/v1/oauth/token).

**Where:**

|Parameter|Description|Value|
| ---------------:|----------------:|----------------:|
|```grant_type```**(required)**|This must have the value as ```authorization_code```.||
|```client_id```**(required)**|Your mobile application's identifier. This will be provided by LoyaltyOne.|
|```code_verifier```**(required)**|The ```code_verifier``` generated in Step 1. The same ```code_verifier``` used to generate the ```code_challenge``` passed to ```/authorize``` in Step 3.||
|```code```**(required)**|The value of the ```code``` extracted out from the response returned in Step 3.||
|```redirect_uri```**(required)**|The URL to which the authorization server will redirect the browser after authorization has been granted to the user. The value of this URL must exactly match the ```redirect_uri``` passed in Step 3.||

##### cURL
```bash
curl --request POST \
  --url 'https://sandbox.loyalty.com/v1/oauth/token' \
  --header 'content-type: application/json' \
  --data '{"grant_type":"authorization_code","client_id": "YOUR_CLIENT_ID","code_verifier": "YOUR_GENERATED_CODE_VERIFIER","code": "YOUR_AUTHORIZATION_CODE","redirect_uri": "com.myclientapp://myclientapp.com/callback", }'
```

##### Java
```json
HttpResponse<String> response = Unirest.post("https://sandbox.loyalty.com/v1/oauth/token")
  .header("content-type", "application/json")
  .body("{\"grant_type\":\"authorization_code\",\"client_id\": \"YOUR_CLIENT_ID\",\"code_verifier\": \"YOUR_GENERATED_CODE_VERIFIER\",\"code\": \"YOUR_AUTHORIZATION_CODE\",\"redirect_uri\": \"com.myclientapp://myclientapp.com/callback\", }")
  .asString();
```

##### Swift 3
```js
import Foundation

let headers = ["content-type": "application/json"]

let postData = NSData(data: "{"grant_type":"authorization_code","client_id": "YOUR_CLIENT_ID","code_verifier": "YOUR_GENERATED_CODE_VERIFIER","code": "YOUR_AUTHORIZATION_CODE","redirect_uri": "com.myclientapp://myclientapp.com/callback", }".dataUsingEncoding(NSUTF8StringEncoding)!)

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

NSData *postData = [[NSData alloc] initWithData:[@"{"grant_type":"authorization_code","client_id": "YOUR_CLIENT_ID","code_verifier": "YOUR_GENERATED_CODE_VERIFIER","code": "YOUR_AUTHORIZATION_CODE","redirect_uri": "com.myclientapp://myclientapp.com/callback", }" dataUsingEncoding:NSUTF8StringEncoding]];

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

The response contains `access_token`, `refresh_token`, `id_token`, and `token_type` values; for example...

```json
{
  "access_token": "eyJz93a...k4laUWw",
  "refresh_token": "GEbRxBN...edjnXbL",
  "id_token": "eyJ0XAi...4faeEoQ",
  "token_type": "Bearer"
}
```

{{% notice note %}}
Note that `refresh_token` will only be present in the response if `scope=offline_access` was included in the request as part of Step 3.
{{% /notice %}}

## 5. Call the underlying API
Once the `access_token` has been obtained it can be used to make calls to the underlying API by including it in
the 'Authorization' header when making the corresponding request.
As an example, click **"Authorize"** in the [swagger spec](http://developer.airmiles.ca/member-banner/docs/swagger.html?url=https://sandbox.api.loyalty.com/v1/api-docs#/Member), then form a `Bearer id_token` string like below and enter this as the `api_key` in the `value` field.

```json
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
[Okta PKCE Write up](https://developer.okta.com/authentication-guide/auth-overview/#authorization-code-with-pkce-flow)
[Auth0 - Overview](https://auth0.com/docs/getting-started/overview)
[Auth0 - Execute an Authorization Code Grant Flow with PKCE](https://auth0.com/docs/api-auth/tutorials/authorization-code-grant-pkce)
[Auth0 - Calling APIs from Mobile Apps](https://auth0.com/docs/api-auth/grant/authorization-code-pkce)
[Auth0 - Mobile + API, Architecture Scenario](https://auth0.com/docs/architecture-scenarios/mobile-api)
