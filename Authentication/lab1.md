# Lab: Username Enumeration via Different Responses

## Overview

This lab demonstrates how differences in server responses can be used to identify valid usernames, making it easier for attackers to perform password brute-force attacks.

## Difficulty

**Apprentice**

## Vulnerability

- Username Enumeration

## Tools

- Burp Suite Community Edition
- Burp Intruder

## Objective

Identify a valid username by analyzing differences in login responses, then brute-force the corresponding password to gain access to the target account.

## Testing Methodology

1. Intercept the `POST /login` request using Burp Suite.
2. Send the request to Burp Intruder.
3. Perform username enumeration using the provided candidate username list.
4. Compare the server responses to identify a valid username.
5. Replace the username with the valid one.
6. Perform a password brute-force attack using the provided password list.
7. Identify the successful login attempt based on the HTTP `302 Found` response.
8. Log in with the discovered credentials.

## Impact

Username enumeration allows attackers to determine which accounts exist on a system. This information can significantly improve the effectiveness of password spraying, credential stuffing, and brute-force attacks.

## Mitigation

- Return generic error messages for all failed login attempts.
- Implement rate limiting.
- Enable account lockout after multiple failed attempts.
- Require Multi-Factor Authentication (MFA).
- Log and monitor suspicious authentication attempts.

## Screenshots

- Login page
- Intercepted login request
- Username enumeration using Burp Intruder
- Password brute-force using Burp Intruder
- Successful login

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet