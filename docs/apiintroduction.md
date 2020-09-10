# Introduction
Welcome to the Paysafe Pay Later Developer Hub. On this page you will find everything you need to start integrating with the Paysafe Pay Later platform.

## Integrations

The Paysafe Pay Later platform offers easy and flexible ways of integrating with your application and processes. The platform supports three different ways of integration:

**Zero Integration**

Allows for instore payments without any integration. The only thing needed is access to the webapp of Paysafe. Read more about how to use this integration on the [Zero integration page](./integration-zero.md).

**Cash Register Integration**

Integration with a cash register provides secure confirmation of payments with little effort. Integration can be done via one of the SDKs or directly through our RESTfull API. More information about this integration can be found on the [Cash register integration page](./integration-cash-register.md).

**API Integration**

API Integration is the most flexible option of allowing payments via Paysafe in your application and process. Using one of our provided SDKs will make integration as easy as possible. Find out more about the possiblities by navigating to the [API integration page](./integration-api.md).

## Server SDKs
Server SDKs contain all functionality of the Paysafe Pay Later API. As most functionality requires a secret key, these SDKs should only be used on a server. The SDKs are based on the API specification and provide the following functionality:

- Handles authentication in all requests towards the API
- Wrapper around all API calls and responses to make building a request and interpreting responses as easy as possible
- Takes care of marshalling and unmarshalling request and responses
- Processes errors from the API and transforms them in specific exceptions
- Handles decrypting of incoming webhook messages

Paysafe Pay Later provides the following Server SDKs:
- [Java SDK](./java-sdk.md)
- [PHP SDK](./php-sdk.md)
- [Node.js SDK](./nodejs-sdk.md)

## Client SDKs
Client SDKs are used by client-side applications such as a mobile phone, tablet or browser on a desktop computer. Theses SDKs are limited in functionality as they are only allowed to invoke certain methods of the API. These calls require a valid access token that you can create using the server SDK. 

Paysafe Pay Later provides the following Client SDKs:
- [Javascript SDK](./javascript-sdk.md)