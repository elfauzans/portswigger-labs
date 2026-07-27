# Lab: Username Enumeration via Account Lock

## Overview

This lab demonstrates how an account lockout mechanism can unintentionally leak valid usernames. By repeatedly attempting authentication with different usernames, the application reveals whether an account exists based on a different lockout response.

Once a valid username is identified, the password can be brute-forced before the account lockout period expires.

## Difficulty

**Practitioner**

## Vulnerability

- Username Enumeration
- Broken Authentication
- Account Lock Logic Flaw

## Tools

- Burp Suite Community Edition
- Burp Intruder

## Objective

Identify a valid username by exploiting the account lockout mechanism, then brute-force the user's password and successfully authenticate.

## Testing Methodology

### Phase 1 - Username Enumeration

1. Intercept the `POST /login` request using Burp Suite.
2. Send the request to **Burp Intruder**.
3. Configure the attack type as **Cluster Bomb**.
4. Add payload positions for:
   - `username`
   - A blank payload at the end of the request body.
5. Load the candidate username list.
6. Configure the second payload using **Null Payloads** with **5 iterations**.
7. Start the attack.
8. Compare the responses and identify the username that triggers the account lockout message.

### Phase 2 - Password Brute Force

1. Create a new Intruder attack.
2. Select **Sniper** attack type.
3. Use the discovered username.
4. Configure the password parameter as the payload position.
5. Load the candidate password list.
6. Configure **Grep - Extract** for the authentication error message.
7. Start the attack.
8. Identify the response that does not contain an authentication error.
9. Wait for the account lockout period to expire.
10. Log in using the discovered credentials.

## Key Observation

After multiple failed login attempts, valid accounts returned a different response indicating that the account had been temporarily locked.

This behavior allowed username enumeration even though the application attempted to protect against brute-force attacks.

During password testing, the successful authentication response contained no error message, making it easy to identify the correct password.

## Impact

Different account lock responses disclose whether a username exists in the system.

This enables attackers to efficiently enumerate valid users before launching password spraying or brute-force attacks.

## Mitigation

- Return identical responses for both valid and invalid usernames.
- Apply account lockout without revealing whether an account exists.
- Use consistent response messages and response lengths.
- Implement rate limiting.
- Require Multi-Factor Authentication (MFA).
- Monitor repeated authentication failures.

## Skills Practiced

- Username Enumeration
- Authentication Testing
- Burp Intruder
- Cluster Bomb Attack
- Sniper Attack
- Grep - Extract
- Response Analysis
- Account Lock Analysis

## Attack Flow

1. Analyze the authentication mechanism.
2. Trigger the account lockout behavior.
3. Identify the valid username.
4. Brute-force the password.
5. Detect the successful authentication.
6. Access the target account.

## Screenshots

- Login page
- POST /login request
- Cluster Bomb configuration
- Username enumeration results
- Account lock response
- Sniper configuration
- Password brute-force results
- Successful login

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet
- OWASP ASVS
- OWASP Testing Guide

## Lessons Learned

- Security controls can unintentionally leak sensitive information.
- Account lockout mechanisms must not disclose whether a username exists.
- Consistent authentication responses are essential to prevent enumeration attacks.
- Authentication defenses should be tested for logic flaws, not only implementation flaws.

## Real-World Scenario

An attacker repeatedly submits authentication requests using different usernames.

If the application returns an account lockout message only for existing accounts, the attacker can enumerate valid users before performing password spraying or credential stuffing attacks.

This issue has been observed in real-world authentication systems where account lockout mechanisms unintentionally disclose account existence.

## Tags

`Authentication`
`Username Enumeration`
`Account Lock`
`Broken Authentication`
`Burp Suite`
`Burp Intruder`
`Cluster Bomb`
`Sniper`
`Web Security`