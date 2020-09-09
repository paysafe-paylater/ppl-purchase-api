# Environments

The Paysafe Pay Later API has two environments:

Environment | URL | Description
---------|----------|---------
 Test | https://test-gateway.payolution.com | Used in test environments for integration testing
 Live | https://gateway.payolution.com | Used in production environments for doing live transaction

## Endpoints

The API has three main services that all have a specific endpoint:

Name | URL | Description
---------|----------|---------
 Purchase Lifecycle | /purchase | Handles all necessary steps for a purchase.
 Purchase Authorization | /purchase/authorization/ | Authorizes a consumer to perform operations on an initialized purchase through a web client.
 Legal Documents | /purchase/legaldocuments/ | Provides all documents required for approval during PurchaseLifecycle.
