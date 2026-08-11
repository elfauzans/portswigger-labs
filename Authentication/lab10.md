# Lab: Password Reset Broken Logic

## Overview

This lab demonstrates a broken logic vulnerability in a password reset functionality.

The application generates a password reset token and includes it in the password reset URL. However, the application fails to properly validate this token when the new password is submitted.

Because the username is also submitted as a parameter, an attacker can manipulate the username and reset another user's password without possessing their password reset token.

## Difficulty

**Practitioner**

## Vulnerability

- Broken Password Reset Logic
- Authentication Bypass
- Improper Token Validation
- Parameter Manipulation
- Account Takeover

## Tools

- Burp Suite Community Edition
- Burp Repeater
- Web Browser
- PortSwigger Email Client

## Objective

Exploit the password reset functionality to change the victim's password and use the new password to access the victim's account.

---

## How the Password Reset Works

The application sends a password reset link containing a temporary token:

```text
/forgot-password?temp-forgot-password-token=TOKEN