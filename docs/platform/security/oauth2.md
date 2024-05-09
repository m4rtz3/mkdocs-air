OAuth2 is an authorization framework that allows applications to obtain limited access to user accounts on an HTTP service, such as Facebook, Google, or GitHub, without exposing the user's credentials. It provides a secure and standardized way for users to grant access to their resources to third-party applications.

The principal process to obtain a valid credential using OAuth2 involves the following steps:

1. **Registration**: The application developer needs to register their application with the OAuth2 provider (e.g., Google, Facebook) to obtain client credentials, including a client ID and client secret. These credentials are used to identify and authenticate the application.

2. **User Authorization**: When a user wants to grant access to their resources, the application redirects them to the OAuth2 provider's authorization endpoint. This typically involves the user being presented with a login screen and being asked to grant permission to the application.

3. **Authorization Grant**: Once the user grants permission, the OAuth2 provider issues an authorization grant to the application. This grant can take various forms, such as an authorization code or an access token.

4. **Token Exchange**: The application then exchanges the authorization grant for an access token by sending a request to the OAuth2 provider's token endpoint. The access token is a credential that the application can use to access the user's resources on behalf of the user.

5. **Accessing Resources**: With the access token, the application can make requests to the OAuth2 provider's API endpoints to access the user's resources. The access token is typically included in the request headers or as a query parameter.

6. **Refreshing Tokens**: Access tokens have a limited lifespan. To continue accessing the user's resources, the application can use a refresh token (if provided) to obtain a new access token without requiring the user to reauthorize the application.

It's important to note that the exact process and terminology may vary slightly depending on the OAuth2 provider and the specific implementation. However, the general flow remains consistent across most OAuth2 implementations.