# Integrations: API integration

API Integration is the most flexible option of allowing payments via Paysafe in the application and process of the merchant. Integration can be done directly via the API or one of our SDKs. This integration has two ways of navigating customers torwards the Paysafe payment application. The URL redirection flow provides the merchant with a URL which can be used to redirect the customer to the Paysafe application from within a browser. The other flow sends the customer a SMS with the link which can be opened on their smartphone and complete the application from there.

**Note**: all SDK examples assume the SDK has been initiliazed. For more details on how to initialize the SDK, check the paragraph 'SDK initialization' in the corresponding SDK documentation.

## URL redirection flow

### 1. Initialize

The flow starts when the customer selects the Pay later option in the system of the merchant. At that point, the system should initialize the payment by providing at least the amount and currency of the order towards the API. Optionaly, customer data can be provided to ease the application process for the customer. The success response of the initialize endpoint returns the **purchaseId** needed for the authorize endpoint.

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
InitializePurchaseRequest request = new InitializePurchaseRequest()
  .withPurchaseAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse = 
  purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
String accessToken = purchaseResponse.getAuthorization();

PurchaseOperationResponse reponse = purchaseResponse.getResponse();
```

### 2. Authorize

Once a succesful initialize response has been received, the **purchaseId** can be provided together with the **successUrl** in which the customer will be redirected back to when the payment has been processed. In addition, a **callbackUrl** can be provided which will receive multiple messages to indicate the status changes of the transaction. The response of this endpoint will include an object called **metaData** with a key **INSTORE_SELFSERVICE_AUTH_URL**. The value of this key contains the URL in which the customer should be redirected towards to complete the Paysafe application.

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
    "method": "URL",
    "successUrl": "https://example.com/successUrl",
    "callbackUrl": "https://example.com/callbackUrl"
  }
}
```

**Java SDK**

```java
AuthorizePurchaseRequest request = new AuthorizePurchaseRequest()
  .withPurchaseId(purchaseId)
  .withMethod(MethodType.URL)
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

PurchaseOperationResponse response = 
  purchaseAuthorizationApi.authorizePaylater(request, secretKey);

String authUrl = reponse.getPurchase().getMetaData().get("INSTORE_SELFSERVICE_AUTH_URL");
```

### 3. Paysafe Pay Later customer application

Once the customer is redirected, they will have to go through a series of screens to provide additional details. If any customer details are provided when invoking the initialize call, these details will be prefilled for the customer on these screens. At the end of the customer application, the customer will be redirected back to the merchant application, which would be the previously given successUrl. If the customer isn't redirected to the successUrl, for any reason, the result can be received via the callbackUrl.

### 4. Capture

After a succesful completion of the application and redirection to the successUrl, or receiving the success message via a callback, the transaction can be captured by the merchant.

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
CapturePurchaseRequest captureRequest = new CapturePurchaseRequest()
  .withOrderId("75761090")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseOperationResponse response = purchaseApi.capture(request, secretKey);
```

### 5. Refund

If any goods are returned, it is possible to (partially) refund the transaction. When refunding a transaction, the purchaseId, amount and currency will need to be provided.

**Rest endpoint**

```json http
{
  "method": "post",
  "url": "https://test-gateway.payolution.com/purchase/refund",
  "headers": {
    "paysafe-pl-secret-key": "secret-key",
    "Content-Type": "application/json"
  },
  "body": {
    "purchaseId": "CID-kdifr9ho54zavijvr9jv",
    "fulfillmentAmount": {
      "amount": 50000,
      "currency": "EUR"
    }
  }
}
```

**Java SDK**

```java
RefundPurchaseRequest request = new RefundPurchaseRequest()
  .withPurchaseId(purchaseId)
  .withRefundAmount(new Amount()
    .withAmount(5000L)
    .withCurrency(Currency.EUR));

PurchaseOperationResponse purchaseResponse = purchaseApi.refund(request, secretKey);
```

## SMS flow

### 1. Initialize

The SMS flow can be started by the merchant from cash register or web application. It could also be an option in your webshop if the customer would like to receive an SMS and continue the purchase on their mobile phone. The system of the merchant should initialize the payment by providing at least the amount and currency of the order towards the API. Optionaly, customer data can be provided to ease the application process for the customer. The success response of the initialize endpoint returns the purchaseId needed for the authorize endpoint.

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
InitializePurchaseRequest request = new InitializePurchaseRequest()
  .withPurchaseAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

ResponseWithAuthorization<PurchaseOperationResponse> purchaseResponse = 
  purchaseApi.intializePurchase(request, secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
String accessToken = purchaseResponse.getAuthorization();

PurchaseOperationResponse reponse = purchaseResponse.getResponse();
```

