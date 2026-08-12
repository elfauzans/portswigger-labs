# Lab: Password Brute-Force via Password Change

## Overview

This lab demonstrates how a flawed password change functionality can be abused to enumerate and brute-force a user's current password.

The application returns different error messages depending on whether the submitted current password is correct or incorrect.

By intentionally providing two different new passwords, the account lockout mechanism can be avoided while using the response message to determine whether a password candidate is valid.

The attack can then be automated using Burp Suite Intruder.

> **Note:** All testing was performed within the authorized PortSwigger Web Security Academy lab environment.

---

## Difficulty

**Practitioner**

## Vulnerability

* Password Brute-Force
* Password Enumeration
* Improper Authentication Logic
* Weak Account Lockout
* Information Disclosure Through Error Messages

## Tools

* Burp Suite Community Edition
* Burp Proxy
* Burp Intruder
* Browser

---

## Objective

Use the provided candidate password list to identify Carlos's current password by abusing differences in the password change responses, then log in to Carlos's account.

---

# 1. Analyze the Password Change Functionality

First, I logged in using my own account:

```text
Username: wiener
Password: peter
```

I then investigated the password change functionality from the **My account** page.

The password change request contains the username as a hidden parameter.

The request follows a structure similar to:

```text
POST /my-account/change-password

username=wiener
current-password=...
new-password-1=...
new-password-2=...
```

This is important because the username is controlled by a request parameter and can potentially be modified.

---

# 2. Identify the Response Difference

I tested different combinations of current and new passwords.

The application behaves differently depending on whether the current password is correct.

### Incorrect Current Password

When an incorrect current password is supplied and the two new passwords are different, the application responds with:

```text
Current password is incorrect
```

### Correct Current Password

When the current password is correct but the two new passwords are different, the application responds with:

```text
New passwords do not match
```

This difference allows us to determine whether a password candidate is correct.

---

# 3. Identify the Logic Flaw

The application attempts to prevent brute-force attacks by locking the account after repeated incorrect password attempts.

However, the lockout behavior can be avoided by intentionally making the two new password fields different.

For example:

```text
new-password-1=123
new-password-2=abc
```

Because the new passwords do not match, the application reaches a different validation path.

This allows the response message to reveal whether the submitted current password is correct.

The important distinction is:

```text
Wrong current password
        ↓
"Current password is incorrect"
```

versus:

```text
Correct current password
        ↓
"New passwords do not match"
```

---

# 4. Send the Request to Burp Intruder

I captured the password change request and sent it to **Burp Intruder**.

The request was modified to target Carlos:

```text
username=carlos
```

The `current-password` parameter was marked as the payload position.

The new password parameters were intentionally set to different values:

```text
username=carlos&current-password=§candidate-password§&new-password-1=123&new-password-2=abc
```

This allows the candidate password list to be tested automatically.

---

# 5. Configure the Payload

In Burp Intruder, I added the provided candidate password list as the payload set.

Conceptually:

```text
Candidate Password
        ↓
current-password
        ↓
Send request
        ↓
Analyze response
```

Each password candidate is tested against Carlos's account.

---

# 6. Configure Grep - Match

To make identifying the correct password easier, I configured a **Grep - Match** rule in the Intruder Settings.

The string used was:

```text
New passwords do not match
```

This response indicates that the submitted current password was valid.

The logic is:

```text
"Current password is incorrect"
        ↓
Wrong candidate
```

and:

```text
"New passwords do not match"
        ↓
Correct candidate
```

---

# 7. Identify the Valid Password

After starting the Intruder attack, I reviewed the results.

Most requests returned:

```text
Current password is incorrect
```

One response contained:

```text
New passwords do not match
```

The payload associated with this response was identified as the correct current password for Carlos.

---

# 8. Log in to Carlos's Account

After identifying the correct password, I logged out of my own account.

I then logged in using:

```text
Username: carlos
Password: [REDACTED]
```

After successfully authenticating, I accessed:

```text
My account
```

This completed the lab.

> The recovered password is intentionally redacted from this public write-up.

---

# Attack Flow

The complete attack can be summarized as:

```text
Analyze Password Change
          ↓
Identify Different Error Messages
          ↓
Find Authentication Logic Flaw
          ↓
Target username=carlos
          ↓
Brute-Force current-password
          ↓
Use two different new passwords
          ↓
Compare server responses
          ↓
Find "New passwords do not match"
          ↓
Identify valid password
          ↓
Login as Carlos
          ↓
Access My account
```

---

# Key Observation

The most important finding is that the application leaks information about the validity of the current password through different error messages.

The application should not expose different responses that allow an attacker to distinguish between:

```text
Invalid current password
```

and:

```text
Valid current password
```

The account lockout mechanism also fails to prevent the attack because the attacker can trigger a different validation path by submitting two different new passwords.

---

# Impact

An attacker may be able to:

* Enumerate valid passwords.
* Bypass ineffective brute-force protection.
* Gain unauthorized access to user accounts.
* Perform account takeover.
* Access sensitive account information.

The impact is particularly serious because the attacker does not need to know the victim's existing password beforehand.

---

# Root Cause

The vulnerability is caused by multiple logic flaws.

### 1. Different Error Messages

The application returns different messages depending on the validity of the current password.

```text
Wrong password
→ Current password is incorrect
```

```text
Correct password
→ New passwords do not match
```

This creates a password oracle.

### 2. Weak Account Lockout Logic

The application attempts to lock the account after failed attempts, but the attacker can avoid the lockout behavior by providing mismatched new passwords.

### 3. User-Controlled Username

The username is submitted as a hidden form parameter:

```text
username=carlos
```

This demonstrates why server-side authentication logic should not rely solely on client-controlled parameters.

---

# Mitigation

## Use Consistent Error Messages

Avoid revealing which authentication step failed.

Instead of:

```text
Current password is incorrect
```

and:

```text
New passwords do not match
```

authentication failures should use generic responses where appropriate.

## Implement Robust Rate Limiting

Apply rate limiting based on multiple signals, such as:

* Account
* IP address
* Session
* Device
* Authentication context

## Implement Secure Account Lockout

Account protection should not be dependent on easily bypassed validation paths.

## Require the Current Password

The server should verify the current password before allowing a password change.

## Do Not Trust Client-Controlled Identity Parameters

The authenticated user's identity should come from the server-side session rather than relying on a hidden form field such as:

```text
username=carlos
```

---

# Skills Practiced

* Authentication Testing
* Password Brute-Force
* Password Enumeration
* Burp Suite
* Burp Intruder
* Grep - Match
* HTTP Request Analysis
* Error Message Analysis
* Account Lockout Testing
* Authentication Logic Analysis
* Account Takeover Analysis

---

# Screenshots

Recommended screenshots for this write-up:

1. My account page
2. Password change form
3. Password change request in Burp Proxy
4. Request sent to Burp Intruder
5. Intruder payload position
6. Candidate password list
7. Grep - Match configuration
8. Intruder results
9. `Current password is incorrect` response
10. `New passwords do not match` response
11. Valid password candidate
12. Successful login to Carlos's account

> Redact passwords, session cookies, tokens, and other sensitive information before publishing screenshots.

---

# Lessons Learned

* Different error messages can create an authentication oracle.
* Brute-force protection can fail when it is implemented only around one validation path.
* Authentication responses should not reveal unnecessary information.
* Password change functionality should be tested just like login functionality.
* Hidden form fields are still controlled by the client and should not be trusted for authorization.
* Burp Intruder can automate password testing and help identify subtle response differences.
* Business logic flaws can sometimes bypass otherwise effective security controls.

---

# References

* PortSwigger Web Security Academy
* OWASP Authentication Cheat Sheet
* OWASP Credential Stuffing Prevention Cheat Sheet
* OWASP ASVS

---

# Tags

`Authentication`
`Password Brute Force`
`Password Enumeration`
`Account Lockout`
`Authentication Logic`
`Burp Suite`
`Burp Intruder`
`Error Message Analysis`
`Account Takeover`
`Web Security`
