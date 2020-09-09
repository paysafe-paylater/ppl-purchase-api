# Authentication

The Paysafe Pay Later API allows certain endpoints to be invoked from a server to server or client side environment. Client side invocations are made possible by the use of an access token. Below an overview of when to use what kind of authentication and how to obtain an access token.

## Secret Key
The secret key is a value that only your application should know and must be kept safe (contact your account manager to obtain one or more secret keys). Therefore, all invocation towards the API with a secret key should be done from a server to server environment. Every key has one or multiple products configured to it. There will always be one default key that can be used to decrypt the incoming webhooks messages. The secret key is provided in the API calls via the 'paysafe-pl-secret-key' header. If one of our SDKs is used, it can be passed on as a parameter to any API requesting method and the SDK will take care of setting the correct header values.

## Access Token
Certain endpoints of the Paysafe Pay Later API can be invoked via a browser or mobile application. For these scenarios the authentication token must be used. The access token can be acquired from the initialize endpoint response. The value will be present in the header named 'access_token'. This first request needs the secret key which means that acquiring the access token should always be done from a server. Once the access token has been acquired, it can be used to execute the following endpoints from a client-side environment:

- /purchase/authorize/paylater
- /purchase/info/{purchaseId}
- /purchase/legaldocuments/termsandconditions/{purchaseId}