# Lab: 2FA Simple Bypass

## Overview

This lab demonstrates a simple two-factor authentication (2FA) bypass caused by improper access control.

The application correctly requires a verification code after the username and password have been submitted. However, the application fails to properly enforce the 2FA verification step before allowing access to the user's account page.

By directly navigating to the account page after authentication, an attacker can bypass the 2FA verification process.

## Difficulty

**Practitioner**

## Vulnerability

- Two-Factor Authentication (2FA) Bypass
- Broken Access Control
- Improper Authentication Enforcement

## Tools

- Burp Suite Community Edition
- Web Browser
- Email Client provided by PortSwigger

## Objective

Bypass the victim's 2FA verification step and access the victim's account page without providing the valid 2FA verification code.

## Testing Methodology

### Phase 1 - Analyze the Authentication Flow

1. Log in using the provided account credentials.
2. Retrieve the 2FA verification code from the provided email client.
3. Complete the 2FA verification process.
4. Navigate to the account page.
5. Take note of the account page URL.
6. Log out of the account.

### Phase 2 - Test the 2FA Enforcement

1. Log in using the victim's username and password.
2. Observe that the application requests a 2FA verification code.
3. Do not provide the 2FA verification code.
4. Manually change the URL to the previously identified account page.
5. Observe that the application allows access to the account page despite the 2FA step not being completed.

## Key Observation

The application successfully validates the username and password but does not properly enforce the 2FA verification state before allowing access to protected resources.

The account page can be accessed directly without completing the second authentication factor.

## Impact

An attacker who obtains a user's username and password can bypass the second authentication factor and gain unauthorized access to the victim's account.

This can lead to:

- Account takeover
- Unauthorized access to sensitive information
- Unauthorized modification of account data
- Exposure of user-specific resources

## Root Cause

The application does not properly verify whether the 2FA authentication step has been completed before granting access to the protected account page.

The server appears to rely on the authentication state established after the username and password are validated without enforcing the additional 2FA requirement on protected endpoints.

## Mitigation

- Enforce 2FA verification server-side before granting access to protected resources.
- Maintain a distinct authentication state for partially authenticated users.
- Do not consider a user fully authenticated until all required authentication factors have been verified.
- Validate the authentication state on every protected endpoint.
- Avoid relying on client-side navigation restrictions for security.
- Implement centralized access control middleware for sensitive resources.

## Skills Practiced

- Authentication Testing
- 2FA Security Testing
- Authentication Flow Analysis
- Broken Access Control
- Access Control Testing
- URL Manipulation
- Web Application Security

## Attack Flow

1. Authenticate using valid credentials.
2. Complete 2FA on the attacker's own account.
3. Identify the protected account page.
4. Log out.
5. Authenticate using the target user's credentials.
6. Stop at the 2FA verification step.
7. Directly request the protected account page.
8. Observe unauthorized access without completing 2FA.

## Screenshots

- Login page
- 2FA verification page
- Email client containing the verification code
- Authenticated account page
- Victim login with 2FA prompt
- Direct navigation to `/my-account`
- Successfully accessed account page

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet
- OWASP ASVS

## Lessons Learned

- Authentication is not complete until every required authentication factor has been verified.
- Client-side navigation restrictions should never be relied upon as a security control.
- Protected endpoints must independently verify the user's authentication state.
- A valid username and password should not automatically grant access to resources protected by 2FA.

## Tags

`Authentication`
`2FA`
`MFA`
`2FA Bypass`
`Broken Access Control`
`Authentication Bypass`
`Burp Suite`
`Web Security`