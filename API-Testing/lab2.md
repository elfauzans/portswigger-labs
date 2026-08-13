# Lab: Finding and Exploiting an Unused API Endpoint

## Description

This lab demonstrates how an unused API endpoint can expose additional functionality that is not directly available through the application's user interface.

By analyzing the API request and changing the HTTP method from `GET` to `PATCH`, it was possible to access an endpoint that allowed the product price to be modified.

The price of the **Lightweight "l33t" Leather Jacket** was changed to `$0.00`, allowing the product to be purchased for free.

## Difficulty

Practitioner

## Vulnerability

Unused API Endpoint

## Category

API Testing

## Tools

- Burp Suite Professional
- Burp Repeater
- Burp Browser

## Credentials

- Username: `wiener`
- Password: `peter`

## Objective

Exploit an unused API endpoint to modify the price of the **Lightweight "l33t" Leather Jacket** and purchase it for `$0.00`.

## Steps

### 1. Identify the Product API

First, I accessed the lab and opened a product page.

In Burp Suite, I opened:

```text
Proxy > HTTP history
```

I identified an API request used to retrieve the product price:

```http
GET /api/products/3/price
```

I sent this request to **Burp Repeater** for further testing.

### 2. Test the OPTIONS Method

In Burp Repeater, I changed the HTTP method from:

```http
GET
```

to:

```http
OPTIONS
```

The server response indicated that the endpoint supported:

```text
GET
PATCH
```

This was an important indication that the API endpoint had additional functionality that was not being used by the normal application interface.

### 3. Test the PATCH Method

I changed the request method from:

```http
GET
```

to:

```http
PATCH
```

The server returned:

```text
Unauthorized
```

This indicated that authentication was required to use the `PATCH` functionality.

### 4. Authenticate to the Application

I logged in to the application using the provided credentials:

```text
Username: wiener
Password: peter
```

After logging in, I opened the **Lightweight "l33t" Leather Jacket** product.

### 5. Identify the Leather Jacket API Endpoint

In Burp Suite HTTP history, I found the API request for the leather jacket price:

```http
GET /api/products/1/price
```

I sent this request to Burp Repeater.

### 6. Change GET to PATCH

I changed the HTTP method from:

```http
GET
```

to:

```http
PATCH
```

The server returned an error indicating that the request had an incorrect `Content-Type`.

The error message specified that the API expected:

```http
Content-Type: application/json
```

This error message helped identify the expected request format.

### 7. Add the Content-Type Header

I added the following HTTP header:

```http
Content-Type: application/json
```

I then added an empty JSON object as the request body:

```json
{}
```

The server returned another error indicating that the `price` parameter was required.

### 8. Modify the Product Price

Based on the error message, I added the required `price` parameter.

The request body became:

```json
{"price":0}
```

The request was then sent through Burp Repeater.

The server accepted the request.

### 9. Verify the Price Change

I returned to the Burp Browser and reloaded the leather jacket product page.

The product price had changed to:

```text
$0.00
```

This confirmed that the API endpoint allowed the authenticated user to modify the product price.

### 10. Purchase the Product

I added the **Lightweight "l33t" Leather Jacket** to the basket.

I then opened the basket and clicked:

```text
Place order
```

The order was successfully placed and the lab was solved.

## Attack Flow

The complete attack can be summarized as:

```text
Product Page
     ↓
Identify API Endpoint
     ↓
GET /api/products/1/price
     ↓
OPTIONS Request
     ↓
Discover PATCH Method
     ↓
PATCH Request
     ↓
Unauthorized
     ↓
Login as wiener
     ↓
PATCH /api/products/1/price
     ↓
Content-Type Error
     ↓
Content-Type: application/json
     ↓
Missing price Parameter
     ↓
{"price":0}
     ↓
Product Price = $0.00
     ↓
Add Product to Basket
     ↓
Place Order
     ↓
Lab Solved
```

## Impact

An attacker who can access an API endpoint with insufficient authorization or business logic controls may be able to modify sensitive application data.

In this lab, the attacker was able to modify the price of a product to:

```text
$0.00
```

This could lead to serious business logic vulnerabilities such as:

- Unauthorized price manipulation
- Purchasing products for an incorrect price
- Financial loss
- Business logic abuse
- Potential fraud

In a real-world application, similar flaws could potentially affect payment amounts, discounts, account balances, inventory, or other sensitive business values.

## Root Cause

The primary issue is that the API exposed a `PATCH` method that allowed the product price to be modified.

The application trusted a client-controlled parameter:

```json
{"price":0}
```

without enforcing sufficient server-side business logic restrictions.

The API also revealed its supported methods through the `OPTIONS` request and provided useful error messages that helped construct a valid request.

## Mitigation

### 1. Restrict HTTP Methods

Only expose HTTP methods that are actually required by the application.

Unused methods such as `PATCH` should be disabled when they are not necessary.

### 2. Implement Proper Authorization

Sensitive API operations must verify that the authenticated user has permission to perform the requested action.

### 3. Protect Sensitive Business Values

Values such as product prices should not be directly controlled by the client.

For example, the server should not blindly accept:

```json
{"price":0}
```

from a normal customer.

### 4. Enforce Server-Side Business Logic

The server should determine the correct product price based on trusted database information.

The client should not be able to arbitrarily modify the price.

### 5. Validate API Input

All API parameters should be validated for:

- Type
- Range
- Format
- Authorization
- Business logic constraints

### 6. Review API Endpoints

Regularly review exposed API endpoints and remove unused functionality.

## Key Takeaways

- API endpoints may contain functionality that is not exposed through the application's UI.
- The `OPTIONS` HTTP method can reveal supported HTTP methods.
- Changing `GET` to `PATCH` can expose additional API functionality.
- Error messages can provide useful information about the expected request format.
- Burp Repeater is useful for manually testing API methods and parameters.
- API security requires both authentication and proper authorization.
- Business logic must always be enforced server-side.
- Sensitive values such as product prices should never be trusted from the client.

## Skills Practiced

- API Reconnaissance
- API Endpoint Discovery
- REST API Testing
- HTTP Method Manipulation
- OPTIONS Method Analysis
- PATCH Request Testing
- Burp Suite Professional
- Burp Repeater
- HTTP Request Analysis
- API Input Validation Testing
- Business Logic Testing
- Authorization Testing

## Screenshots

Recommended screenshots:

1. Product page
2. API request in Burp HTTP history
3. `OPTIONS` request
4. Response showing allowed HTTP methods
5. `PATCH` request
6. `Content-Type` error
7. JSON request body
8. `{"price":0}` request
9. Product price changed to `$0.00`
10. Basket
11. Successful order / lab solved

> Redact passwords, session cookies, API tokens, and other sensitive information before publishing screenshots publicly.

## References

- PortSwigger Web Security Academy - API Testing
- PortSwigger Web Security Academy - Finding and Exploiting an Unused API Endpoint

## Tags

`API Testing`
`API Security`
`API Reconnaissance`
`REST API`
`HTTP Methods`
`PATCH`
`OPTIONS`
`Burp Suite`
`Burp Repeater`
`Business Logic`
`Authorization`
`Web Security`