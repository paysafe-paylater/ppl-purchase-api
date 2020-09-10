# Javascript SDK
The Javascript client SDK provides a simplified way to integrate with the Paysafe Pay Later instore purchase API. The source code of this SDK can be found on [Github](https://github.com/Paysafecard-DEV/ppl-purchase-sdk-js). This documentation contains information on how to get started with the Paysafe Pay Later Javascript SDK.

# Installation
The Paysafe Pay Later Javascript SDK can be installed using [NPM](https://www.npmjs.com/) by including the following to the package.json file:

```json
"paysafe-paylater-sdk-client-javascript": "^1.0.0"
```

The package requires the following dependencies, which will automatically be installed when using npm:

Dependency | Version 
--- | ---
axios | ^0.19.2


## Browser bundle

The client SDK package contains a minified `paysafe-paylater-sdk.js`-file for inclusion in a browser as follows.

After loading the bundle, the SDK is available in the `SDK` constant.

```html
<script src="paysafe-paylater-sdk.js"></script>

<script defer>
    const Communicator = SDK.Communicator;
    const Connection = SDK.AxiosConnection;
    const PurchaseLifecycleApi = SDK.PurchaseLifecycleApi;

    const baseURL = "https://test-gateway.payolution.com";

    const conn = new Connection({
        baseURL: baseURL,
        log: SDK.logging.ConsoleLogger,
    });

    const comm = new Communicator(conn);

    const lifecycleAPI = new PurchaseLifecycleApi(comm);

</script>
```


# Authentication
The Paysafe Pay Later Javascript SDK can only be used when a valid access token has been acquired via a server application. Navigate to the [authentication](./authentication.md) for more information.

**Make sure to never use the secret key in a client side application or mobile app!**

# Usage
Before API calls can be made, the Paysafe Pay Later Javascript SDK needs to be initialised. A *Communicator* needs to be created, which needs to be passed to one of the API classes. Once the communicator has been created the following API classes can be initiated which can be used to invoke the different endpoints of the purchase API.

**PurchaseLifecycleApi**
 - [Purchase Info with Authorization token](#purchase-info)

**PurchaseAuthorizationApi**
- [Authorize with Authorization token](#authorize)

**LegalDocumentsApi**
- [Terms and Conditions with Authorization token](#terms-and-conditions)

## Base class for connector

The default connection uses axios to connect to the Paysafe Pay Later APIs. The baseclass covers the error handling.

Class | BaseClass | Description
--- | --- | ---
*AxiosConnection* | *BaseConnection* | The *BaseConnection* is responsible for the communication with the API endpoints.

To execute API calls, its adviced to instantiate the [Communicator](#communicator) class which provides functions for executing the calls and handling authorization tokens.

## Setup example

### Using browser bundle
```html
<script src="paysafe-paylater-sdk.js"></script>

<script defer>
    const Communicator = SDK.Communicator;
    const Connection = SDK.AxiosConnection;
    const PurchaseLifecycleApi = SDK.PurchaseLifecycleApi;
    const Logger = SDK.logging.ConsoleLogger;

    const config = {
      baseUrl: "https://test-gateway.payolution.com",
      log: Logger,
    }

    const conn = new Connection(config);
    const comm = new Communicator(conn);

    const purchaseLifecycleApi = new PurchaseLifecycleApi(communicator);
    const purchaseAuthorizationApi = new PurchaseAuthorizationApi(communicator);
    const legalDocumentsApi = new LegalDocumentsApi(communicator);

</script>
```

### Using package manager
```javascript
const SDK = require("src/paysafe-paylater/index");

const Communicator = SDK.Communicator;
const Connection = SDK.AxiosConnection;
const PurchaseLifecycleApi = SDK.PurchaseLifecycleApi;
const PurchaseAuthorizationApi = SDK.PurchaseAuthorizationApi;
const LegalDocumentsApi = SDK.LegalDocumentsApi;

const config = {
  baseUrl: "https://test-gateway.payolution.com"
}

const connection = new Connection(config);
const communicator = new Communicator(connection);

const purchaseLifecycleApi = new PurchaseLifecycleApi(communicator);
const purchaseAuthorizationApi = new PurchaseAuthorizationApi(communicator);
const legalDocumentsApi = new LegalDocumentsApi(communicator);
```

The *config* is a JavaScript Object to set the endpoint and logger with the following keys

```javascript
{
  baseUrl: "https://test-gateway.payolution.com/",
  timeout: 5000,
  headers:{},
  log: {},
}
```

### Configuration

Property name | Default value | Description
---------|----------|---------
 baseUrl | <no default, required> | The base URL of the Paysafe Pay Later server to connect to.
 timeout | 10000 | Time in milliseconds in which a connection should be established.
 headers | {} | Default headers used for the connection.
 log | <no default, optional> | Logger object containing specific logging methods.

## Exceptions
When the API returns an error, the SDK translates this into an exception that can then be handled by the application. The exceptions are mapped to the HTTP status codes the API responds with. The following table gives an overview of the exceptions that can occur:

Exception name | HTTP status code | Description
---------|----------|---------
 ApiException | - | Any other error that might occur.
 AuthorizationException | 401 or 403 | Authorization failed. Invalid/Expired token or secret key.
 PaysafeException | 500,502 or 503 | Something went wrong at the platform or further downstream.
 ReferenceException | 404, 409 or 410 | Non-existing or removed object is trying to be accessed.
 ValidationException | 400 | Validation of requests failed.

**Note:** Responses with a HTTP status code of 200 will never throw an exception, even if the transaction results in a decline. For more information about all possible error codes and their meaning, check the API specification or the status codes page.

Every error received from the API is wrapped in an exception that contains all the necessary information about the given error. The below snippet shows how the information can be extracted and used for any means necessary:

The API is based on Promises, which means then() and catch() or async/await can be used.

Example then() and catch()
```javascript
const PurchaseLifecycleApi = SDK.api.PurchaseLifecycleApi;
const InitializePurchaseRequest = SDK.models.InitializePurchaseRequest;
const { Amount, Currency } = SDK.models;

const purchaseLifecycleApi = new PurchaseLifecycleApi(communicator);

const request = new InitializePurchaseRequest().withPurchaseAmount(new Amount(50000, Currency.EUR));
purchaseLifecycleApi.initializePurchase(request, privateKey)
  .then(initializeResponse => {
    const purchaseOperationResponse = initializeResponse.response;
    const purchaseAuthorization = initializeResponse.authorization;
  }, error => {
    const status = error.getResponseStatus();
    const result = eerror.getOperationResult();
  });
```

Example async/await
```javascript
const PurchaseLifecycleApi = SDK.api.PurchaseLifecycleApi;
const InitializePurchaseRequest = SDK.models.InitializePurchaseRequest;
const { Amount, Currency } = SDK.models;

const purchaseLifecycleApi = new PurchaseLifecycleApi(communicator);

const request = new InitializePurchaseRequest().withPurchaseAmount(new Amount(50000, Currency.EUR));
try {
  const initializeResponse = await purchaseLifecycleApi.initializePurchase(request, privateKey);
  const purchaseOperationResponse = initializeResponse.response;
  const purchaseAuthorization = initializeResponse.authorization;
} catch (error) {
  const status = error.getResponseStatus();
  const result = error.getOperationResult();
}
```

#### ApiException
The *ApiException* class extends Error and is the parent of all Paysafe specific exceptions.
All exceptions wrap the Error from the (Axios) communicator.

## Logging
Logging of the requests, responses and exceptions can be enabled via the connection class.

The following example using the provided logging class would log all requests, responses and all errors to console:

```javascript
const Connection = SDK.connection.AxiosConnection;

const logger = SDK.logging.ConsoleLogger;

const connection = new Connection({
  baseURL: baseURL,
  log: logger,
});
```

You can provide your own logger by passing an object with the following methods:

```javascript
{
    request(req) {};
    requestError(e) {};
    response(resp) {};
    responseError(e) {};
}
```

## Examples
Below a few examples on how to invoke the different API endpoints with the Javascript SDK. The source code of the SDK also contains unit and integration tests which are a good reference as well.
The API is based on Promises, which means then() and catch() or async/await can be used. The examples are based on the async/await method. For then() and catch() examples you can refer to the unit and integration tests.

### Authorize

#### Authorize with authorization token
Authorize calls with an authorization token using URL redirect.

```javascript
const PurchaseAuthorizationApi = SDK.api.PurchaseAuthorizationApi;
const AuthorizePurchaseRequest = SDK.models.AuthorizePurchaseRequest;
const MethodType = SDK.models.MethodType;

const request = new AuthorizePurchaseRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withMethod(MethodType.URL)
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

const purchaseOperationResponse = await purchaseAuthorizationApi.authorizePayLaterWithAuthorization(request, authorizationToken);
```

### Purchase Info

#### Get purchase information with authorization token
```javascript
const purchaseResponse = await purchaseApi.getPurchaseWithAuthorization("CID-kdifr9ho54zavijvr9jv", authorizationToken);
```

### Terms and Conditions

#### Terms and Conditions with authorization
```javascript
// HTML response
const response = await legalDocumentsApi.getTermsAndConditionsWithAuthorization("CID-kdifr9ho54zavijvr9jv", authorizationToken);
```
