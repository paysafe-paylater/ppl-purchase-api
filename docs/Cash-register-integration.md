# Integrations: Cash register integration

Integration with a cash register takes little effort and provides secure confirmation of payments. The Cash register integration has two ways of starting the flow.

**Note**: all SDK examples assume the SDK has been initiliazed. For more details on how to initialize the SDK, check the paragraph 'SDK initialization' in the corosponding SDK documentation.

## Start at cash register

### 1. Initialize

This flow starts with the cash register specifying the amount of the order and invoking the initialize endpoint with the amount and currency. Optionaly, customer data can be provided to ease the application process for the customer. The success response of the initialize endpoint returns the **purchaseId** needed for the authorize endpoint.

Optional: The initialize response also contains an **access_token** header which can be used for requests made via a client system. The SDKs take care of retrieving the access token, in *Bearer* format, from the header and providing it together with the initialize reponse. The endpoints that support this access tokken are limited to:

- /purchase/authorize/paylater
- /purchase/info/{purchaseId}
- /purchase/legaldocuments/termsandconditions/{purchaseId}


**Rest endpoint**

```json http
{
  "method": "post",
  "url": "https://test-gateway.payolution.com/purchase/initialize",
  "headers": {
    "paysafe-pl-secret-key": "secret-key",
    "Content-Type": "application/json"
  },
  "body": {
    "purchaseAmount": {
      "amount": 50000,
      "currency": "EUR"
    }
  }
}
```

**Java SDK**

```java
PurchaseInitializationRequest request = new PurchaseInitializationRequest()
  .withPurchaseAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse = 
  purchaseApi.intializePurchase(request, secretKey);
String accessToken = purchaseResponse.getAuthorization(); // Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
PurchaseOperationResponse reponse = purchaseResponse.getResponse();
```

**PHP SDK**

```php
$request = new PurchaseInitializationRequest();
```

**Javascript SDK**

```javascript
const request = new PurchaseInitializationRequest();
```

### 2. Authorize
Once a succesful initialize response has been received, the **purchaseId** can be provided together with the phone number of the customer to the authorize endpoint. This will send a SMS to the provided phone number.

**Rest endpoint**

```json http
{
  "method": "post",
  "url": "https://test-gateway.payolution.com/purchase/authorize/paylater",
  "headers": {
    "paysafe-pl-secret-key": "secret-key",
    "Content-Type": "application/json"
  },
  "body": {
    "purchaseId": "CID-owfqe6dvnhsvp4mkfxuw",
    "phone": "+4300000000000",
    "method": "SMS",
    "successUrl": "https://example.com/successUrl",
    "callbackUrl": "https://example.com/callbackUrl"
  }
}
```

**Java SDK**

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

**PHP SDK**

```php
$request = new PurchaseInitializationRequest();
```

**Javascript SDK**

```javascript
const request = new PurchaseInitializationRequest();
```

### 3. Customer application

The customer will receive an SMS with a link. This link will take the customer through a series of screens for the customer to provide additional details. If any customer details are provided when invoking the initialize call, these details will be prefilled for the customer on these screens. At the end of the customer application, a barcode will be presented.

### 4. Capture

The barcode on the customers device can be scanned by the cash register and should invoke the capture endpoint with the orderId. The orderId is part of the barcode. The purchaseId received in the initialize response can also be used instead of the orderId.

**Rest endpoint**

```json http
{
  "method": "post",
  "url": "https://test-gateway.payolution.com/purchase/capture",
  "headers": {
    "paysafe-pl-secret-key": "secret-key",
    "Content-Type": "application/json"
  },
  "body": {
    "orderId": "75761090",
    "fulfillmentAmount": {
      "amount": 50000,
      "currency": "EUR"
    }
  }
}
```

**Java SDK**

```java
CaptureRequest captureRequest = new CaptureRequest()
  .withOrderId("75761090")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseOperationResponse response = purchaseApi.capture(request, secretKey);
```

**PHP SDK**

```php
$request = new PurchaseInitializationRequest();
```

**Javascript SDK**

```javascript
const request = new PurchaseInitializationRequest();
```

## Start with webapp

This has the least amount of integration needed from the cash register as it will only need to handle capture requests.

### 1. Initialize & authorize

The flow starts with logging into the webapp of Paysafe and creating a new transaction (Contact your account manager for details on the webapp). An amount and customer phone number must be provided to create the transaction.

### 2. Customer application

Once the transaction has been created in the webapp, the customer will receive an SMS with a link. This link will take the customer through a series of screens for the customer to provide additional details. If any customer details were provided when creating the transaction, these details will be prefilled for the customer on these screens. At the end of the customer application, a barcode will be presented.

### 3. Capture

The barcode on the customers device can be scanned by the cash register and should invoke the capture endpoint with the orderId. The orderId is part of the barcode.

**Rest endpoint**

```json http
{
  "method": "post",
  "url": "https://test-gateway.payolution.com/purchase/capture",
  "headers": {
    "paysafe-pl-secret-key": "secret-key",
    "Content-Type": "application/json"
  },
  "body": {
    "orderId": "75761090",
    "fulfillmentAmount": {
      "amount": 50000,
      "currency": "EUR"
    }
  }
}
```

**Java SDK**

```java
CaptureRequest captureRequest = new CaptureRequest()
  .withOrderId("75761090")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseLifecycleApi purchaseApi = new PurchaseLifecycleApi(communicator);
PurchaseOperationResponse response = purchaseApi.capture(request, secretKey);
```

**PHP SDK**

```php
$request = new PurchaseInitializationRequest();
```

**Javascript SDK**

```javascript
const request = new PurchaseInitializationRequest();
```