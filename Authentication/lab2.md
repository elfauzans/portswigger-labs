# Lab: Username Enumeration via Subtly Different Responses

## Overview

This lab demonstrates how subtle differences in server responses can reveal valid usernames. Even when error messages appear identical, small variations such as whitespace or punctuation can enable attackers to enumerate existing accounts.

## Difficulty

**Practitioner**

## Vulnerability

- Username Enumeration

## Tools

- Burp Suite Community Edition
- Burp Intruder

## Objective

Identify a valid username by detecting subtle differences in authentication responses, then brute-force the corresponding password to gain access to the target account.

## Testing Methodology

1. Intercept the `POST /login` request using Burp Suite.
2. Send the request to **Burp Intruder**.
3. Configure the **username** parameter as the payload position.
4. Load the provided candidate username list.
5. Configure **Grep - Extract** to capture the authentication error message.
6. Start the Intruder attack.
7. Compare the extracted responses and identify the username with the slightly different error message.
8. Replace the username with the valid one.
9. Configure the **password** parameter as the payload position.
10. Load the provided candidate password list.
11. Start the password brute-force attack.
12. Identify the successful login attempt based on the **HTTP 302 Found** response.
13. Log in using the discovered credentials.

## Key Observation

The application returned nearly identical error messages for both valid and invalid usernames. However, one response contained a subtle difference (a trailing whitespace instead of a period), allowing the valid username to be identified.

## Impact

Subtle response differences can leak valid usernames, making brute-force, password spraying, and credential stuffing attacks significantly more effective.

## Mitigation

- Return identical authentication responses for both invalid usernames and incorrect passwords.
- Remove any differences in whitespace, punctuation, response length, and formatting.
- Implement rate limiting.
- Enable account lockout after repeated failed login attempts.
- Require Multi-Factor Authentication (MFA).
- Log and monitor suspicious authentication attempts.

## Skills Practiced

- HTTP Request Analysis
- Burp Suite Intruder
- Grep - Extract
- Username Enumeration
- Authentication Testing
- Response Comparison

## Screenshots

- Login page
- Intercepted `POST /login` request
- Intruder payload configuration
- Grep - Extract configuration
- Username enumeration results
- Password brute-force results
- Successful login

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet

## Tags

`Authentication` `Username Enumeration` `Burp Suite` `Intruder` `Web Security`