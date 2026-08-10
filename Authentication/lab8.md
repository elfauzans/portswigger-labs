# Lab: Brute-Forcing a Stay-Logged-In Cookie

## Overview

This lab demonstrates how a weak "Stay logged in" cookie can be brute-forced when its value is predictable.

The application creates the cookie using the following format:

```text
Base64(username + ":" + MD5(password))
```

Because the username and password hash are predictable, an attacker can generate possible cookie values from a password list and test them automatically.

## Difficulty

**Practitioner**

## Vulnerability

- Weak Authentication Cookie
- Predictable Session Token
- Weak Password Hashing
- Brute-Force Attack

## Tools

- Burp Suite Community Edition
- Burp Intruder
- Burp Inspector
- MD5
- Base64

## Objective

Understand how the `stay-logged-in` cookie is generated, then use Burp Intruder to brute-force the victim's cookie and access the victim's account.

---

## How the Cookie Works

First, log in to your own account with **Stay logged in** enabled.

The application creates a cookie similar to:

```text
stay-logged-in=BASE64_VALUE
```

After decoding the Base64 value, we can see a structure similar to:

```text
username:MD5(password)
```

For example:

```text
wiener:51dc30ddc473d43a6011e9ebba6ca770
```

The second part looks like an MD5 hash.

After hashing the known password with MD5, we can confirm that the hash matches.

Therefore, we can determine that the cookie is generated using:

```text
Base64(username + ":" + MD5(password))
```

This is the key discovery in this lab.

---

## Testing Methodology

### Step 1 - Analyze the Cookie

1. Log in to the account with **Stay logged in** enabled.
2. Inspect the `stay-logged-in` cookie using Burp Suite.
3. Decode the cookie from Base64.
4. Identify the username and MD5 hash.
5. Hash the known password using MD5.
6. Compare the result with the decoded cookie.
7. Confirm how the cookie is generated.

### Step 2 - Test the Cookie Against Our Own Account

Before attacking the target account, test the cookie generation method using our own credentials.

Send the request containing the `stay-logged-in` cookie to **Burp Intruder**.

Configure the payload processing rules in this order:

```text
1. Hash → MD5
2. Add prefix → username:
3. Encode → Base64
```

For example:

```text
password
    ↓
MD5(password)
    ↓
username:MD5(password)
    ↓
Base64
    ↓
stay-logged-in cookie
```

Use the known password as a test payload.

If the request successfully loads the authenticated account page, the cookie generation process has been confirmed.

---

## Step 3 - Brute-Force the Target Cookie

After confirming the process works, change the request to target the victim account.

Make these changes:

```text
id=wiener
```

becomes:

```text
id=carlos
```

Change the prefix from:

```text
wiener:
```

to:

```text
carlos:
```

Then replace the test password with the provided candidate password list.

The payload processing remains:

```text
Candidate password
        ↓
     MD5 hash
        ↓
   Add "carlos:"
        ↓
    Base64 encode
        ↓
Generated stay-logged-in cookie
```

Start the Intruder attack.

---

## How to Identify the Correct Cookie

A successful request will behave differently from the failed attempts.

The authenticated **My account** page contains:

```text
Update email
```

Therefore, configure **Grep - Match** in Burp Intruder to search for:

```text
Update email
```

The request containing this text indicates that the generated cookie successfully authenticated as the target user.

The corresponding password payload is the candidate password that produced the valid cookie.

---

## Key Observation

The vulnerability is caused by the predictable structure of the authentication cookie.

The application effectively uses:

```text
Base64(username + ":" + MD5(password))
```

Because the cookie can be recreated from a username and a password guess, an attacker can generate many possible cookies and test them automatically.

The cookie therefore behaves more like a **password-derived authentication token** than a securely generated session token.

---

## Impact

An attacker who knows a user's username and can guess their password may be able to generate a valid `stay-logged-in` cookie.

Successful exploitation could result in:

- Authentication bypass
- Account takeover
- Unauthorized access to private account information
- Persistent access through the "Stay logged in" functionality

## Root Cause

The main problems are:

1. The authentication cookie is derived directly from the password.
2. MD5 is not suitable for password hashing.
3. The cookie contains predictable information.
4. There is no strong random secret or server-side session identifier.
5. The cookie can be brute-forced using a password candidate list.

---

## Mitigation

- Use securely generated random session tokens.
- Do not derive authentication cookies directly from passwords.
- Never use MD5 for password hashing.
- Store passwords using modern password hashing algorithms such as Argon2id, bcrypt, or scrypt.
- Make persistent login tokens random and unpredictable.
- Store persistent session information server-side.
- Rotate and invalidate persistent login tokens when appropriate.
- Apply rate limiting to authentication attempts.
- Provide a mechanism for users to revoke active persistent sessions.

---

## Skills Practiced

- Authentication Testing
- Cookie Analysis
- Burp Suite Inspector
- Burp Intruder
- Payload Processing
- MD5 Hash Analysis
- Base64 Encoding/Decoding
- Authentication Bypass
- Brute-Force Testing

## Attack Flow

```text
Analyze stay-logged-in cookie
            ↓
Decode Base64
            ↓
Identify username + MD5(password)
            ↓
Confirm cookie generation
            ↓
Create password-based cookie payload
            ↓
Target victim username
            ↓
Generate multiple cookie candidates
            ↓
Test with Burp Intruder
            ↓
Find "Update email"
            ↓
Valid authentication cookie
```

## Screenshots

- Stay logged-in cookie
- Decoded cookie value
- MD5 hash comparison
- Burp Intruder payload processing
- MD5 payload processing rule
- Base64 encoding rule
- Grep - Match configuration
- Intruder results
- Successful authenticated response

## Lessons Learned

- Always inspect authentication cookies instead of assuming they are secure.
- Encoding such as Base64 does not provide security.
- Predictable authentication tokens can be brute-forced.
- Password hashes should never be directly used as session identifiers.
- Authentication tokens should be random, unpredictable, and independent from passwords.
- Burp Intruder payload processing can be useful for testing custom token-generation schemes.

## References

- PortSwigger Web Security Academy
- OWASP Authentication Cheat Sheet
- OWASP Session Management Cheat Sheet

## Tags

`Authentication`
`Cookie Security`
`Session Management`
`Brute Force`
`MD5`
`Base64`
`Burp Suite`
`Burp Intruder`
`Authentication Bypass`
`Web Security`