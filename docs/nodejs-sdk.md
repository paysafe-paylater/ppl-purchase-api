# Node.js SDK
The Node.js server SDK provides a simplified way to integrate with the Paysafe Pay Later instore purchase API. The source code of this SDK can be found on [Github](https://github.com/Paysafecard-DEV/ppl-purchase-sdk-nodejs). This documentation contains information on how to get started with the Paysafe Pay Later Javascript SDK.

# Installation
The Paysafe Pay Later Node.js SDK can be installed using [NPM](https://www.npmjs.com/) by including the following to the package.json file:

```json
"paysafe-paylater-sdk-nodejs": "^1.0.0"
```

The package requires the following dependencies, which will automatically be installed when using npm:

Dependency | Version 
--- | ---
axios | ^0.19.2

## Requirements
The Paysafe Pay Later Node.js SDK requires Node.js version 10 or higher.

# Authentication
The Paysafe Pay Later Node.js SDK can be used with a secret key or authorization token for authentication.
Secret key authentication is recommended for server to server communication and authorization token authentication is recommended for client side applications or apps.

## Secret key
Multiple secret keys can be requested and each key can have one or multiple products configured to it. One of the secret keys will always be the default key. The default key can be used to decrypt incoming [webhook](#webhooks) messages.

**Make sure to never use the secret key in a client side application or mobile app!**

## Authorization token
An authorization token can be obtained by performing a [initialize](#initialize) call. 
The authentication token can be used for the following endpoints:

- /purchase/authorize/paylater
- /purchase/info/{purchaseId}
- /purchase/legaldocuments/termsandconditions/{purchaseId}

# Usage
Before API calls can be made, the Paysafe Pay Later Node.js SDK needs to be initialised. A *Communicator* needs to be created, which needs to be passed to one of the API classes. Once the communicator has been created the following API classes can be initiated which can be used to invoke the different endpoints of the purchase API.

**PurchaseLifecycleApi**
 - [Capture](#capture)
 - [Initialize](#initialize)
 - [Purchase Info](#purchase-info)
 - [Purchase Info with Authorization token](#purchase-info)
 - [Refund](#refund)

**PurchaseAuthorizationApi**
- [Authorize](#authorize)
- [Authorize with Authorization token](#authorize)

**LegalDocumentsApi**
- [Terms and Conditions](#terms-and-conditions)
- [Terms and Conditions with Authorization token](#terms-and-conditions)

## Base class for connector

The default connection uses axios to connect to the Paysafe Pay Later APIs. The baseclass covers the error handling.

Class | BaseClass | Description
--- | --- | ---
*AxiosConnection* | *BaseConnection* | The *BaseConnection* is responsible for the communication with the API endpoints.

To execute API calls, its adviced to instantiate the [Communicator](#communicator) class which provides functions for executing the calls and handling authorization tokens.

## Setup example

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

## Responses
All endpoints return the same response in the form of a PurchaseOperationResponse object. This object contains all data about the transaction like the status, authorized/captured amount and other metadata. The initialize endpoint however, returns this response differently. The initialize endpoint also returns an access token, and thus the data for both the PurchaseOperationResponse and access token have been wrapped in a seperate object. Below an example of how to handle this case.

```javascript
const purchaseResponse = await purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
const accessToken = purchaseResponse.authorization;

const response = purchaseResponse.response;
const purchaseId = response.getPurchaseId();
```

# Webhooks
Incoming  webhooks messages need to be validated based on the default secret key. The SDK provides a helper class that can be used to determine if the incoming webhook is valid and decodes it into a PurchaseOperationResponse object. Below an example of decrypting an incoming webhook.

```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const WebhookDecrypter = SDK.WebhookDecrypter;

const purchaseOperationResponse = WebhookDecrypter.decrypt(webhookMessage, secretKey);

const operationResult = response.getResult();
const purchaseInformation = response.getPurchase();
```

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
const SDK = require("paysafe-paylater-sdk-nodejs");

const PurchaseLifecycleApi = SDK.api.PurchaseLifecycleApi;
const InitializePurchaseRequest = SDK.models.InitializePurchaseRequest;
const { Amount, Currency } = SDK.models;

const purchaseLifecycleApi = new PurchaseLifecycleApi(communicator);

const request = new InitializePurchaseRequest(new Amount(50000, Currency.EUR));
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
const SDK = require("paysafe-paylater-sdk-nodejs");

const PurchaseLifecycleApi = SDK.api.PurchaseLifecycleApi;
const InitializePurchaseRequest = SDK.models.InitializePurchaseRequest;
const { Amount, Currency } = SDK.models;

const purchaseLifecycleApi = new PurchaseLifecycleApi(communicator);

const request = new InitializePurchaseRequest(new Amount(50000, Currency.EUR));
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
const SDK = require("paysafe-paylater-sdk-nodejs");

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
Below a few examples on how to invoke the different API endpoints with the Node.js SDK. The source code of the SDK also contains unit and integration tests which are a good reference as well.
The API is based on Promises, which means then() and catch() or async/await can be used. The examples are based on the async/await method. For then() and catch() examples you can refer to the unit and integration tests.

### Authorize

#### Authorize with SMS
```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const PurchaseAuthorizationApi = SDK.api.PurchaseAuthorizationApi;
const AuthorizePurchaseRequest = SDK.models.AuthorizePurchaseRequest;
const MethodType = SDK.models.MethodType;

const request = new AuthorizePurchaseRequest("CID-kdifr9ho54zavijvr9jv")
  .withMethod(MethodType.SMS)
  .withPhone("+4300000000000")
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

const purchaseOperationResponse = await purchaseAuthorizationApi.authorizePayLater(request, secretKey);
```

#### Authorize with redirect URL
```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const PurchaseAuthorizationApi = SDK.api.PurchaseAuthorizationApi;
const AuthorizePurchaseRequest = SDK.models.AuthorizePurchaseRequest;
const MethodType = SDK.models.MethodType;

const request = new AuthorizePurchaseRequest("CID-kdifr9ho54zavijvr9jv")
  .withMethod(MethodType.URL)
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

const purchaseOperationResponse = await purchaseAuthorizationApi.authorizePayLater(request, secretKey);
const authUrl = response.getPurchase().getMetaData()["INSTORE_SELFSERVICE_AUTH_URL"];
```

#### Authorize with authorization token
Authorize calls with an authorization token use a different method:

```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const PurchaseAuthorizationApi = SDK.api.PurchaseAuthorizationApi;
const AuthorizePurchaseRequest = SDK.models.AuthorizePurchaseRequest;
const MethodType = SDK.models.MethodType;

const request = new AuthorizePurchaseRequest("CID-kdifr9ho54zavijvr9jv")
  .withMethod(MethodType.URL)
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

const purchaseOperationResponse = await purchaseAuthorizationApi.authorizePayLaterWithAuthorization(request, authorizationToken);
const authUrl = response.getPurchase().getMetaData()["INSTORE_SELFSERVICE_AUTH_URL"];
```

The value of variable `authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.

### Capture

#### Capture with purchase ID
```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const CapturePurchaseRequest = SDK.models.CapturePurchaseRequest;
const { Amount, Currency } = SDK.models;

const request = new CapturePurchaseRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withFulfillmentAmount(new Amount(50000, Currency.EUR));

const purchaseOperationResponse = await purchaseApi.capturePurchase(request, secretKey);
```

#### Capture with order ID
```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const CapturePurchaseRequest = SDK.models.CapturePurchaseRequest;
const { Amount, Currency } = SDK.models;

const request = new CapturePurchaseRequest()
  .withOrderId("75761090")
  .withFulfillmentAmount(new Amount(50000, Currency.EUR));

const purchaseOperationResponse = await purchaseApi.capturePurchase(request, secretKey);
```

### Initialize

#### Initialize with minimal required fields
```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const InitializePurchaseRequest = SDK.models.InitializePurchaseRequest;
const { Amount, Currency } = SDK.models;

const request = new InitializePurchaseRequest(new Amount(50000, Currency.EUR));

const purchaseResponse = await purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
const accessToken = purchaseResponse.authorization;

const response = purchaseResponse.response;
const purchaseId = response.getPurchaseId();
```

#### Initialize with consumer data
```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const AuthorizePurchaseRequest = SDK.models.InitializePurchaseRequest;
const { Address, Amount, Consumer, Country, Currency, Person } = SDK.models;

const request = new InitializePurchaseRequest(new Amount(50000, Currency.EUR))
  .withConsumer(new Consumer()
    .withEmail("instore-test@paysafe.com")
    .withPhone("123456789")
    .withPerson(new Person()
      .withFirstName("Ernst")
      .withLastName("Muller")
      .withBirthdate("1990-07-21"))
    .withBillingAddress(new Address()
      .withCountryCode(Country.AT)
      .withZipCode("5500")
      .withCity("Bischofshofen")
      .withStreet("Hauptstrasse")
      .withHouseNumber("2")));

const purchaseResponse = await purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
const accessToken = purchaseResponse.authorization;

const response = purchaseResponse.response;
const purchaseId = response.getPurchaseId();
```

### Purchase Info

#### Get purchase information
```javascript
const purchaseResponse = await purchaseApi.getPurchase("CID-kdifr9ho54zavijvr9jv", secretKey);
```

#### Get purchase information with authorization token
```javascript
const purchaseResponse = await purchaseApi.getPurchaseWithAuthorization("CID-kdifr9ho54zavijvr9jv", authorizationToken);
```

The value of variable `authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.

### Refund
```javascript
const SDK = require("paysafe-paylater-sdk-nodejs");

const AuthorizePurchaseRequest = SDK.models.RefundPurchaseRequest;
const Amount = SDK.models.Amount;
const Currency = SDK.models.Currency;

RefundPurchaseRequest request = new RefundPurchaseRequest("CID-kdifr9ho54zavijvr9jv", new Amount(5000, Currency.EUR));

const purchaseResponse = await purchaseApi.refundPurchase(request, secretKey);
```
### Terms and Conditions

#### Terms and Conditions
```javascript
// HTML response
const response = await legalDocumentsApi.getTermsAndConditions("CID-kdifr9ho54zavijvr9jv", secretKey);
```

#### Terms and Conditions with authorization
```javascript
// HTML response
const response = await legalDocumentsApi.getTermsAndConditionsWithAuthorization("CID-kdifr9ho54zavijvr9jv", authorizationToken);
```

The value of variable `authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.
