# Java SDK
The Java server SDK provides a simplified way to integrate with the Paysafe Pay Later instore purchase API. The source code of this SDK can be found on [Github](https://github.com/Paysafecard-DEV/ppl-purchase-sdk). This documentation contains information on how to get started with the Paysafe Pay Later Java SDK.

# Installation
The Paysafe Pay Later Java SDK can be installed using [Maven](http://maven.apache.org/) by including the following to the POM file:

<dependency>
  <groupId>com.paysafe.paylater</groupId>
  <artifactId>paysafe-sdk-java</artifactId>
  <version>x.y.z</version>
</dependency>

If Maven is not a possibility, download the latest version of the SDK from GitHub. Retrieve the `paysafe-sdk-java-x.y.z-bin.zip` file from the [releases](https://github.com/Paysafecard-DEV/ppl-purchase-sdk/releases) page, where `x.y.z` is the version number.
Add the JAR files inside the `lib` folder of your project, except for `paysafe-sdk-java-x.y.z-sources.jar`

The package requires the following dependencies, which will automatically be installed when using maven:

Dependency | Version 
--- | ---
com.google.code.gson | 2.8.5
org.apache.httpcomponents.httpclient | 4.5.6

## Requirements
The Paysafe Pay Later Java SDK requires Java version 8 or higher.

# Authentication
The Paysafe Pay Later Java SDK can be used with a secret key or authorization token for authentication.
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
Before API calls can be made, the Paysafe Pay Later Java SDK needs to be initialised. A *Communicator* needs to be created, which needs to be passed to one of the API classes. Once the communicator has been created the following API classes can be initiated which can be used to invoke the different endpoints of the purchase API.

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

## Interfaces
Each part of the Paysafe Pay Later Java SDK communication process implements an interface:

Class | Interface | Description
--- | --- | ---
*PaysafeConnection* | *Connection* | A *Connection* is responsible for the communication with the API endpoints.
*PaysafeCommunicator* | *Communicator* | A *Communicator* class is responsible for converting API responses into specified *Response* objects.

## Setup example

```java
import com.paysafe.paylater.Factory;
import com.paysafe.paylater.api.LegalDocumentsApi;
import com.paysafe.paylater.api.PurchaseAuthorizationApi;
import com.paysafe.paylater.api.PurchaseLifecycleApi;
import com.paysafe.paylater.communication.Communicator;

Communicator communicator = Factory.createCommunicator(propertiesUrl.toURI());

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseAuthorizationApi purchaseAuthorizationApi = new PurchaseAuthorizationApi(communicator);
LegalDocumentsApi legalDocumentsApi = new LegalDocumentsApi(communicator);
```

The *propertiesUrl* should be refering to the property file with the connection configuration. The property file should at least contain the host URL of the Paysafe Pay Later server. Below an example of a few property values:

```
paysafe.paylater.api.endpoint.host=test-gateway.payolution.com
paysafe.paylater.api.connectTimeout=5000
paysafe.paylater.api.socketTimeout=30000
paysafe.paylater.api.maxConnections=10
paysafe.paylater.api.endpoint.port=-1
```

### Configuration

Property name | Default value | Description
---------|----------|---------
 paysafe.paylater.api.endpoint.host | <no default, required> | The host of the Paysafe Pay Later server to connect to.
 paysafe.paylater.api.connectTimeout | 10000 | Time in milliseconds in which a connection should be established.
 paysafe.paylater.api.readTimeout | 10000 | Time in milliseconds in which a connection should start returning data.
 paysafe.paylater.api.maxConnections | 10 | The SDK uses a connection pool by default. This property limits the amount of connections in the pool.
 paysafe.paylater.api.endpoint.port | -1 | The port of the Paysafe Pay Later server to connect on. -1 indicates any port can be used.
 paysafe.paylater.api.endpoint.scheme | https | The scheme to use to setting up communications. All Paysafe servers communicate via SSL-encrypted connections
 paysafe.paylater.api.https.protocols | TLSv1.2 | The TLS protocol(s) to use for SSL connections. Paysafe servers require at least TLS1.2.

### Initializing via code
If other ways of configuration are used in the application, a connection can be created manually. This can be done by providing the configuration values to the constructor of the PaysafeConnection class and pass it on to the Factory to create a Communicator.

```java
import com.paysafe.paylater.Factory;
import com.paysafe.paylater.api.LegalDocumentsApi;
import com.paysafe.paylater.api.PurchaseAuthorizationApi;
import com.paysafe.paylater.api.PurchaseLifecycleApi;
import com.paysafe.paylater.communication.Communicator;
import com.paysafe.paylater.communication.standard.PaysafeConnection;

PaysafeConnection connection = new PaysafeConnection(connectionTimeout, readTimeout);
Communicator communicator = Factory.createCommunicator(apiEndpoint, connection)

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseAuthorizationApi purchaseAuthorizationApi = new PurchaseAuthorizationApi(communicator);
LegalDocumentsApi legalDocumentsApi = new LegalDocumentsApi(communicator);
```

## Responses
All endpoints return the same response in the form of a PurchaseOperationResponse object. This object contains all data about the transaction like the status, authorized/captured amount and other metadata. The initialize endpoint however, returns this response differently. The initialize endpoint also returns an access token, and thus the data for both the PurchaseOperationResponse and access token have been wrapped in a seperate object. Below an example of how to handle this case.

```java
ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse = 
  purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
String accessToken = purchaseResponse.getAuthorization();

PurchaseOperationResponse response = purchaseResponse.getResponse();
```

# Webhooks
Incoming  webhooks messages need to be validated based on the default secret key. The SDK provides a helper class that can be used to determine if the incoming webhook is valid and decodes it into a PurchaseOperationResponse object. The incoming webhook can be marshalled into a WebhookMessage object which can then be used in the decrypt method. Below an example of decrypting an incoming webhook.

```java
import com.paysafe.paylater.communication.Marshaller;
import com.paysafe.paylater.communication.standard.JsonMarshaller;
import com.paysafe.paylater.model.OperationResult;
import com.paysafe.paylater.model.OperationStatus;
import com.paysafe.paylater.model.PurchaseInformation;
import com.paysafe.paylater.model.PurchaseOperationResponse;
import com.paysafe.paylater.webhook.WebhookDecrypter;
import com.paysafe.paylater.webhook.WebhookMessage;

Marshaller marshaller = JsonMarshaller.INSTANCE;
WebhookMessage webhookMessage = marshaller.unmarshal(request.getData(), WebhookMessage.class);

WebhookDecrypter decoder = new WebhookDecrypter(marshaller);
PurchaseOperationResponse response = decoder.decrypt(webhookMessage, secretKey);

OperationResult result = response.getResult();
Assert.assertEquals(OperationStatus.OK, result.getStatus());
Assert.assertEquals("0.0.0", result.getStatusCode());

PurchaseInformation purchase = response.getPurchase();
Assert.assertNotNull(purchase.getPurchaseId());
```

## Exceptions
When an error is returned by the API, the SDK will translate this into an exception that can then be handled by the application. The exceptions are mapped to the HTTP status codes the API responds with. The following table gives an overview of the exceptions that can occur:

Exception name | HTTP status code | Description
---------|----------|---------
 ValidationException | 400 | Validation of requests failed.
 AuthorizationException | 401 or 403 | Authorization failed. Invalid/Expired token or secret key.
 ReferenceException | 404, 409 or 410 | Non-existing or removed object is trying to be accessed.
 NotFoundException | 404 | Occurs with non JSON responses. As example, an HTML error response.
 PaysafeException | 500,502 or 503 | Something went wrong at the platform or further downstream.
 ApiException | - | Any other error that might occur.
 CommunicationException | - | Indicates an exception regarding the communication with the Paysafe Pay Later platform

**Note:** Responses with a HTTP status code of 200 will never throw an exception, even if the transaction results in a decline. For more information about all possible error codes and their meaning, check the API specification or the status codes page.

Every error received from the API is wrapped in an exception that contains all the necessary information about the given error. The below snippet shows how the information can be extracted and used for any means necessary:

```java
import com.paysafe.paylater.api.PurchaseLifecycleApi;
import com.paysafe.paylater.exception.ValidationException;
import com.paysafe.paylater.model.*;

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
InitializePurchaseRequest request = new InitializePurchaseRequest()
  .withPurchaseAmount(new Amount(50000L, Currency.EUR));

try {
  purchaseApi.initializePurchase(request, privateKey);
} catch (ValidationException e) {
  Assert.assertEquals(e.getResponseStatusCode(), 400);
  OperationResult result = e.getOperationResult();
  Assert.assertEquals(OperationStatus.ERROR, result.getStatus());
  Assert.assertEquals("3.1.0", result.getStatusCode());
}
```

#### ApiException
The *ApiException* class extends RuntimeException and is the parent of all Paysafe specific exceptions. The class has the following properties:

Property name | Description
---------|---------
message | Inherited from Exception
errorCode | Inherited from Exception
responseBody | Optional: response body
responseHeaders | List of *ResponseHeader* objects containing all available response headers
operationResult | Optional: instance of *OperationResult*

## Logging
Logging of the requests, responses and exceptions can be enabled via the communicator class, by default this is not enabled. To use the logging capabilities of the SDK, an implementation of the CommunicatorLogger interface must be provided. This can either be a custom implementation or one of the two provided implementations:

- SysOutCommunicatorLogger (logs to System.out)
- JdkCommunicatorLogger (logs to a java.util.logging.Logger)

To enable the logging on the communicator, the implementation can be provided to the *enableLogging* method. Disabling the logging is as easy as invoking the *disableLogging* method on the communicator class.

```java
Communicator communicator = Factory.createCommunicator(propertiesUrl.toURI());
communicator.enableLogging(SysOutCommunicatorLogger.INSTANCE);
PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);

// Disable the logging
communicator.disableLogging();
```

**Note:** Logging can also be enabled via the created API classes, but do keep in mind that it will be enabled on the communicator class. If the communicator is shared among other API classes, they will all have their logging enabled.

## Examples
Below a few examples on how to invoke the different API endpoints with the Java SDK. The source code of the SDK also contains unit and integration tests which are a good reference as well.

### Authorize

#### Authorize with SMS
```java
import com.paysafe.paylater.model.AuthorizePurchaseRequest;
import com.paysafe.paylater.model.MethodType;
import com.paysafe.paylater.model.PurchaseOperationResponse;

AuthorizePurchaseRequest request = new AuthorizePurchaseRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withMethod(MethodType.SMS)
  .withPhone("+4300000000000")
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

PurchaseOperationResponse response = 
  purchaseAuthorizationApi.authorizePaylater(request, secretKey);
```

#### Authorize with redirect URL
```java
import com.paysafe.paylater.model.AuthorizePurchaseRequest;
import com.paysafe.paylater.model.MethodType;
import com.paysafe.paylater.model.PurchaseOperationResponse;

AuthorizePurchaseRequest request = new AuthorizePurchaseRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withMethod(MethodType.URL)
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

PurchaseOperationResponse response = 
  purchaseAuthorizationApi.authorizePaylater(request, secretKey);

String authUrl = response.getPurchase().getMetaData().get("INSTORE_SELFSERVICE_AUTH_URL");
```

#### Authorize with authorization token
Authorize calls with an authorization token use a different method:

```java
import com.paysafe.paylater.model.AuthorizePurchaseRequest;
import com.paysafe.paylater.model.MethodType;
import com.paysafe.paylater.model.PurchaseOperationResponse;

AuthorizePurchaseRequest request = new AuthorizePurchaseRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withMethod(MethodType.URL)
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

PurchaseOperationResponse response = 
  purchaseAuthorizationApi.authorizePayLaterWithAuthorization(request, authorizationToken);

String authUrl = response.getPurchase().getMetaData().get("INSTORE_SELFSERVICE_AUTH_URL");
```

The value of variable `authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.

### Capture

#### Capture with purchase ID
```java
import com.paysafe.paylater.model.*;

CapturePurchaseRequest request = new CapturePurchaseRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseOperationResponse response = purchaseApi.capturePurchase(request, secretKey);
```

#### Capture with order ID
```java
import com.paysafe.paylater.model.*;

CapturePurchaseRequest request = new CapturePurchaseRequest()
  .withOrderId("75761090")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseOperationResponse response = purchaseApi.capturePurchase(request, secretKey);
```

### Initialize

#### Initialize with minimal required fields
```java
import com.paysafe.paylater.model.*;

InitializePurchaseRequest request = new InitializePurchaseRequest()
  .withPurchaseAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse = 
  purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
String accessToken = purchaseResponse.getAuthorization();

PurchaseOperationResponse response = purchaseResponse.getResponse();
String purchaseId = response.getPurchaseId();
```

#### Initialize with consumer data
```java
import com.paysafe.paylater.model.*;

import java.time.LocalDate;

Amount purchaseAmount = new Amount(50000L, Currency.EUR);
InitializePurchaseRequest request = new InitializePurchaseRequest(purchaseAmount)
  .withConsumer(new Consumer()
    .withEmail("instore-test@paysafe.com")
    .withPhone("123456789")
    .withPerson(new Person()
      .withFirstName("Ernst")
      .withLastName("Muller")
      .withBirthdate(LocalDate.parse("1989-08-22")))
    .withBillingAddress(new Address()
      .withCountryCode(Country.AT)
      .withZipCode("5500")
      .withCity("Bischofshofen")
      .withStreet("Hauptstrasse")
      .withHouseNumber("2")));

ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse = 
  purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
String accessToken = purchaseResponse.getAuthorization();

PurchaseOperationResponse response = purchaseResponse.getResponse();
String purchaseId = response.getPurchaseId();
```

### Purchase Info

#### Get purchase information
```java
import com.paysafe.paylater.model.PurchaseOperationResponse;

PurchaseOperationResponse purchaseResponse = purchaseApi.getPurchase("CID-kdifr9ho54zavijvr9jv", secretKey);
```

#### Get purchase information with authorization token
```java
import com.paysafe.paylater.model.PurchaseOperationResponse;

PurchaseOperationResponse purchaseResponse = purchaseApi.getPurchaseWithAuthorization("CID-kdifr9ho54zavijvr9jv", authorizationToken);
```

The value of variable `authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.

### Refund
```java
import com.paysafe.paylater.model.Amount;
import com.paysafe.paylater.model.Currency;
import com.paysafe.paylater.model.PurchaseOperationResponse;
import com.paysafe.paylater.model.RefundPurchaseRequest;

RefundPurchaseRequest request = new RefundPurchaseRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withRefundAmount(new Amount()
    .withAmount(5000L)
    .withCurrency(Currency.EUR));

PurchaseOperationResponse purchaseResponse = purchaseApi.refundPurchase(request, secretKey);
```
### Terms and Conditions

#### Terms and Conditions
```java
// HTML response
String response = legalDocumentsApi.getTermsAndConditions("CID-kdifr9ho54zavijvr9jv", secretKey);
```

#### Terms and Conditions with authorization
```java
// HTML response
String response = legalDocumentsApi.getTermsAndConditionsWithAuthorization("CID-kdifr9ho54zavijvr9jv", authorizationToken);
```

The value of variable `authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.
