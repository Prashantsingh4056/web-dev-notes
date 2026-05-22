Let's go ahead and talk about how we are going to register a user.

![alt text](image-18.png)

## Code :

`auth.controllers.js`

```js
import { User } from "../models/user.models.js";
import { ApiResponse } from "../utils/api-response.js";
import { ApiError } from "../utils/api-error.js";
import { asyncHandler } from "../utils/async-handler.js";
import { emailVerificationMailgenContent, sendEmail } from "../utils/mail.js";

const generateAccessAndRefreshTokens = async (userId) => {
    try {
        const user = await User.findById(userId)
        const accessToken = user.generateAccessToken();
        const refreshToken = user.generateRefreshToken();

        user.refreshToken = refreshToken;
        await user.save({validateBeforeSave: false})

        return {accessToken , refreshToken};
    } catch (error) {
        throw new ApiError(500 , "Something went wrong while generating the access token")
    }
}

const registerUser = asyncHandler(async (req, res) => {
  // step1 -> we received the data
  const { email, username, password, role } = req.body; // destructuring

  //? step2 -> we will see validate data later


  // step3 -> Check in the DB if user already exists

  const existedUser = await User.findOne({
    //The findOne() method in MongoDB retrieves exactly one document that matches your search criteria. If multiple documents match, it returns only the first one it encounters based on natural disk order.
    $or: [{ username }, { email }],
  });

  // validate the user
  if (existedUser) {
    throw new ApiError(409, "User with email or username already exists", []);
  }

  // step4 -> SAVE the user
  const user = await User.create({
    email,
    password,
    username,
    isEmailVerified: false,
  });

  // now we need to send an email to the user
  const {unHashedToken , hashedToken , tokenExpiry} = user.generateTemporaryToken();

  user.emailVerificationToken = hashedToken;
  user.emailVerificationExpiry = tokenExpiry;

  await user.save({validateBeforeSave: false});

  await sendEmail(
    {
        email: user?.email,
        subject: "Please verify your email",
        mailgenContent: emailVerificationMailgenContent(
            user.username,
            `${req.protocol}://${req.get("host")}/api/v1/users/verify-email/${unHashedToken}`  // This is the URL 
        ),
    });


    // to remove the fields we don't want to send
    const createdUser = await User.findById(user._id).select(
        "-password -refreshToken -emailVerificationToken -emailVerificationExpiry",
    );

    if(!createdUser){
        throw new ApiError(500 , "Something went wrong while registering a user")
    }

    return res
    .status(201)
    .json(
        new ApiResponse(
            200,
            {user: createdUser},
            "User registered successfully and verification email has been sent on your email"
        )
    )

});

