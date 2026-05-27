Let's first write the validators here : 

Go to `validators` , inside `index.js` : 

```js

const userChangeCurrentPasswordValidator = () => {
  return [
    body("oldPassword").notEmpty().withMessage("Old password is required"),

    body("newPassword").notEmpty().withMessage("New Password is required"),
  ];
};

const userForgotPasswordValidator = () => {
  return [
    body("email")
      .notEmpty()
      .withMessage("Email is required")
      .isEmail()
      .withMessage("Email is invalid"),
  ];
};

const userResetForgotPasswordValidator = () => {
  return [body("newPassword").notEmpty().withMessage("Password is required")];
};

export {
  userRegisterValidator,
  userLoginValidator,
  userChangeCurrentPasswordValidator,
  userForgotPasswordValidator,
  userResetForgotPasswordValidator,
};

```

**All Validators Done till here..**

Coming to `routes` , inside `auth.routes.js` : 

```js
import {
  changeCurrentPassword,
  forgotPasswordRequest,
  getCurrentUser,
  refreshAccessToken,
  registerUser,
  resendEmailVerification,
  resetForgotPassword,
  verifyEmail,
} from "../controllers/auth.controllers.js";

import {
  userChangeCurrentPasswordValidator,
  userForgotPasswordValidator,
  userLoginValidator,
  userRegisterValidator,
  userResetForgotPasswordValidator,
} from "../validators/index.js";

router.route("/verify-email/:verificationToken").get(verifyEmail);

router.route("/refresh-token").post(refreshAccessToken);

router
  .route("/forgot-password")
  .post(userForgotPasswordValidator(), validate, forgotPasswordRequest);

router
  .route("/reset-password/:resetToken")
  .post(userResetForgotPasswordValidator(), validate, resetForgotPassword);

//! Secure Routes
router.route("/logout").post(verifyJWT, logoutUser); // we just pass reference of JWT , here verifyJWT is a middleware

router.route("/current-user").post(verifyJWT, getCurrentUser);

router
  .route("/change-password")
  .post(
    verifyJWT,
    userChangeCurrentPasswordValidator(),
    validate,
    changeCurrentPassword,
  );

router
  .route("/resend-email-verification")
  .post(verifyJWT, resendEmailVerification);


// Small Assignment -> verify all these routes with postman
  
export default router;


```

---
---

## Final Summary

# Backend Auth Flow Notes (Final Revision)

You have now completed the **core authentication system** of a backend application.

The remaining work is mostly:

* adding more routes
* validations
* writing business logic

The **main concepts are already covered**:

* JWT
* Access Token
* Refresh Token
* Cookies
* Middleware
* Protected Routes
* Email Verification
* Forgot Password
* Reset Password
* Change Password
* Validators
* Route Protection

---

# 1. Big Picture of Authentication System

## Authentication Flow

```text
User Registers
      ↓
Server creates user
      ↓
Verification email sent
      ↓
User verifies email
      ↓
User logs in
      ↓
Access Token + Refresh Token generated
      ↓
Tokens stored in cookies
      ↓
Protected routes use verifyJWT middleware
      ↓
User accesses secure APIs
      ↓
Access token expires
      ↓
Refresh token generates new access token
      ↓
User logs out
      ↓
Cookies cleared + refresh token removed from DB
```

---

# 2. What is a Token?

A token is basically a digital identity card.

After login:

```text
Server says:
"Okay, this user is authenticated"
```

Instead of asking password again and again:

```text
Server gives token
```

Now user sends token with every request.

---

# 3. Access Token

Short-lived token.

Example:

```text
Valid for 15 min or 1 day
```

Used for:

* accessing protected routes
* authentication

Stored mostly in:

* cookies
* memory
* local storage (less preferred)

---

# 4. Refresh Token

Long-lived token.

Purpose:

```text
Generate new access token without login again
```

Flow:

