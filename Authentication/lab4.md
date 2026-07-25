# Lab: Broken Brute-Force Protection, IP Block

## Overview

This lab demonstrates a logic flaw in the application's brute-force protection mechanism. Although the application temporarily blocks an IP address after multiple failed login attempts, the failed-attempt counter can be reset by successfully authenticating with another valid account.

By exploiting this flaw, an attacker can bypass the IP-based protection and brute-force another user's password.

## Difficulty

**Practitioner**

## Vulnerability

- Broken Authentication
- Weak Brute-Force Protection
- Authentication Logic Flaw

## Tools

- Burp Suite Community Edition
- Burp Intruder

## Objective

Exploit the flawed brute-force protection mechanism to discover the victim's password and gain unauthorized access to the target account.

## Testing Methodology

1. Intercept the `POST /login` request using Burp Suite.
2. Send the request to **Burp Intruder**.
3. Configure a **Pitchfork** attack.
4. Add payload positions for:
   - `username`
   - `password`
5. Create a Resource Pool with:
   - Maximum concurrent requests: **1**
6. Prepare the username payload list by alternating between:
   - Your valid username
   - The victim's username
7. Prepare the password payload list by inserting your valid password before each candidate password.
8. Start the attack.
9. Filter out responses with **HTTP 200 OK**.
10. Identify the request that returned **HTTP 302 Found** for the victim's username.
11. Use the discovered password to log in to the victim's account.

## Key Observation

The application resets the failed login counter after every successful authentication.

By alternating successful logins with brute-force attempts against the victim account, the IP block can be bypassed indefinitely.

## Impact

An attacker can bypass the application's brute-force protection and perform unlimited password guessing attempts against other users.

This significantly increases the likelihood of account compromise through brute-force or password guessing attacks.

## Mitigation

- Track failed login attempts per account, not only per IP address.
- Do not reset failed-attempt counters after unrelated successful logins.
- Implement progressive delays after failed authentication attempts.
- Require Multi-Factor Authentication (MFA).
- Monitor authentication anomalies and brute-force patterns.
- Use CAPTCHA or additional verification after repeated failures.

## Skills Practiced

- Authentication Testing
- Burp Intruder
- Pitchfork Attack
- Resource Pool Configuration
- Brute-Force Testing
- Authentication Logic Analysis

## Screenshots

- Login page
- Intercepted `POST /login`
- Intruder payload configuration
- Resource Pool configuration
- Username and password payload lists
- Attack results
- Successful login

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet
- OWASP ASVS

## Lessons Learned

- Brute-force protection should not rely solely on IP-based tracking.
- Authentication state changes must not unintentionally reset security controls.
- Business logic flaws can completely bypass otherwise effective security mechanisms.
- Successful authentication should never weaken brute-force defenses for other accounts.

## Tags

`Authentication`
`Broken Authentication`
`Brute Force`
`Burp Suite`
`Burp Intruder`
`Pitchfork`
`Business Logic`
`Web Security`