export { registerUser };
```

---
---

# Final Summary

You’ve now reached the point where backend development starts feeling like a **real production system** instead of a small demo app.

This controller is doing a lot more than just:

```js
User.create(req.body)
```

because real applications must handle:

* security
* duplicate users
* email verification
* token generation
* database consistency
* proper API responses
* hiding sensitive data

---

# Big Picture — What Happens During User Registration

The flow shown in your image is exactly how production authentication systems work.

## Complete Flow

```text
1. Take user data
2. Validate data
3. Check if user already exists
4. Save user
5. Generate tokens
6. Save verification token
7. Send verification email
8. Send response back
```

---

# Step-by-Step Explanation

# STEP 1 — Receive Data

```js
const { email, username, password, role } = req.body;
```

When frontend sends:

```json
{
  "email": "abc@gmail.com",
  "username": "prashant",
  "password": "123456"
}
```

Express stores it inside:

```js
req.body
```

You destructure it into variables.

---

# STEP 2 — Validate Data

(Not implemented yet in this video.)

This usually checks:

* empty fields
* valid email
* password length
* username rules

Example:

```js
if (!email || !username || !password) {
   throw new ApiError(400, "All fields are required")
}
```

---

# STEP 3 — Check If User Already Exists

```js
const existedUser = await User.findOne({
   $or: [{ username }, { email }]
});
```

This checks MongoDB.

---

## What `$or` Means

```js
$or: [
   { username },
   { email }
]
```

means:

```text
Find a user where:
username matches
OR
email matches
```

---

## Example

If DB already has:

```json
{
   "email": "abc@gmail.com",
   "username": "prashant"
}
```

and another user tries same email or username:

```js
throw new ApiError(409, "User already exists")
```

---

# Why Status Code 409?

```text
409 = Conflict
```

because requested resource conflicts with existing data.

---

# STEP 4 — Create User

```js
const user = await User.create({
   email,
   password,
   username,
   isEmailVerified: false,
});
```

This creates new MongoDB document.

---

# VERY IMPORTANT

Password gets hashed automatically because earlier you created:

```js
pre("save")
```

middleware using bcrypt.

So password stored in DB becomes:

```text
$2b$10$ksdhfkjsdhf...
```

NOT plain password.

---

# STEP 5 — Generate Temporary Token

```js
const {
   unHashedToken,
   hashedToken,
   tokenExpiry
} = user.generateTemporaryToken();
```

This is for:

* email verification
* password reset

---

# Why TWO Tokens?

You generate:

```text
1. unhashed token
2. hashed token
```

---

# Unhashed Token

Sent to user in email URL.

Example:

```text
abc123xyz789
```

---

# Hashed Token

Stored in database.

Example:

```text
8f3e2f1a9d...
```

---

# Why Hash It?

Because if database leaks:

attacker SHOULD NOT see actual verification tokens.

Exactly same philosophy as passwords.

---

# Flow of Verification Token

## Server creates:

```text
unhashedToken = abc123
hashedToken = SHA256(abc123)
```

---

## Database stores:

```text
hashedToken
```

---

## Email sends:

```text
abc123
```

---

## Later User Clicks Link

Backend receives:

```text
abc123
```

Then backend hashes it again:

```text
SHA256(abc123)
```

and compares with DB.

---

# STEP 6 — Save Verification Token in DB

```js
user.emailVerificationToken = hashedToken;
user.emailVerificationExpiry = tokenExpiry;
```

Now user document contains:

```json
{
   "emailVerificationToken": "...",
   "emailVerificationExpiry": "..."
}
```

---

# Save Without Validation

```js
await user.save({
   validateBeforeSave: false
});
```

This skips validations.

Why?

Because you’re only updating internal backend fields.

No need to validate everything again.

This improves performance.

---

# STEP 7 — Send Email

```js
await sendEmail({...})
```

This is your custom utility.

---

# What Happens Inside `sendEmail()`

It does:

```text
1. Create email HTML
2. Create email text version
3. Connect SMTP server
4. Send email
```

---

# Verification URL Generation

This is VERY important:

```js
`${req.protocol}://${req.get("host")}/api/v1/users/verify-email/${unHashedToken}`
```

---

# Example Output

```text
http://localhost:8000/api/v1/users/verify-email/abc123
```

or in production:

```text
https://myapp.com/api/v1/users/verify-email/abc123
```

---

# Breakdown

## `req.protocol`

Gives:

```text
http
```

or

```text
https
```

---

## `req.get("host")`

Gives:

```text
localhost:8000
```

or:

```text
myapp.com
```

---

## Final URL

```text
http://localhost:8000/api/v1/users/verify-email/token
```

---

# STEP 8 — Remove Sensitive Fields

VERY IMPORTANT SECURITY STEP.

---

## Problem

This:

```js
const user = await User.create(...)
```

contains EVERYTHING:

```json
{
   "password": "...",
   "refreshToken": "...",
   "emailVerificationToken": "..."
}
```

You MUST NOT send these to frontend.

---

# Solution

```js
const createdUser = await User.findById(user._id).select(
   "-password -refreshToken -emailVerificationToken -emailVerificationExpiry"
);
```

---

# What `select()` Does

```js
"-password"
```

means:

```text
exclude password field
```

---

# Final Safe User Object

Frontend receives:

```json
{
   "_id": "...",
   "email": "...",
   "username": "..."
}
```

---

# STEP 9 — Send Final Response

```js
return res.status(201).json(...)
```

---

# Why 201?

```text
201 = Resource Created
```

Perfect status code for registration.

---

# ApiResponse Utility

```js
new ApiResponse(...)
```

creates standardized responses.

Example:

```json
{
   "statusCode": 200,
   "data": {...},
   "message": "User registered successfully"
}
```

---

# MOST IMPORTANT CONCEPT YOU LEARNED

This entire architecture is based on:

# Separation of Concerns

Different files do different jobs.

---

# Your Architecture

| File          | Responsibility         |
| ------------- | ---------------------- |
| Controller    | Handles request flow   |
| Model         | Database structure     |
| Token methods | Generate tokens        |
| Mail utility  | Send emails            |
| Mailgen       | Generate email HTML    |
| ApiError      | Standardized errors    |
| ApiResponse   | Standardized responses |

---

# Why This Is Production Grade

Because code is:

* reusable
* modular
* scalable
* secure
* maintainable
* readable

---

# Services Mentioned in the Video

# 1. Mailtrap

Used for testing emails in development.

Instead of sending real emails to users, Mailtrap captures them safely.

## SMTP Config Example

```env
MAILTRAP_SMTP_HOST=sandbox.smtp.mailtrap.io
MAILTRAP_SMTP_PORT=2525
MAILTRAP_SMTP_USER=your_username
MAILTRAP_SMTP_PASS=your_password
```

---

# 2. Nodemailer

Library used to send emails from Node.js.

## Installation

```bash
npm install nodemailer
```

---

## Basic Usage

```js
import nodemailer from "nodemailer";