### 2. Authorize

Once a succesful initialize response has been received, the **purchaseId** can be provided together with the phone number of the customer. A **successUrl** in which the customer will be redirected back to when the payment has been processed. In addition, a **callbackUrl** can be provided which will receive multiple messages to indicate the status changes of the transaction. The customer will receive an SMS with a URL that they can follow to start the application.

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
    "method": "SMS",
    "phone": "+4300000000000",
    "successUrl": "https://example.com/successUrl",
    "callbackUrl": "https://example.com/callbackUrl"
  }
}
```

**Java SDK**

```java
AuthorizePurchaseRequest request = new AuthorizePurchaseRequest()
  .withPurchaseId(purchaseId)
  .withMethod(MethodType.SMS)
  .withPhone('+4300000000000')
  .withSuccessUrl("https://example.com/successUrl")
  .withCallbackUrl("https://example.com/callbackUrl");

PurchaseOperationResponse response = purchaseAuthorizationApi.authorizePaylater(request, secretKey);
```

### 3. Paysafe Pay Later customer application

Once the customer has clicked on the link in the SMS, they will have to go through a series of screens to provide additional details. If any customer details are provided when invoking the initialize call, these details will be prefilled for the customer on these screens. At the end of the customer application, the customer will be redirected back to the merchant application if any succesUrl is provived. If the customer isn't redirected to the successUrl, for any reason, the result can be received via the callbackUrl.

### 4. Capture

After a succesful completion of the application and redirection to the successUrl, or receiving the success message via a callback, the transaction can be captured by the merchant.

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
CapturePurchaseRequest captureRequest = new CapturePurchaseRequest()
  .withOrderId("75761090")
  .withFulfillmentAmount(new Amount()
    .withAmount(50000L)
    .withCurrency(Currency.EUR));

PurchaseOperationResponse response = purchaseApi.capture(request, secretKey);
```

### 5. Refund

If any goods are returned, it is possible to (partially) refund the transaction. When refunding a transaction, the purchaseId, amount and currency will need to be provided.

**Rest endpoint**

```json http
{
  "method": "post",
  "url": "https://test-gateway.payolution.com/purchase/refund",
  "headers": {
    "paysafe-pl-secret-key": "secret-key",
    "Content-Type": "application/json"
  },
  "body": {
    "purchaseId": "CID-kdifr9ho54zavijvr9jv",
    "fulfillmentAmount": {
      "amount": 50000,
      "currency": "EUR"
    }
  }
}
```

**Java SDK**

```java
RefundPurchaseRequest request = new RefundPurchaseRequest()
  .withPurchaseId(purchaseId)
  .withRefundAmount(new Amount()
    .withAmount(5000L)
    .withCurrency(Currency.EUR));

PurchaseOperationResponse purchaseResponse = purchaseApi.refund(request, secretKey);
```

## Optional

### Retrieve transaction status

The merchant application can retrieve the most recent status of the transaction by invoking the purchase info endpoint with the purchaseId, received in the initiliaze call.

**Rest endpoint**

```json http
{
  "method": "get",
  "url": "https://test-gateway.payolution.com/purchase/info/{purchaseId}",
  "headers": {
    "paysafe-pl-secret-key": "secret-key"
  }
}
```

**Java SDK**

```java
PurchaseOperationResponse purchaseResponse = purchaseApi.getPurchase(purchaseId, secretKey);

// OK, NOK, ERROR, PENDING, UNKNOWN
OperationStatus status = purchaseRseponse.getResult().getStatus();

// INITIALIZED, AUTHORIZED, FULFILLMENT, CLOSED
PurchaseState state = purchaseResponse.getPurchase().getState();
```

### Terms and Conditions
The terms and conditions can be retrieved in HTML format by invoking the terms and conditions endpoint with the purchaseId, received in the initiliaze call.

**Rest endpoint**

```json http
{
  "method": "get",
  "url": "https://test-gateway.payolution.com/purchase/legaldocuments/termsandconditions/{purchaseId}",
  "headers": {
    "paysafe-pl-secret-key": "secret-key"
  }
}
```

**Java SDK**

```java
// HTML response
String response = legalDocumentsApi.getTermsAndConditions(purchaseId, secretKey);
```