```text
Access token expired
        ↓
Client sends refresh token
        ↓
Server verifies refresh token
        ↓
Server issues new access token
```

Refresh token is also stored:

* in cookies
* AND in database

Why DB?
Because logout can invalidate it.

---

# 5. What are Cookies?

Cookies are small pieces of data stored in browser.

Server sends:

```http
Set-Cookie: accessToken=abc123
```

Browser automatically stores it.

Next request:

```http
Cookie: accessToken=abc123
```

Browser automatically sends it.

---

# 6. Why Cookies are Used Here?

Because browser automatically handles them.

Without cookies:

```text
Frontend manually stores token
Frontend manually sends token
```

With cookies:

```text
Browser does everything automatically
```

Very convenient.

---

# 7. httpOnly Cookie

```js
httpOnly: true
```

Meaning:

```text
JavaScript cannot access this cookie
```

Very important for security.

Protects against:

* XSS attacks

---

# 8. secure Cookie

```js
secure: true
```

Meaning:

```text
Cookie only sent over HTTPS
```

Important in production.

---

# 9. What Happens During Login?

## Step-by-step

### Client sends login request

```text
POST /login
email + password
```

---

### Server verifies credentials

```js
user.isPasswordCorrect(password)
```

---

### Generate tokens

```js
generateAccessAndRefreshTokens()
```

---

### Store refresh token in DB

```js
user.refreshToken = refreshToken
```

---

### Send cookies

```js
.cookie("accessToken", accessToken)
.cookie("refreshToken", refreshToken)
```

---

# 10. What is verifyJWT Middleware?

Middleware that protects routes.

Example:

```js
router.route("/current-user")
.post(verifyJWT, getCurrentUser)
```

Flow:

```text
Request comes
      ↓
verifyJWT runs first
      ↓
Token validated
      ↓
User extracted from token
      ↓
req.user added
      ↓
Controller runs
```

---

# 11. Why req.user Exists?

Because middleware adds it.

Inside middleware:

```js
req.user = decodedUser
```

Now controller can use:

```js
req.user._id
```

without querying token again.

---

# 12. Protected Routes

Routes that require authentication.

Examples:

```text
/current-user
/logout
/change-password
/resend-email-verification
```

Protected using:

```js
verifyJWT
```

---

# 13. Logout Flow

## What actually means "logging out user"?

It does NOT:

* delete user account
* remove user data
* remove email/password from DB

It DOES:

* invalidate session/tokens

---

## Logout Steps

### Remove refresh token from DB

```js
refreshToken: ""
```

Meaning:

```text
User session no longer trusted
```

---

### Clear browser cookies

```js
.clearCookie("accessToken")
.clearCookie("refreshToken")
```

Meaning:

```text
Browser no longer sends tokens
```

---

# 14. Why Remove Refresh Token from DB?

Very important.

Suppose attacker stole refresh token.

If user logs out but token still exists in DB:

```text
Attacker can still generate new access tokens
```

Removing refresh token prevents this.

---

# 15. getCurrentUser Route

Very simple route.

Since middleware already added user:

```js
req.user
```

You can directly return it.

```js
return res.json(req.user)
```

---

# 16. Email Verification Flow

## During Registration

### Generate temporary token

```js
generateTemporaryToken()
```

Returns:

* unhashed token
* hashed token
* expiry

---

### Save hashed token in DB

```js
user.emailVerificationToken = hashedToken
```

---

### Send unhashed token in email

Reason:

```text
User should receive readable token
```

---

# 17. Why Hash Token Before Saving?

Security.

If database leaks:

```text
Real token should not be exposed
```

Same idea as password hashing.

---

# 18. Verification Route

Example:

```text
/verify-email/:verificationToken
```

Example actual URL:

```text
/verify-email/abc123xyz
```

---

# 19. req.params

Used for URL parameters.

Example:

```js
const { verificationToken } = req.params
```

Comes from:

```js
"/verify-email/:verificationToken"
```

The `:` defines route parameter.

---

# 20. Email Verification Logic

## Steps

### Get token from URL

```js
req.params
```

---

### Hash token again

```js
crypto.createHash("sha256")
```

---

### Find matching user

```js
emailVerificationToken: hashedToken
```

---

### Check expiry

```js
emailVerificationExpiry: { $gt: Date.now() }
```

Meaning:

```text
Expiry time must be greater than current time
```

---

### Mark verified

```js
user.isEmailVerified = true
```

---

### Cleanup token fields

```js
undefined
```

---

# 21. Resend Email Verification

Purpose:

```text
User didn't verify within expiry time
```

Flow:

* user logged in
* generate new token
* save token
* send email again

---

# 22. Refresh Access Token Flow

When access token expires:

```text
Client sends refresh token
```

---

## Steps

### Read refresh token

```js
req.cookies.refreshToken
```

---

### Verify refresh token

```js
jwt.verify()
```

---

### Decode user ID

```js
decodedToken._id
```

---

### Find user

```js
User.findById()
```

---

### Compare DB refresh token

```js
incomingRefreshToken === user.refreshToken
```

Important security check.

---

### Generate new tokens

```js
generateAccessAndRefreshTokens()
```

---

### Update DB refresh token

```js
user.refreshToken = newRefreshToken
```

---

### Send new cookies

```js
.cookie()
```

---

# 23. Forgot Password Flow

## Step 1

User enters email.

```text
POST /forgot-password
```

---

## Step 2

Server checks user exists.

---

## Step 3

Generate temporary reset token.

---

## Step 4

Save hashed token in DB.

---

## Step 5

Send reset email.

---

# 24. Reset Password Flow

User clicks email link:

```text
/reset-password/:resetToken
```

---

## User enters new password

Request body:

```js
{
  newPassword
}
```

---

## Steps

### Hash reset token

### Find matching user

### Check expiry

### Update password

```js
user.password = newPassword
```

---

### Mongoose pre-save hook hashes password automatically

---

### Cleanup reset token fields

```js
undefined
```

---

# 25. Change Password Flow

Different from forgot password.

## Change Password

User is already logged in.

Needs:

* old password
* new password

---

## Forgot Password

User is NOT logged in.

Uses:

* email
* reset token

---

# 26. Validators

Purpose:

```text
Validate incoming request data
```

Example:

```js
body("email")
.notEmpty()
.isEmail()
```

---

# 27. Why Validators are Useful?

Keeps controller clean.

Without validators:

```text
Controller becomes messy
```

With validators:

```text
Validation separated
```

Better architecture.

---

# 28. validate Middleware

Collects validation errors.

Flow:

```text
Validator runs
      ↓
Errors collected
      ↓
validate middleware checks errors
      ↓
Controller runs only if valid
```

---

# 29. Route Structure Pattern

Pattern repeated everywhere:

```js
router.route("/path")
.post(
    middleware,
    validator(),
    validate,
    controller
)
```

---

# 30. Secure vs Unsecure Routes

## Unsecure Routes

No login needed.

Examples:

* register
* login
* forgot-password
* verify-email
* refresh-token

---

## Secure Routes

Need JWT.

Examples:

* logout
* current-user
* change-password
* resend-email-verification

---

# 31. Core Backend Pattern Learned

Almost entire backend follows this:

```text
Request
   ↓
Middleware
   ↓
Validation
   ↓
Controller
   ↓
Database
   ↓
Response
```

This same architecture is reused in:

* e-commerce
* social media
* SaaS
* dashboards
* enterprise apps

Everything repeats this pattern.

---

# 32. Most Important Learning

The biggest learning is NOT syntax.

It is understanding:

* request flow
* middleware flow
* token flow
* route protection
* database updates
* controller logic

Once flow is understood:

```text
Backend becomes much easier
```
