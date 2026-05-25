## What is auth middleware ? 

*Authentication (Auth) middleware is a piece of code that intercepts incoming HTTP requests and verifies a user's identity and permissions before allowing them to access a specific route or resource. If the check fails, it blocks the request; if it passes, it forwards it to the final destination.*

## How it Works (The Request-Response Cycle)

*In modern web development, an application relies on a "stack" of middleware functions that execute in a chain. Auth middleware operates in this chain:*

__**1. The Request:**__ A client (e.g., a web browser or mobile app) sends an HTTP request to an endpoint, usually attaching an authentication token (like a JSON Web Token) in the headers.

**__2. Interception:__** The auth middleware intercepts this request before it reaches your primary controller or database logic.

**__3. Verification:__** The middleware evaluates the token (or session) to see if it is valid, expired, or belongs to an active user.

**__4. Decision:__**

* **_Valid:_** The middleware grants access and passes the request down the chain to the intended destination.

* **_Invalid:_** The middleware terminates the request, sending back an error (like an HTTP 401 Unauthorized) or redirecting the user to a login page.

![alt text](image-33.png)

---
---

![alt text](image-34.png)

Based on the provided architectural diagram, here is the visual workflow broken down step-by-step:
------------------------------
## 1. The Repetitive Route (Top Path)

* **__Client to Server:__** The client sends a direct request containing an accessToken.

* **__Server Processing:__** The request directly hits the server circle.

* **__The Problem:__** The server contains a block of individual controllers. Without interception, the accessToken must travel into each individual controller box inside the cluster, forcing every single controller to manually verify it.

------------------------------
## 2. The Interception Route (Bottom Path)

* **__Client Initiates Request:__** The client sends another request along with the accessToken.

* **__Middleware Interception:__** Before reaching the main server circle, an arrow explicitly breaks away into the auth.middleware component.

* **__The Gatekeeper:__** The request passes through a small middle box (the middleware logic), which validates the accessToken centrally.

------------------------------
## 3. Reaching the Protected Controllers

* **__Forwarding:__** Once auth.middleware successfully verifies the token, the request flows out of the middleware box and into the main server.

* **__Isolated Controllers:__** The request is then safely routed to the specific target box inside the controllers container. Because the middleware already validated the token, these controller blocks remain clean, uncluttered, and free of repetitive verification code.


--- 
Go to `middlewares` folder , create file `auth.middleware.js` : 

__**Step by step plan :**__
 
1. at first , we want to intercept the request in the middleware ,

2. we want to access the access token 

3. once we access the access token , decode the information out 

4. Once we have that decoded information , we will inject this information in the request 


### **Let's work on this :**

`auth.middleware.js`

```js

import { User } from "../models/user.models.js";
import { ApiError } from "../utils/api-error.js";
import { asyncHandler } from "../utils/async-handler.js";
import jwt from "jsonwebtoken";

export const verifyJWT = asyncHandler(async (req, res, next) => {
  const token =
    req.cookies?.accessToken ||
    req.header("Authorization")?.replace("Bearer", ""); // to remove the "Bearer" part

  if (!token) {
    throw new ApiError(401, "Unauthorized request");
  }

  try {
        const decodedToken = jwt.verify(token , process.env.ACCESS_TOKEN_SECRET)

        const user = await User.findById(decodedToken?._id).select(
            "-password -refreshToken -emailVerificationToken -emailVerificationExpiry"
        );

        if(!user){
            throw new ApiError(401 , "Invalid Access Token")
        }

        req.user = user
        next()
  } catch (error) {
            throw new ApiError(401 , "Invalid Access Token")

    }
});

```

![alt text](image-35.png)

![alt text](image-36.png)

## Final Summary : 

# Auth Middleware + Cookies Notes

# 1. Why Auth Middleware is Needed

Many routes need authentication.

Example:

* logout
* profile
* create project
* upload file

Without middleware, every controller must repeatedly check:

```text id="a55y6g"
"Is user logged in?"
```

This creates repeated code.

So we create one common middleware:

```text id="xsm6n8"
verifyJWT
```

which checks authentication before request reaches controller.

---

# 2. Middleware Concept