const transporter = nodemailer.createTransport({
   host: process.env.MAILTRAP_SMTP_HOST,
   port: process.env.MAILTRAP_SMTP_PORT,
   auth: {
      user: process.env.MAILTRAP_SMTP_USER,
      pass: process.env.MAILTRAP_SMTP_PASS,
   }
});

await transporter.sendMail({
   from: "test@example.com",
   to: "user@gmail.com",
   subject: "Hello",
   text: "Welcome!"
});
```

---

# 3. Mailgen

Used for generating beautiful HTML emails.

## Installation

```bash
npm install mailgen
```

---

## Example

```js
import Mailgen from "mailgen";

const mailGenerator = new Mailgen({
   theme: "default",
   product: {
      name: "Task Manager",
      link: "https://taskmanager.com"
   }
});

const emailHtml = mailGenerator.generate({
   body: {
      name: "Prashant",
      intro: "Welcome!",
   }
});
```

---

# 4. AWS SES (Amazon Simple Email Service)

Production-grade email sending service.

Used in real large-scale applications.

---

## Example

```js
import AWS from "aws-sdk";

const ses = new AWS.SES({
   region: "us-east-1"
});

await ses.sendEmail({
   Source: "test@example.com",
   Destination: {
      ToAddresses: ["user@gmail.com"]
   },
   Message: {
      Subject: {
         Data: "Hello"
      },
      Body: {
         Text: {
            Data: "Welcome!"
         }
      }
   }
}).promise();
```

---

# 5. Brevo (formerly Sendinblue)

Another production email provider.

Used for:

* transactional emails
* newsletters
* OTPs
* verification emails

---

## Example

```js
import SibApiV3Sdk from 'sib-api-v3-sdk';

const client = SibApiV3Sdk.ApiClient.instance;

client.authentications['api-key'].apiKey = process.env.BREVO_API_KEY;
```

---

# One VERY Important Thing You Learned

This line:

```js
await sendEmail(...)
```

is called:

# Abstraction

You hid all email complexity inside one reusable function.

Now controller simply says:

```js
sendEmail(options)
```

instead of rewriting 50 lines every time.

That is how large backend systems are built.
