# Java SDK

The Java server SDK provides a simplified way to integrate with the Instore purchase API. The sourcecode of this SDK can be found on [Github](https://github.com). Below you will find all the information needed to get started with the Java SDK.

## SDK initialization

The easiest method for Initializing the SDK is by by using the *Factory* class provided by the SDK. With the following code snippet an instance of the *communicator* can be created. Once the communicator has been created the following API classes can be iniated which can be used to invoke the different endpoints of the purchase API.

**PurchaseLifecycleApi**
 - [Capture](#capture)
 - [Initialize](#initialize)
 - [Purchase Info](#purchase-info)
 - [Refund](#refund)

**PurchaseAuthorizationApi**
- [Authorize](#authorize)

**LegalDocumentsApi**
- [Terms and Conditions](#terms-and-conditions)

```java
Communicator communicator = Factory.createCommunicator(propertiesUrl.toURI());

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseAuthorizationApi purchaseAuthorizationApi = new PurchaseAuthorizationApi(communicator);
LegalDocumentsApi legalDocumentsApi = new LegalDocumentsApi(communicator);
```

The *propertiesUrl* should be refering to the property file with the connection configuration. The property file should contain the following fields:

```
paysafe.paylater.api.endpoint.host=test-gateway.payolution.com
paysafe.paylater.api.connectTimeout=5000
paysafe.paylater.api.socketTimeout=30000
paysafe.paylater.api.maxConnections=10
```

**Note**: all Java SDK examples assume the SDK has been initiliazed like mentioned above.

## Webhooks
Incoming webhooks need to be validated based on the secret key. The SDK provides a helper class that can be used to determine if the incoming webhook is valid and decode it into a readable string.

```java
WebhooksDecoder decoder = Webhooks.createHelper(secretKey);
String message = decoder.decrypt(base64StrToDecrypt);
```

## Exceptions

## Logging
Logging of the requests, responses and exceptions can be enabled via the communicator class, by default this is not enabled. In order to use the logging capabilities of the SDK an implemention of the CommunicatorLogger interface must be provided. This can either be a custom implementation or one of the two provided implementations:

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

## Examples

### Authorize

```java
PurchaseAuthorizationRequest request = new PurchaseAuthorizationRequest()
  .withPurchaseId(purchaseId)
  .withMethod(MethodType.SMS)
  .withPhone("+4300000000000")
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

PurchaseAuthorizationApi purchaseAuthorizationApi = new PurchaseAuthorizationApi(communicator);
PurchaseOperationResponse response = 
  purchaseAuthorizationApi.authorizePaylater(request, secretKey);
```

### Capture

Capture with purchase ID
```java
CaptureRequest captureRequest = new CaptureRequest()
  .withPurchaseId("CID-kdifr9ho54zavijvr9jv")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseOperationResponse response = purchaseApi.capture(request, secretKey);
```

Capture with order ID
```java
CaptureRequest captureRequest = new CaptureRequest()
  .withOrderId("75761090")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseOperationResponse response = purchaseApi.capture(request, secretKey);
```

### Initialize

```java
PurchaseInitializationRequest request = new PurchaseInitializationRequest()
  .withPurchaseAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse = 
  purchaseApi.intializePurchase(request, secretKey);
```

### Purchase Info
```java
PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseOperationResponse purchaseResponse = purchaseApi.getPurchase(purchaseId, secretKey);
```

### Refund

```java
PurchaseInitializationRequest request = new PurchaseInitializationRequest()
  .withPurchaseAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse =
  purchaseApi.intializePurchase(request, secretKey);
```

### Terms and Conditions
```java
LegalDocumentsApi legalDocumentsApi = new LegalDocumentsApi(communicator);
String response = legalDocumentsApi.termsandconditions(purchaseId, secretKey);
```