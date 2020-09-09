# PHP SDK
The Paysafe Pay Later PHP SDK provides a simplified way to integrate with the Paysafe Pay Later instore purchase API. The source of this SDK can be found on [Github](https://github.com/Paysafecard-DEV/ppl-purchase-sdk). This documentation contains information on how to get started with the Paysafe Pay Later PHP SDK.

# Installation
The Paysafe Pay Later PHP SDK can be installed using composer:

```
composer require paysafe/paylater-sdk
```

The package requires the following dependencies, which will automatically be installed when using composer:

Dependency | Version 
--- | ---
guzzlehttp/guzzle | ^6.5
psr/log | ^1.1

## Requirements
The Paysafe Pay Later PHP SDK requires PHP version 7.2 or higher with the following extensions installed:

- json
- openssl


# Authentication
The Paysafe Pay Later PHP SDK can be used with a secret key or authorization token for authentication.
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
Before API calls can be made, the Paysafe Pay Later PHP SDK needs to be initialised. A *Communicator* needs to be created, which needs to be passed to one of the API classes. A *Communicator* requires a *Connection* instance, which may require a *Configuration* object.

Once the *Communicator* has been created the following API classes can be instantiated which can be used to invoke the different endpoints of the purchase API. 

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
Each part of the Paysafe Pay Later PHP SDK communication process implements an interface:

Class | Interface | Description
--- | --- | ---
*ConnectionConfiguration* | *Configuration* | A *Configuration* contains all connection configuration used by the *Connection* factory.
*ApiConnection* | *Connection* | A *Connection* is responsible for the communication with the API endpoints.
*PaySafePayLaterCommunicator* | *Communicator* | A *Communicator* class is responsible for converting API responses into specified *Response* objects.

## Setup example

```php
<?php

use Monolog\Logger;
use Paysafe\Paylater\Api\LegalDocumentsApi;
use Paysafe\Paylater\Api\PurchaseAuthorizationApi;
use Paysafe\Paylater\Api\PurchaseLifecycleApi;
use Paysafe\Paylater\Communication\ApiConnection;
use Paysafe\Paylater\Communication\Communicator;
use Paysafe\Paylater\Communication\Configuration;
use Paysafe\Paylater\Communication\Connection;
use Paysafe\Paylater\Communication\ConnectionConfiguration;
use Paysafe\Paylater\Communication\PaySafePayLaterCommunicator;

$logger = new Logger('paysafe_pay_later');

/** @var Configuration $configuration */
$configuration = new ConnectionConfiguration($url);
$configuration->setLogger($logger);

/** @var Connection $connection */
$connection = ApiConnection::Create($configuration);

/** @var Communicator $communicator */
$communicator = new PaySafePayLaterCommunicator($connection);

$legalDocumentsApi = new LegalDocumentsApi($communicator);
$purchaseAuthorizationApi = new PurchaseAuthorizationApi($communicator);
$purchaseLifecycleApi = new PurchaseLifecycleApi($communicator);
```

## ApiConnection
The *ApiConnection* class implements the *Connection* interface and is used to communicate with the API endpoints. An instance of the *ApiConnection* class can be created using 1 of the 2 factory methods. 

The `create` method will create an *ApiConnection* instance based on a provided *Configuration*. The `createWithClient` method will create a *ApiConnection* instance based on a provided [Guzzle Client](http://docs.guzzlephp.org/en/stable/quickstart.html#creating-a-client).

### Factory: `create`
The `create` method requires a *Configuration* instance which contains all configuration needed to instantiate a Guzzle Client. The provided *ConnectionConfiguration* class contains the baseurl, timeout, debug settings and an optional logger. 

Note when a logger is provided, the logs will automatically be anonymized to prevent sensitive information from ending up in log files. This is done using the *MessageAnonymizerFormatter*.

**Example:**

```php
<?php

use Monolog\Logger;
use Paysafe\Paylater\Communication\ApiConnection;
use Paysafe\Paylater\Communication\ConnectionConfiguration;

$logger = new Logger('paysafe_pay_later');

$connection = ApiConnection::create(
    new ConnectionConfiguration(
        $url,
        30,
        10,
        true,
        $logger
    )
);
```

#### Configuration
The *ConnectionConfiguration* contains the following properties:

Property name | Default value | Description
---------|----------|---------
  baseUrl | <no default, required> | The host of the Paysafe Pay Later server to connect to.
  connectTimeout | 30 | The number of seconds to wait while trying to connect to a server. See [Guzzle Documentation](http://docs.guzzlephp.org/en/stable/request-options.html#connect-timeout)
  readTimeout | 10 | The timeout to use when reading a streamed body. See [Guzzle Documentation](http://docs.guzzlephp.org/en/stable/request-options.html#read-timeout)
  debug | false | Should debug logging be enabled
  logger | null | A Psr\Log\LoggerInterface instance. For example a Monolog instance.
 
### Factory: `createWithClient`
The `create` method requires a *Client* instance.  
Note when logging is required, a logger instance must be provided to the client before creating the *ApiConnection* instance.

**Example:**

```php
<?php

use GuzzleHttp\Client;
use Paysafe\Paylater\Communication\ApiConnection;

$client = new Client([
    'base_uri' => $url,
    'timeout'  => 15.0,
]);

$connection = ApiConnection::createWithClient($client); 
```


## PaySafePayLaterCommunicator
The *PaySafePayLaterCommunicator* class implements the *Communicator* interface and is responsible for converting API responses into specified *Response* objects. A *Connection* is required when creating a *Communicator*.

Example:

```php
<?php

use Paysafe\Paylater\Communication\ApiConnection;
use Paysafe\Paylater\Communication\ConnectionConfiguration;
use Paysafe\Paylater\Communication\PaySafePayLaterCommunicator;

$connection = ApiConnection::create(
    new ConnectionConfiguration($url)
);
$communicator = new PaySafePayLaterCommunicator($connection);
```

## Responses
All endpoints return the same response in the form of a PurchaseOperationResponse object. This object contains all data about the transaction like the status, authorized/captured amount and other metadata. The initialize endpoint however, returns this response differently. The initialize endpoint also returns an access token, and thus the data for both the PurchaseOperationResponse and access token have been wrapped in a seperate object. Below an example of how to handle this case.

```php
$response = $purchaseLifecycleApi->initializePurchase($request, $secretKey);

// Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
$accessToken = $purchaseResponse->getAuthorization();

$response = $purchaseResponse->getResponse();
$purchaseId = $response->getPurchaseId();
```

## Webhooks
Incoming webhooks need to be validated with the default secret key. The Paysafe Pay Later PHP SDK provides a helper class that can be used to validate and decrypt webhook messages. 

**Example:**

```php
<?php

use Paysafe\Paylater\Exception\WebhookDecrypterException;
use Paysafe\Paylater\Webhook\WebhookDecrypter;

// $response contains the raw webhook message body.
$json = json_decode($response, true);

$decrypter = new WebhookDecrypter();
try {
	$response = $decrypter->decrypt($json['infoResponseMessage'], $secretKey);
} catch (WebhookDecrypterException $exception) {
	// Handle WebhookDecrypterException
}
```


## Exceptions
When an error occurs within the Paysafe Pay Later PHP SDK an exception will be thrown. All exceptions thrown by the Paysafe Pay Later PHP SDK extend from the *PaysafePaylaterException* class.

### API Exception
When the Paysafe Pay Later API returns an error, the Paysafe Pay Later PHP SDK will convert the Exception into a *ApiResponseException*. The *ApiResponseException* has a few child exceptions, which are thrown based upon the HTTP status code the API responds with.

The following exceptions can be thrown based on HTTP status code:

Exception | HTTP status code | Description
--- | --- | ---
ValidationException | 400 | Validation of requests failed.
AuthorizationException | 401 or 403 | Authorization failed. Invalid/Expired token or secret key.
ReferenceException | 404, 409 or 410 | Non-existing or removed object is trying to be accessed.
ServerErrorException | 500, 502 or 503 | Something went wrong at the platform or further downstream.
ApiResponseException | - | Indicates an exception regarding the communication with the Paysafe Pay Later platform

**Note:** Responses with a HTTP status code of 200 will never throw an exception, even if the transaction results in a decline. For more information about all possible error codes and their meaning, check the API specification or the status codes page.

**Example:**

Note that *\$purchaseLifecycleApi* must be a valid instance of *PurchaseLifecycleApi*.

```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\Amount;
use Paysafe\Paylater\Model\Currency;
use Paysafe\Paylater\Model\InitializePurchaseRequest;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

$request = new InitializePurchaseRequest(
    new Amount(50000, new Currency(Currency::EUR))
);

try {
    /** @var ResponseWithAuthorization $response */
    $response = $purchaseLifecycleApi->initialize($request, $secretKey);
} catch (ApiResponseException $exception) {
    echo $exception->getErrorId();
    exit;
}
```

#### ApiException
The *ApiException* class has the following properties

Property name | Description
---------|---------
message | Inherited from Exception
code | Inherited from Exception
previous | Inherited from Exception. When available this will contain a Guzzle Exception.
responseBody | Optional: response body
responseHeaders | Instance of *ResponseHeaderCollection* containing all available response headers
errorId | Optional: errorId
operationResult | Optional: instance of *OperationResult*


## Examples
The Paysafe Pay Later PHP SDK contains unit tests for each endpoint, which can be used as a reference point.

### Authorize
Note that *\$purchaseAuthorizationApi* must be a valid instance of *PurchaseAuthorizationApi*.

#### Authorize with SMS
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\AuthorizePurchaseRequest;
use Paysafe\Paylater\Model\MethodType;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

$request = new AuthorizePurchaseRequest(
    $purchaseId,
    new MethodType(MethodType::SMS),
    '+4300000000000',
    'https://example.com/successUrl',
    'https://example.com/callbackUrl'
);

try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseAuthorizationApi->authorizePayLater($request, $secretKey);
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

#### Authorize with redirect URL
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\AuthorizePurchaseRequest;
use Paysafe\Paylater\Model\MethodType;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

$request = new AuthorizePurchaseRequest(
    $purchaseId,
    new MethodType(MethodType::URL),
    null,
    'https://example.com/successUrl',
    'https://example.com/callbackUrl'
);

try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseAuthorizationApi->authorizePayLater($request, $secretKey);
    $authUrl = $response->getPurchase()->getMetaData()['INSTORE_SELFSERVICE_AUTH_URL'];
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

#### Authorize with authorization token
Authorize calls with an authorization token use a different method:

```php
<?php
try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseAuthorizationApi->authorizePayLaterWithAuthorization(
    	$request, 
    	$authorizationToken
    );
    $authUrl = $response->getPurchase()->getMetaData()['INSTORE_SELFSERVICE_AUTH_URL'];
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

The value of variable `$authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.

### Capture
Note that *\$purchaseAuthorizationApi* must be a valid instance of *PurchaseAuthorizationApi*.

#### Capture with purchase ID
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\Amount;
use Paysafe\Paylater\Model\CapturePurchaseRequest;
use Paysafe\Paylater\Model\Currency;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

$request = new CapturePurchaseRequest(
	new Amount(25000, New Currency(Currency::EUR)),
   'CID-kdifr9ho54zavijvr9jv'
);
try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseAuthorizationApi->capturePurchase($request, $secretKey);
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

#### Capture with order ID
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\Amount;
use Paysafe\Paylater\Model\CapturePurchaseRequest;
use Paysafe\Paylater\Model\Currency;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

$request = (new CapturePurchaseRequest(
	new Amount(25000, New Currency(Currency::EUR))
))->setOrderId('75761090');

try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseAuthorizationApi->capturePurchase($request, $secretKey);
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

### Initialize
Note that *\$purchaseLifecycleApi* must be a valid instance of *PurchaseLifecycleApi*.

#### Initialize with minimal required fields
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\Amount;
use Paysafe\Paylater\Model\Currency;
use Paysafe\Paylater\Model\InitializePurchaseRequest;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

$request = new InitializePurchaseRequest(
    new Amount(25000, new Currency(Currency::EUR))
);

try {
    /** @var ResponseWithAuthorization $response */
    $purchaseResponse = $purchaseLifecycleApi->initializePurchase($request, $secretKey);

    // Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
    $accessToken = $purchaseResponse->getAuthorization();

    $response = $purchaseResponse->getResponse();
    $purchaseId = $response->getPurchaseId();
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

#### Initialize with consumer data
```php
<?php

use DateTime;
use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\Address;
use Paysafe\Paylater\Model\Amount;
use Paysafe\Paylater\Model\Consumer;
use Paysafe\Paylater\Model\Country;
use Paysafe\Paylater\Model\Currency;
use Paysafe\Paylater\Model\InitializePurchaseRequest;
use Paysafe\Paylater\Model\Person;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

$person = (new Person())
    ->setFirstName('Ernst')
    ->setLastName('Muller')
    ->setBirthdate(new DateTime('1989-08-22');
    
$billingAddress = (new Address())
    ->setZipCode('5500')
    ->setCity('Bischofshofen')
    ->setStreet('Hauptstrasse')
    ->setHouseNumber('2')
    ->setCountryCode(new Country(Country::AT));

$consumer = (new Consumer())
    ->setEmail('instore-test@paysafe.com')
    ->setPhone('123456789')
    ->setPerson($person)
    ->setBillingAddress($billingAddress);

$request = new InitializePurchaseRequest(
    new Amount(25000, new Currency(Currency::EUR)),
    $consumer
);

try {
    /** @var ResponseWithAuthorization $response */
    $response = $purchaseLifecycleApi->initializePurchase($request, $secretKey);

    // Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
    $accessToken = $purchaseResponse->getAuthorization();

    $response = $purchaseResponse->getResponse();
    $purchaseId = $response->getPurchaseId();
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

### Purchase Info
Note that *\$purchaseLifecycleApi* must be a valid instance of *PurchaseLifecycleApi*.

#### Get purchase information
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseLifecycleApi->getPurchase('CID-kdifr9ho54zavijvr9jv', $secretKey);
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

#### Get purchase information with authorization token
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\PurchaseOperationResponse;

try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseLifecycleApi->getPurchaseWithAuthorization(
    	'CID-kdifr9ho54zavijvr9jv',
    	$authorizationToken
    );
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

The value of variable `$authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.


### Refund
Note that *\$purchaseLifecycleApi* must be a valid instance of *PurchaseLifecycleApi*.

```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;
use Paysafe\Paylater\Model\Amount;
use Paysafe\Paylater\Model\Currency;
use Paysafe\Paylater\Model\PurchaseOperationResponse;
use Paysafe\Paylater\Model\RefundPurchaseRequest;

$request = new RefundPurchaseRequest(
    'CID-kdifr9ho54zavijvr9jv',
    new Amount(5000, New Currency(Currency::EUR))
);

try {
    /** @var PurchaseOperationResponse $response */
    $response = $purchaseLifecycleApi->refundPurchase($request, $secretKey);
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

### Terms and Conditions
Note that *\$legalDocumentsApi* must be a valid instance of *LegalDocumentsApi*.

#### Terms and Conditions
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;

try {
    $response = $legalDocumentsApi->getTermsAndConditions(
        'CID-kdifr9ho54zavijvr9jv',
        $secretKey
    );
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

#### Terms and Conditions with authorization
```php
<?php

use Paysafe\Paylater\Exception\ApiResponseException;

try {
    $response = $legalDocumentsApi->getTermsAndConditionsWithAuthorization(
        'CID-kdifr9ho54zavijvr9jv', 
        $authorizationToken
    );
} catch (ApiResponseException $exception) {
    // Handle ApiResponseException
}
```

The value of variable `$authorizationToken` is part of the `ResponseWithAuthorization` response, received when calling the [initialize](#initialize) api endpoint.
