# Identity
[[عربي]](readme.ar.md)

A library for integrating WebPlatform applications with public authentication services, such as Google authentication
service, which allows users to log in using their Gmail accounts. It is designed to support multiple authentication
services, but currently only supports Google's service.

## Usage

These steps assume that you already have a WebPlatform project and you want to add authentication to it.

* Add the library to your project:

```
import "Apm";
Apm.importFile("Alusus/Identity");
```

* Create an application definition in the Google service by visiting the website https://console.cloud.google.com/apis/,
  then add an OAuth 2.0 identifier for this application through the Credentials option in the menu. Copy the values of
  clientId and clientSecret from there.

* Using the OAuth consent screen option, select the required scopes for the application. You will need at least openid,
  email, and profile.

* Create an RSA key to be used for authenticating authentication identities and store it in a file within your project.
  The key should be based on RSA 256 encoding.

* Setup the REST interface of the library by calling `setupRestApi` and providing it with the name of the RSA key file,
  along with a closure that is invoked when logging in to create a user account and return its unique identifier.

```
Identity.setupRestApi("rsakey", closure (res: Identity.AuthenticationResponse): String {
    // Use user info from res to lookup the user record in the project's DB, or to create a new
    // record if this is a new user. This closure should return the user's unique id.
    return userId;
}
```

* Add Google provider to the library's list of providers:

```
def clientId: "<the value of clientId from Google's dashboard>";
def clientSecret: "<the value of clientSecret from Google's dashboard>";
Identity.addProvider(Identity.GoogleProvider(String(clientId), String(clientSecret)));
```

* Add a Google login button to your website:

```
def redirectUrl: "the URL within your site to send the user to after authorizing to complete the login";
Identity.GoogleAuthenticationButton(String(clientId), String(redirectUrl), 0).{
    // Customize styles.
}
```

The third argument to this component specifies whether the required button style is dark or light. The value 0 indicates
the light style, while the value 1 indicates the dark style.

* On the login completion page (the page to which the user is redirected after authentication), add the following to
complete the login process:

```
if await[Bool](Identity.finalizeAuthentication(redirectUrl)) {
    // Authentication is complete.
    // Remove auth params from the URL bar by redirecting the user to another page, or to the same page.
    Window.instance.pushLocation(redirectUrl);
    // Login is complete now, and we can proceed with the app.
} else {
    // Authentication failed, or there were no auth params in the URL.
}
```

* To check the login status and automatically renew the access token (using the refresh token):

```
def loggedIn: Bool = await[Bool](Identity.checkAuthenticationStatus());
if not loggedIn {
    // Show login screen.
}
```

### Using the Access Token for Authenticated Calls

When calling an authenticated endpoint on the server, you need to include the access token in your request. The way to
include the token in the request is up to you (e.g., you can add it to the headers). You can obtain the access token
from the client by calling the function `getAccessToken`:

```
def accessToken: String = getAccessToken();
```

After receiving the access token from the server, you need to validate it using the function `validateAccessToken`.

### Logging Out

You can log the user out by removing the authentication data from the browser:

```
clearAuthenticationData();
```

## Functions and Classes

### authenticate

```
function authenticate(providerName: String, code: String, redirectUri: String): Possible[AuthenticationResponse];
```

A function to complete the authentication process from the server-side. This function is internally called by the
library's endpoints.

* `providerName`: The name of the provider used for authentication (e.g., `google`).
* `code`: The authentication code provided by the service provider to the page the user is redirected to after
  granting consent.
* `redirectUri`: The URL of the page the user is redirected to after granting consent.

### reauthenticate

```
function reauthenticate(providerName: String, refreshToken: String): Possible[AuthenticationResponse];
```

A function to create a new external access token using the refresh token. This function is used by the library's
endpoints.

* `providerName`: The name of the provider used for authentication (e.g., `google`).
* `refreshToken`: The refresh token provided by the service provider during the initial authentication process.

### getUserInfo

```
function getUserInfo(providerName: String, extAccessToken: String): Possible[UserInfoResponse];
```

Retrieves the user's profile data from the service provider.

* `providerName`: The name of the provider used for authentication (e.g., `google`).
* `extAccessToken`: The user's external access token associated with the service provider.

### validateAccessToken

```
function validateAccessToken(
    token: String, providerName: ref[String], extAccessToken: ref[String], userId: ref[String]
): Bool
```

This function returns a boolean value, `true` (1), if the access token is valid, and `false` (0) otherwise.

* `token`: The access token to be validated.
* `providerName`: If the token is valid, this variable is set to the name of the provider associated with the token.
  Currently, the library only supports Google as the provider, so the value of this variable will be `google`.
* `extAccessToken`: If the token is valid, this variable is set to the external access token (associated with a service
  like Google). The external access token can be used to communicate with the external service (e.g., the Google API).
* `userId`: The unique identifier of the user, which is returned by the closure passed to the `setupRestApi` function.

### TokenInfo

```
class TokenInfo {
    def accessToken: String;
    def accessTokenExpiry: Int[64] = 0;
    def refreshToken: String;

    handler this~init(accessToken: String, accessTokenExpiry: Int[64], refreshToken: String);
}
```

### UserInfo

```
class UserInfo {
    def id: String;
    def name: String;
    def email: String;

    handler this~init(id: String, name: String, email: String);
}
```

### AuthenticationResponse

```
class AuthenticationResponse {
    def providerName: String;
    @injection def tokenInfo: TokenInfo;
    @injection def userInfo: UserInfo;
}
```

### UserInfoResponse

```
class UserInfoResponse {
    def providerName: String;
    @injection def userInfo: UserInfo;
}

```

