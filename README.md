# Connected Applications Made Simple with APIs

> **Warning**
> **This is Jon Catanzano's writing sample for the Comcast Writing Challenge.** This document does not refer to an in-use API.

This repository contains documentation that summarizes how you can use these sample API services to create a running application.

#### Contents

- [Overview](#1-overview)
- [API Architecture](#2-api-architecture)
  - [Authorization](#21-authorization)
  - [API Services](#22-api-services)
  - [Path Parameter Naming](#23-path-parameter-naming)
- [API Reference](#3-api-reference)
  - [Catalog](#31-catalog)
  - [Customer](#32-customer)
  - [Customer Payment Cards](#33-customer-payment-cards)
  - [Customer Addresses](#34-customer-addresses)
  - [Customer Cart](#35-customer-cart)
  - [Customer Order](#36-customer-order)
  - [Dispatcher](#37-dispatcher)
- [Return Code Convention](#4-return-code-convention)
- [Testing](#5-testing)

## 1. Overview

Our story offers an easy-to-follow, practical, high-level description of how you can use API services, also called resources, to select and tailor endpoints (or methods) and other parameters to set up an application. In fact APIs make it simpler to communicate with and connect to an application, such as to rapidly validate new service ideas, iterate the design, and quickly deliver services for release.  

This story describes how you can use API services to meet your specific use case requirements and set up the interactions needed to create a running application.  

### Purpose
 
This story provides guidance that decision makers, solution architects, developers, and technicians can follow to model the API services that are available in sample architecture below. For this reason, this story is written to be less technical.

As a high-level guide, this story should be read in tandem with the [Open API v3.x Specifications](https://swagger.io/specification/) for more details. 

(Note that API contracts aren't covered in this story).

### Target Audience

The target audiences for this story is:

* Decision-makers and solution architects who need to understand how the API functions and what it can do.
* Developers and other technicians who need to interact intimately with the API.

## 2. API Architecture

The architecture for the sample API applied in this story is shown below. 

![image](https://user-images.githubusercontent.com/16295975/227042763-0a260225-ba70-4ab8-baab-3885fd987116.png)


The sample API architecture illustrates the services (resources) that might be used in an online shopping, ordering, payment, and shipment platform.

The architecture also shows the status and error return codes in the event the service request is invalid, review [Return Code Convention](#4-return-code-convention) for more details.  

### 2.1 Authorization

All user access to this sample API is provided through OAuth authorization. This story should be read in tandem with the [End User Authentication with OAuth 2.0](https://oauth.net/articles/authentication/) for more details. 

The OAuth flow grants the access token, which contains the customerID.

`redirect_uri`

* For this sample, the customerID is: `043fa14b-f6b5-4b96-9cd3-b5f0819b6283`.
* Any v4 UUID can be used in samples where an ID needs to be depicted.

### 2.2 API Services

There are seven services (resources) in total. All are represented in the API Reference.

Six of these API services use Open API v3.x Specifications (OAS) as listed below:

* User (users.yaml)
* Orders (orders.yaml)
* Catalogs (catalogs.yaml)
* Carts (carts.yaml)
* Payments (payments.yaml)
* Shipments (shipments.yaml)

The last API service is listed as:

* Dispatcher (dispatcher.yaml)

The Dispatcher service does not require Open API v3.x Specifications (OAS). 

### 2.3 Path Parameter Naming

The path parameter naming convention is described below:

* ID ({id}) following a noun in the path always corresponds to the noun.
  For example, for '/customers/{id}':
  * 'customers' is the noun.
  * '{id}' is the customer ID.
* When the ID ({id}) doesn't correspond to the noun it is explicitly named.
  For example, for '/carts/{customerId}/items':
  * 'carts' is the noun.
  * '{customerId}' is the customer ID.

## 3. API Reference

### 3.1 Catalog

Catalog is not user-specific. All users (authenticated or unauthenticated) can view the available catalog of items.

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List catalog items  |  catalogs.yaml     |  GET |  /catalogs |
| Add catalog items  |  catalogs.yaml  | POST |  /catalogs |
| Update catalog item  |  catalogs.yaml  | POST |  /catalogs |
| Add catalog items  |  catalogs.yaml  | PATCH | /catalogs/{id} |
| Update catalog item  |  catalogs.yaml  | PATCH | /catalogs/{id} |
| Delete catalog item: | catalogs.yaml | DELETE |  /catalogs/{id}

### 3.2 Customer

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| Get customer | users.yaml | GET |   /customers/{id} |
| Get customer | users.yaml | GET | /customers/{id} |
| Add customer | users.yaml | POST | /customers
| Update customer | users.yaml | PATCH | /customers/{id} |
| Delete customer | users.yaml | DELETE | /customers/{id} |

### 3.3 Customer Payment Cards

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List customer payment cards | users.yaml | GET | /customers/{id}/cards |
| Add customer payment cards | users.yaml | POST | /customers/{id}/cards |
| Update customer payment card | users.yaml | PATCH | /cards/{id} |
| Delete customer payment card | users.yaml | DELETE | Path:/cards/{id} |

### 3.4 Customer Addresses

| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List customer addresses | users.yaml | GET | /customers/{id}/addresses |
| Add customer addresses | users.yaml | POST | /customers/{id}/addresses |
| Update customer address | users.yaml | PATCH | /addresses/{id} |
| Delete customer address | users.yaml | DELETE | /addresses/{id} |

### 3.5 Customer Cart
| Method | File | Operation |  Path | 
|-----------------------------------  | -------------    | ------  | ----------------------------------  |
| List customer cart items  |  carts.yaml     |  GET |  /carts/{customerId}/items |
| Update customer cart items  |  carts.yaml | PUT | /carts/{customerId}/items

### 3.6 Customer Order
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

### 3.7 Dispatcher
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

## 4. Return Code Convention

The services send a variety of status codes which are listed in each of the related OAS files, although they can be categorized more generally as follows:
* 2XX: Success response
* 4XX: Client errors
For example, request validation errors
* 5XX: Server errors
For example, database connectivity or internal service call errors

## 5. Testing
TBD.  