Middleware = code that runs between request and controller.

## Flow

```text id="p5vgjc"
Client Request
      |
      v
Auth Middleware
      |
      v
Controller
```

---

# 3. Real-Life Analogy

Think of a mall.

* User = customer
* Access Token = ID card
* Middleware = security guard
* Controller = shops inside mall

Security guard checks ID first.

If valid ✅
→ enter mall

If invalid ❌
→ stop there

---

# 4. Why Access Token is Sent in Every Request

HTTP is stateless.

Server forgets user after every request.

So client must send proof again and again.

That proof is:

```text id="x01k0u"
Access Token (JWT)
```

---

# 5. Ways to Send Token

## Method 1 → Cookies

Browser automatically sends token.

```js id="jznk1j"
req.cookies.accessToken
```

---

## Method 2 → Authorization Header

Mostly used in mobile apps.

```text id="7c1mh7"
Authorization: Bearer token_here
```

---

# 6. Why `cookie-parser` is Needed

Express cannot read cookies directly.

Install:

```bash id="e5m9f6"
npm install cookie-parser
```

Use:

```js id="rf1jfj"
app.use(cookieParser())
```

Now Express can access:

```js id="a2p1y0"
req.cookies
```

---

# 7. Cookies Concept

Cookies are small pieces of data stored in browser.

Server sends cookie:

```js id="snnhr2"
res.cookie("accessToken", token)
```

Browser stores it automatically.

Later browser auto-sends it in future requests.

---

# 8. Cookie Flow

```text id="rpbqwf"
Login
   |
   v
Server generates token
   |
   v
Server sends cookie
   |
   v
Browser stores cookie
   |
   v
Browser auto-sends cookie in future requests
```

---

# 9. Cookie Options

## httpOnly

```js id="eq9k7v"
httpOnly: true
```

Browser JavaScript cannot access cookie.

More secure.

---

## secure

```js id="7nvg70"
secure: true
```

Cookie only works on HTTPS.

---

# 10. verifyJWT Middleware

## Purpose

* get token
* verify token
* find user
* attach user to request
* continue request

---

# 11. Step-by-Step Flow

## Step 1 → Get Token

```js id="j94r13"
const token =
    req.cookies?.accessToken ||
    req.header("Authorization")?.replace("Bearer ", "");
```

Meaning:

* first check cookies
* otherwise check Authorization header

---

# 12. Why `.replace("Bearer ", "")`

Header looks like:

```text id="yx43h6"
Bearer abc123token
```

But JWT verify only needs:

```text id="g4th6s"
abc123token
```

So we remove `"Bearer "` part.

---

# 13. If Token Not Found

```js id="eq6iwl"
if (!token)
```

Throw:

```text id="i2d4iq"
Unauthorized request
```

---

# 14. Verify JWT

```js id="9r8iq0"
jwt.verify(token, process.env.ACCESS_TOKEN_SECRET)
```

Purpose:

* check token is real
* decode user information

---

# 15. Decoded Token

JWT contains encoded user info.

Example:

```js id="4p4i1h"
{
   _id,
   email,
   username
}
```

After verification:

```js id="91ho3u"
decodedToken._id
```

becomes available.

---

# 16. Find User from Database

```js id="2lh16y"
User.findById(decodedToken._id)
```

Purpose:

* verify user still exists
* get latest user data

---

# 17. Remove Sensitive Fields

```js id="t3lxwt"
.select("-password -refreshToken")
```

Never send sensitive fields.

---

# 18. Add User to Request

```js id="yn7kaf"
req.user = user
```

This is VERY important.

Now all future controllers can directly access:

```js id="qlf4a4"
req.user
```

---

# 19. Why `req.user`

Because `req` object travels through middleware and controllers.

Middleware adds user info once.

Controllers reuse it.

---

# 20. Continue Request

```js id="sww4wg"
next()
```

Meaning:

```text id="bvtq5o"
"Authentication complete, continue request"
```

---

# 21. Complete Flow

```text id="dh0pk4"
Client Request
      |
      | sends token
      v
verifyJWT Middleware
      |
      | verify token
      | decode token
      | find user
      | req.user = user
      v
Controller
      |
      v
Response
```
