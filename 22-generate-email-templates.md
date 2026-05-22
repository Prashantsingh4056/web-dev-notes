We want to register a user , for that the process is : 

* take some data
* validate the data
* check in DB if user already exists
* SAVED the new user
* user verification -> send him an email

For sending an Email , we use some methods and some libraries as well

most common one is `mailgen` , it create beautiful emails 

## Install 
`npm install mailgen`


then go to `utils` folder and create a file `mail.js`

```js
import mailgen from "mailgen";

const emailVerificationMailgenContent = (username, verificationUrl) => {
  return {
    body: {
      name: username,
      intro: "Welcome to our App! we'are excited to have you on board.",
      action: {
        instructions:
          "To verify your email , please click on the following button",
        button: {
          color: "#22BC66",
          text: "Verify your email",
          link: verificationUrl,
        },
      },
      outro:
        "Need help, or have questions? Just reply to this email, we\'d love to help.",
    },
  };
};

const forgotPasswordMailgenContent = (username, passwordResetUrl) => {
  return {
    body: {
      name: username,
      intro: "We got a request to reset the password of your account.",
      action: {
        instructions: "To reset your password click on the button or link",
        button: {
          color: "#22BC66",
          text: "Reset Password",
          link: passwordResetUrl,
        },
      },
      outro:
        "Need help, or have questions? Just reply to this email, we\'d love to help.",
    },
  };
};

export { emailVerificationMailgenContent, forgotPasswordMailgenContent };

```

---
---


# Final Summary : 

What you learned today is:

```txt id="8u2m1q"
How backend generates professional email content
```

NOT sending emails yet.

Only:

```txt id="1k7p4x"
creating email body/template
```

---

# WHY DO WE NEED THIS?

When users:

* register
* reset password
* verify email

backend needs to send emails.

Example:

```txt id="4n8q2m"
"Verify your email"
"Reset your password"
```

Instead of ugly plain text emails:

```txt id="6p1x7v"
Click here: xyz.com
```

we generate:

* styled emails
* buttons
* colors
* proper formatting

using:

```txt id="9m3q5k"
mailgen
```

---

# WHAT IS mailgen?

`mailgen` is a library that:

```txt id="2v7p1x"
creates beautiful HTML email templates
```

It DOES NOT send emails.

It ONLY creates:

```txt id="5x9m2q"
email content/design
```

---

# FLOW

```txt id="7m4p8n"
User registers
   ↓
Backend generates verification token
   ↓
Backend creates email content using mailgen
   ↓
Backend sends email
```

---

# IMPORT

```js id="3p8m1q"
import mailgen from "mailgen";
```

### Meaning

Imports Mailgen library.

---

# emailVerificationMailgenContent()

---

# Function

```js id="8n2x5m"
const emailVerificationMailgenContent = (username, verificationUrl)
```

### Meaning

Function generates:

```txt id="4m7p1x"
email verification email content
```

Parameters:

| Parameter         | Meaning           |
| ----------------- | ----------------- |
| `username`        | user's name       |
| `verificationUrl` | verification link |

---

# return { body: { ... } }

This object structure is required by:

```txt id="6x1m9q"
mailgen
```

---

# name

```js id="1q7p4m"
name: username
```

### Meaning

Used in email greeting.

Example:

```txt id="5v2m8x"
Hello Prashant
```

---

# intro

```js id="9n4p1q"
intro: "Welcome to our App!"
```

### Meaning

Main introduction text.

Shown at top of email.

---

# action

```js id="2m8x5q"
action: {
```

### Meaning

Special section where user performs action.

Usually contains:

* instructions
* button

---

# instructions

```js id="7p1m4x"
instructions:
"To verify your email..."
```

### Meaning

Explains what user should do.

---

# button

```js id="4x9q2m"
button: {
```

### Meaning

Creates clickable email button.

---

# color

```js id="8m2p5x"
color: "#22BC66"
```

### Meaning

Button color (hex color).

---

# text

```js id="1x7m9q"
text: "Verify your email"
```

### Meaning

Button text.

---

# link

```js id="5p2x8m"
link: verificationUrl
```

### Meaning

URL user goes to after clicking button.

Example:

```txt id="9q4m1x"
https://website.com/verify/token123
```

---

# outro

```js id="3m8p2q"
outro:
"Need help..."
```

### Meaning

Bottom/footer text of email.

---

# FINAL GENERATED EMAIL

This function generates something like:

```txt id="6x2m5p"
Hello Prashant

Welcome to our app!

[ Verify your email ]

Need help? Reply to this email.
```

but professionally styled in HTML.

---

# FORGOT PASSWORD EMAIL

Second function:

```js id="2q8m4x"
forgotPasswordMailgenContent()
```

works EXACTLY same way.

Difference is ONLY:

* text
* button
* reset URL

---

# passwordResetUrl

```js id="7m1x5q"
passwordResetUrl
```

### Meaning

Link user clicks to reset password.

Example:

```txt id="4p9m2x"
https://website.com/reset-password/token123
```

---

# WHY TWO DIFFERENT FUNCTIONS?

Because:

* verification email
* password reset email

have different:

* messages
* instructions
* buttons

---

# IMPORTANT THING

These functions:

```txt id="8x2p5m"
DO NOT SEND EMAILS
```

They ONLY:

```txt id="1m7q4x"
generate email content/template
```

Actual sending happens later using:

* nodemailer
* resend
* sendgrid
* mailtrap
  etc.

---

# WHY KEEP THIS IN utils?

Because:

```txt id="5q2m8x"
these are reusable helper functions
```

Any controller can use them.

---

# EXPORT

```js id="9m1x4q"
export {
   emailVerificationMailgenContent,
   forgotPasswordMailgenContent
};
```

### Meaning

Allows other files to import these functions.

---

# HOW IT WILL BE USED LATER

Example:

```js id="3x8m2q"
const mailContent =
emailVerificationMailgenContent(
   user.username,
   verificationUrl
)
```

Then:

```txt id="6m2p9x"
mailContent
```

gets passed to email sender.

---

# COMPLETE REAL FLOW

---

# User Registers

```txt id="2p7m5x"
POST /register
```

---

# Backend Generates Token

```js id="8q1m4x"
generateTemporaryToken()
```

---

# Backend Creates Verification URL

```txt id="4m2x9q"
https://app.com/verify/token
```

---

# Backend Generates Email Template

```js id="7x5m1p"
emailVerificationMailgenContent()
```

---

# Backend Sends Email

using:

```txt id="1q8m4x"
nodemailer
```

---

# User Clicks Button

Backend verifies token.

---

# MOST IMPORTANT THING YOU LEARNED TODAY

You learned:

```txt id="5m1x8q"
separation of concerns
```

Meaning:

| Responsibility         | File         |
| ---------------------- | ------------ |
| Generate token         | model        |
| Generate email content | utils        |
| Send email             | mail service |
| Handle request         | controller   |

This separation is VERY important in scalable backend architecture.
