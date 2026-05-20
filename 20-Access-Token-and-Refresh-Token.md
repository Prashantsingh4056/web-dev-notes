This whole topic is about:

```txt id="jlwmk1"
How websites keep users logged in securely
```

using:

* JWT
* Access Token
* Refresh Token

---

# First Understand: What is a Token?

A token is simply:

```txt id="0jlwm1"
proof that user is authenticated
```

Think of it like:

* movie ticket
* hotel key card
* entry pass

If you have valid token:

```txt id="1jlwm2"
server trusts you
```

---

# Two Main Types of Tokens

---

# 1. Token WITHOUT Data

Simple random string.

Example:

```txt id="2jlwm3"
ajshd7823hdbsajd8923
```

These are usually used for:

* forgot password
* email verification
* OTP-like systems

---

# How It Works

Suppose user clicks:

```txt id="3jlwm4"
Forgot Password
```

---

## Step 1

Server creates random token:

```txt id="4jlwm5"
abc123xyz
```

---

## Step 2

Server:

* stores token in database
* sends same token in email

---

## Step 3

User clicks reset link:

```txt id="5jlwm6"
website/reset/abc123xyz
```

---

## Step 4

Server compares:

```txt id="6jlwm7"
Does database token match incoming token?
```

If yes:

```txt id="7jlwm8"
allow password reset
```

---

# Important

These tokens:

```txt id="8jlwm9"
contain NO user data
```

Just random strings.

---

# 2. Token WITH Data (JWT)

This is:

```txt id="9jlwm0"
JSON Web Token
```

JWT contains:

* user ID
* email
* role
* expiry time

Example payload:

```json id="0jlwmq"
{
   "id": "123",
   "email": "abc@gmail.com"
}
```

But encrypted/signed.

---

# JWT is Mainly of TWO Types

---

# Access Token

Short-lived token.

Example:

```txt id="1jlwmw"
5 min
15 min
1 hour
```

Used for:

```txt id="2jlwme"
accessing protected routes
```

---

# Refresh Token

Long-lived token.

Example:

```txt id="3jlwմr"
7 days
30 days
```

Used for:

```txt id="4jlwmt"
generating new access tokens
```

---

# VERY IMPORTANT DIFFERENCE

| Access Token             | Refresh Token                       |
| ------------------------ | ----------------------------------- |
| short life               | long life                           |
| used frequently          | used rarely                         |
| sent in every request    | used only when access token expires |
| not stored in DB usually | often stored in DB                  |

---

# FULL LOGIN FLOW

This is the MOST important part.

---

# Step 1 — User Logs In

Frontend sends:

```json id="5jlwmy"
{
   "email": "abc@gmail.com",
   "password": "123456"
}
```

---

# Step 2 — Server Verifies User

Checks database.

If correct:

```txt id="6jlwmu"
user authenticated
```

---

# Step 3 — Server Creates Tokens

Creates:

* access token
* refresh token

---

# Step 4 — Tokens Sent to Client

Client receives:

```txt id="7jlwmi"
access token
refresh token
```

Usually stored in:

* cookies
  or
* localStorage

---

# Access Token Usage

Every protected request sends access token.

Example:

```txt id="8jlwmo"
GET /profile
```

Header:

```txt id="9jlwmp"
Authorization: Bearer accessToken
```

---

# Server Checks Token

Server verifies:

* valid?
* expired?
* signed correctly?

If valid:

```txt id="0jlwma"
allow request
```

---

# Problem: Access Token Expires

Suppose after 15 minutes:

```txt id="1jlwms"
access token expired
```

Server responds:

```txt id="2jlwmd"
401 Unauthorized
```

---

# Then Refresh Token Comes In

Frontend automatically sends:

```txt id="3jlwme"
refresh token
```

to special route:

```txt id="4jlwmy"
/refresh-token
```

---

# Server Checks Refresh Token

Server compares:

* incoming refresh token
* database refresh token

If valid:

```txt id="5jlwmz"
generate NEW access token
```

---

# New Access Token Sent

Frontend receives fresh access token.

User continues normally WITHOUT logging in again.

---

# WHY Use BOTH Tokens?

---

# If Only Long Access Token Exists

Suppose hacker steals token.

Then:

```txt id="6jlwm0"
hacker stays logged in for very long time
```

Dangerous.

---

# Solution

Use:

* short access token
* long refresh token

This improves security.

---

# VERY IMPORTANT CONCEPT

---

# Access Tokens are Stateless

Meaning:

```txt id="7jlwm1"
usually NOT stored in database
```

Server only verifies signature.

---

# Refresh Tokens are Stateful

Meaning:

```txt id="8jlwm2"
often stored in database
```

Why?

Because server may want to:

* revoke sessions
* logout users
* block stolen tokens

---

# SIMPLE ANALOGY

---

## Access Token

Like:

```txt id="9jlwm3"
temporary entry pass
```

---

## Refresh Token

Like:

```txt id="0jlwm4"
master renewal card
```

When entry pass expires:
master card gives new one.

---

# FINAL FLOW

```txt id="1jlwm5"
Login
   ↓
Server creates:
   ↓
Access Token + Refresh Token
   ↓
Client stores them
   ↓
Access token used in requests
   ↓
Access token expires
   ↓
Refresh token sent
   ↓
Server verifies refresh token
   ↓
New access token issued
```

---

# MOST IMPORTANT THING TO REMEMBER

| Token Type    | Purpose            |
| ------------- | ------------------ |
| Random Token  | verification/reset |
| Access Token  | authentication     |
| Refresh Token | renew access token |

This entire mechanism is the foundation of authentication systems in:

* MERN apps
* social media apps
* ecommerce
* dashboards
* SaaS products
* mobile apps
