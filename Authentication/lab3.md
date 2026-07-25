# Lab: Username Enumeration via Response Timing

## Overview

This lab demonstrates how response time differences during the authentication process can reveal valid usernames. Although the application returns identical error messages, processing time varies depending on whether the supplied username exists.

Additionally, the application implements IP-based brute-force protection, which can be bypassed by spoofing the client IP using the `X-Forwarded-For` header.

## Difficulty

**Practitioner**

## Vulnerability

- Username Enumeration
- Authentication Timing Attack

## Tools

- Burp Suite Community Edition
- Burp Intruder
- Burp Repeater

## Objective

Identify a valid username by measuring response times, bypass IP-based brute-force protection, then brute-force the user's password to gain access to the account.

## Testing Methodology

1. Intercept the `POST /login` request using Burp Suite.
2. Send the request to **Burp Repeater**.
3. Test different usernames and observe the response times.
4. Identify that the application accepts the `X-Forwarded-For` header.
5. Use the `X-Forwarded-For` header to spoof different IP addresses and bypass the brute-force protection.
6. Send the request to **Burp Intruder**.
7. Configure a **Pitchfork** attack.
8. Add payload positions for:
   - `X-Forwarded-For`
   - `username`
9. Set the password to a long random string (approximately 100 characters).
10. Load sequential numbers into the `X-Forwarded-For` payload.
11. Load the candidate username list into the username payload.
12. Start the attack.
13. Compare the response times to identify the valid username.
14. Configure a second Intruder attack using the discovered username.
15. Replace the password parameter with a payload position.
16. Load the candidate password list.
17. Start the attack.
18. Identify the successful login by the **HTTP 302 Found** response.
19. Log in using the discovered credentials.

## Key Observation

Although all authentication attempts returned the same error message, requests containing a valid username consistently required more processing time than requests with invalid usernames.

The application's IP-based brute-force protection could be bypassed by modifying the `X-Forwarded-For` header for every request.

## Impact

Timing differences in authentication responses may allow attackers to enumerate valid usernames even when generic error messages are implemented.

Once valid usernames are identified, attackers can perform password spraying, credential stuffing, or brute-force attacks more efficiently.

## Mitigation

- Return consistent response times for both valid and invalid usernames.
- Avoid user-specific processing before authentication is completed.
- Implement strong rate limiting based on multiple factors, not only IP addresses.
- Prevent spoofing of trusted headers such as `X-Forwarded-For`.
- Enable Multi-Factor Authentication (MFA).
- Monitor and alert on abnormal authentication activity.

## Skills Practiced

- Authentication Testing
- Timing Attack Analysis
- Username Enumeration
- Burp Repeater
- Burp Intruder
- HTTP Header Manipulation
- Brute Force Testing

## Screenshots

- Login page
- Intercepted `POST /login`
- Burp Repeater testing
- `X-Forwarded-For` header modification
- Intruder Pitchfork configuration
- Username enumeration results
- Password brute-force results
- Successful login

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet
- OWASP ASVS

## Tags

`Authentication`
`Timing Attack`
`Username Enumeration`
`Burp Suite`
`Burp Repeater`
`Burp Intruder`
`Web Security`

