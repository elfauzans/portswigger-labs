# Lab: 2FA Broken Logic

## Overview

This lab demonstrates a broken logic vulnerability in a two-factor authentication (2FA) implementation.

The application uses a `verify` parameter during the 2FA verification process to determine which user's account is being accessed. Because this value can be manipulated, an attacker can change the target username and brute-force the victim's temporary verification code.

This allows the attacker to bypass the intended 2FA protection and gain access to the victim's account.

## Difficulty

**Practitioner**

## Vulnerability

- 2FA Broken Logic
- Authentication Bypass
- Parameter Manipulation
- Improper Authentication State Validation

## Tools

- Burp Suite Community Edition
- Burp Repeater
- Burp Intruder
- Web Browser
- PortSwigger Email Client

## Objective

Exploit the flawed 2FA logic to obtain a valid verification code for the victim account and access the victim's account page.

## Testing Methodology

### Phase 1 - Analyze the 2FA Process

1. Log in using the provided account.
2. Complete the normal 2FA verification process.
3. Use Burp Suite to inspect the 2FA authentication requests.
4. Identify the `POST /login2` request.
5. Observe that the `verify` parameter determines which user's account is being verified.

### Phase 2 - Manipulate the Verification Parameter

1. Log out of the account.
2. Send the `GET /login2` request to **Burp Repeater**.
3. Change the `verify` parameter to the target username.
4. Send the modified request.
5. Observe that a temporary 2FA code is generated for the target account.

### Phase 3 - Brute-Force the 2FA Code

1. Return to the login page.
2. Submit the required username and password.
3. Submit an invalid 2FA code.
4. Intercept the resulting `POST /login2` request.
5. Send the request to **Burp Intruder**.
6. Set the `verify` parameter to the target username.
7. Add a payload position to the `mfa-code` parameter.
8. Load the candidate 2FA codes.
9. Start the Intruder attack.
10. Analyze the responses and identify the request returning **HTTP 302 Found**.
11. Load the successful response in the browser.
12. Navigate to the account page.

## Key Observation

The `verify` parameter is trusted by the application to determine which user's account is being verified.

The application does not properly bind the 2FA verification process to the authenticated user. As a result, the parameter can be modified to target another account.

## Impact

An attacker may be able to bypass two-factor authentication and gain unauthorized access to another user's account.

If the attacker already knows or obtains the victim's username and password, this vulnerability can potentially lead to:

- Account takeover
- Unauthorized access to sensitive information
- Unauthorized modification of account data
- Bypass of the intended MFA security control

## Root Cause

The primary issue is improper server-side validation of the user's authentication state.

The application trusts a client-controlled `verify` parameter instead of securely associating the 2FA verification process with the account that successfully completed the first authentication factor.

## Mitigation

- Never trust client-controlled parameters to determine the identity being authenticated.
- Bind the 2FA verification process to the authenticated session.
- Store the intended user identity server-side.
- Validate that the username in the 2FA request matches the identity authenticated during the first factor.
- Invalidate temporary 2FA codes after successful authentication.
- Limit the number of 2FA verification attempts.
- Implement rate limiting and account lockout for repeated MFA failures.
- Require MFA verification before granting access to protected resources.

## Skills Practiced

- 2FA Security Testing
- Authentication Flow Analysis
- Parameter Manipulation
- Burp Repeater
- Burp Intruder
- MFA Testing
- Authentication Bypass
- Session Analysis

## Attack Flow

1. Analyze the normal 2FA authentication flow.
2. Identify the `verify` parameter.
3. Determine that the parameter controls the target account.
4. Manipulate the parameter to target another user.
5. Generate a temporary 2FA challenge for the target account.
6. Brute-force the verification code.
7. Identify the successful authentication response.
8. Access the target account.

## Screenshots

- Normal 2FA login flow
- `POST /login2` request
- `verify` parameter in Burp Repeater
- Modified `GET /login2` request
- Invalid 2FA request
- Intruder configuration
- MFA code brute-force results
- Successful `302 Found` response
- Access to the account page

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet
- OWASP ASVS

## Lessons Learned

- Client-controlled parameters should never determine the identity being authenticated.
- MFA verification must be tied to the authenticated session.
- Authentication logic should always be enforced server-side.
- MFA can be bypassed when the application fails to properly associate the second authentication factor with the correct user.
- Security mechanisms should be tested for logic flaws, not only technical implementation flaws.

## Tags

`Authentication`
`2FA`
`MFA`
`2FA Broken Logic`
`Authentication Bypass`
`Parameter Manipulation`
`Burp Suite`
`Burp Repeater`
`Burp Intruder`
`Web Security`