# Connected Applications Made Simple with APIs

> **Warning**
> **This is Jon Catanzano's writing sample for the Comcast Writing Challenge.** This document does not refer to an in-use API.

This repository contains a summary of how you can use these sample API services to create a running application.

#### Contents

- [Overview](#1-overview)
- [API Documentation](#2-api-documentation)
- [API Architecture](#3-api-architecture)
  - [Authorization](#31-authorization)
  - [API Services (Resources)](#32-api-services)
- [API Reference](#4-api-reference)
[Status Code Convention](#5-status-code-convention)
[Testing](#6-testing)

## 1. Overview

Our story offers an easy to understand, practical, high-level description of how you can use API services, also called resources, to select and tailor endpoints (or methods) and other parameters to create an application. In fact APIs make it simpler to communicate with and connect to an application, such as to rapidly validate new service ideas, iterate the design, and quickly deliver services for release.  

This story describes how you can use API services to meet your specific use case requirements and to set up interactions needed to create a running application.  

### Purpose
 
This story provides proper guidance that any decision maker, solution architect, developer, and others can follow to model the services that are available in this sample API. (For this reason, this story is written to be less technical).

It only serves as a high-level guide for flows and should be consulted in tandem with the provided [Open API v3.x Specifications](https://swagger.io/specification/). 

Note that API contracts aren't covered in this story.

### Target Audience

The target audiences for this story is:

* Developers who need to interact intimately with the API.
* Decision-makers and solution architects who need to understand how the API functions.

## 2. API Documentation

## 3. API Architecture



### Potential Flows

The example API contains services (resources) that might be used in an online shopping, ordering, payment, and shipment platform. 

There is also a Dispatcher service that is represented near the end of this story.

### 3.1 Authorization
All user access to this sample API needs is provided through OAuth authorization. For more details, go to the [End User Authentication with OAuth 2.0](https://oauth.net/articles/authentication/) specification. 

The OAuth flow grants the access token, which contains the customerID.
* For this challenge, consider 043fa14b-f6b5-4b96-9cd3-b5f0819b6283 as the customerID.
* Any v4 UUID can be used in examples, where an ID needs to be depicted.

### 3.2 API Services (Resources)

There are seven services (resources) in total. Six of these services have Open API v3.x Specifications (OAS) as described earlier.

These six services and related OAS files are listed below:

* User (users.yaml)
* Orders (orders.yaml)
* Catalogs (catalogs.yaml)
* Carts (carts.yaml)
* Payments (payments.yaml)
* Shipments (shipments.yaml)

####  Path Parameter Naming Convention

* ID ({id}) following a noun in the path always corresponds to the noun.
For example,  /customers/{id}
  * customers is the noun.
  * {id} is the customer ID.
* When the ID ({id}) doesn't correspond to the noun it is explicitly named.
For example,  /carts/{customerId}/items
  * carts is the noun.
  * {customerId} is the customer ID.

## 4. API Reference

### 4.1 Catalog
Catalog is not user specific. All users (authenticated or unauthenticated) can view the available catalog of items.

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List catalog items  |  catalogs.yaml     |  GET |  /catalogs |
| Add catalog items  |  catalogs.yaml  | POST |  /catalogs |
| Update catalog item  |  catalogs.yaml  | POST |  /catalogs |
| Add catalog items  |  catalogs.yaml  | PATCH | /catalogs/{id} |
| Update catalog item  |  catalogs.yaml  | PATCH | /catalogs/{id} |
| Delete catalog item: | catalogs.yaml | DELETE |  /catalogs/{id}

### 4.2 Customer

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| Get customer | users.yaml | GET |   /customers/{id} |
| Get customer | users.yaml | GET | /customers/{id} |
| Add customer | users.yaml | POST | /customers
| Update customer | users.yaml | PATCH | /customers/{id} |
| Delete customer | users.yaml | DELETE | /customers/{id} |

### 4.3 Customer Payment Cards

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List customer payment cards | users.yaml | GET | /customers/{id}/cards |
| Add customer payment cards | users.yaml | POST | /customers/{id}/cards |
| Update customer payment card | users.yaml | PATCH | /cards/{id} |
| Delete customer payment card | users.yaml | DELETE | Path:/cards/{id} |

### 4.4 Customer Addresses

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List customer addresses | users.yaml | GET | /customers/{id}/addresses |
| Add customer addresses | users.yaml | POST | /customers/{id}/addresses |
| Update customer address | users.yaml | PATCH | /addresses/{id} |
| Delete customer address | users.yaml | DELETE | /addresses/{id} |

### 4,5 Customer Cart
| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List customer cart items  |  carts.yaml     |  GET |  /carts/{customerId}/items |
| Update customer cart items  |  carts.yaml | PUT | /carts/{customerId}/items

### 4.6 Customer Order
| Method | File | Operation |  Path | 
|---------------------------------------------- | -------------  | ------  | -----------------  |
| List customer orders | orders.yaml | GET | /orders/{customerId} |
|||| Order Service gets the shipment status from the Shipment Service, using the shipmentID that is sent on a successful shipment creation by the Shipment Service.  See below to learn more about when the shipmentID is generated.
| List customer shipment status | shipments.yaml | GET | /shipments/{id} |
| Add customer order | orders.yaml | POST | /orders/{customerId} |
|||| Extracts cardID from the request and calls Payments Service to authorize the payment.
| Verify customer payment | payments.yaml | POST | /payments |
|||| Payments Service extracts cardID from the request and calls User Service to get the payment card details.
| List customer payment cards | users.yaml | GET | /cards/{id} |
|||| Tries to process the payment, continues if successful, else breaks the transaction.
|||| Extracts customerID from the request path, generates orderID and calls Shipment Service for order fulfillment.
| Add customer shipment | shipments.yaml | POST | /shipments
|||| Shipment Service extracts the customerID and orderID from the request, generates shipmentID and puts a payload with the 3 IDs onto the Dispatcher Queue.
|||| On successful completion, sends back with the shipmentID and status to the Order Service, else breaks the transaction.

### 4.7 Dispatcher
The Dispatcher service picks up the payload placed by the Shipment Service on the dispatcher queue. Dispatcher then extracts the customerID and orderID from the queue payload and calls Order Service to get order item details.

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| Get customer order | orders.yaml | GET | /orders/{customerId}/{orderId} |
|||| Extracts the itemID of each item in the order from the response (n items = n itemIDs = n calls to the Catalog Service) and calls the Catalog Service.
| Get Catalog Item | catalogs.yaml | GET |  /catalogs/{id} |
|||| Extracts addressId from the response and calls User Service to get the address.
| Get customer address | users.yaml | GET | /addresses/{id} |
|||| Ships the package, extracts the shipmentID from the queue payload and calls Shipment Service to update the shipment status
| Update customer shipment status | shipments.yaml | PATCH | /shipments/{id} |

## 5. Status Code Convention
The services send a variety of status codes which are listed in each of the related OAS files, although they can be categorized more generally as follows:
* 2XX: Success response
* 4XX: Client errors
For example, request validation errors
* 5XX: Server errors
For example, database connectivity or internal service call errors

## 6. Testing
TBD.
