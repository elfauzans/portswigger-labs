# Lab: Exploiting an API Endpoint Using Documentation

## Description

This lab demonstrates how exposed API documentation can be discovered and used to identify available API endpoints.

The API documentation was exposed through the `/api` endpoint. By accessing the documentation, it was possible to discover a `DELETE` API endpoint that could be used to delete a user account.

## Difficulty

Practitioner

## Vulnerability

Exposed API Documentation

## Category

API Testing

## Tools

- Burp Suite Professional
- Burp Repeater
- Burp Browser

## Credentials

- Username: `wiener`
- Password: `peter`
- Target user: `carlos`

## Steps

1. Log in using the provided credentials.
2. Update the email address to generate an API request.
3. Open **Proxy > HTTP history** in Burp Suite.
4. Find the `PATCH /api/user/wiener` request.
5. Send the request to Burp Repeater.
6. Remove `/wiener` from the endpoint and test `/api/user`.
7. Remove `/user` and test `/api`.
8. The `/api` endpoint exposes API documentation.
9. Open the API documentation in Burp Browser.
10. Identify the `DELETE` endpoint.
11. Enter `carlos` as the username.
12. Send the request.
13. The `carlos` account is successfully deleted and the lab is solved.

## Impact

Exposed API documentation can help attackers discover hidden API endpoints and understand how the application's API works.

If sensitive API endpoints do not properly enforce authorization, attackers may be able to perform unauthorized actions such as deleting user accounts.

## Mitigation

- Restrict API documentation to authorized users.
- Do not expose sensitive API documentation publicly in production.
- Implement proper authentication and authorization for all API endpoints.
- Apply role-based access control (RBAC).
- Do not rely on undocumented or hidden endpoints for security.
- Review API endpoints before deploying the application.

## Key Takeaways

- API documentation can reveal an application's attack surface.
- API endpoints can be discovered by analyzing application requests.
- Burp Repeater is useful for testing API endpoints.
- Exposed documentation can make API reconnaissance easier.
- Sensitive API operations must always enforce proper authorization.

## Screenshots

- Login page
- `PATCH /api/user/wiener` request
- `/api/user` endpoint response
- Exposed API documentation
- DELETE endpoint
- Lab solved

## References

- PortSwigger Web Security Academy - API Testing