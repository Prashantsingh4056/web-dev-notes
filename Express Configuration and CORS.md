**CORS stands for Cross-Origin Resource Sharing. It is a browser security mechanism that allows a web page from one domain to request and access resources from a different domain.**

`LINK` -> https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS

`app.use` -> *middleware*

```
// basic configurations

app.use(express.json({ limit: "16kb"}))
app.use(express.urlencoded({extended: true, limit: "16kb"}))
app.use(express.static("public"))

```

## CORS
*CORS is a Node.js middleware for Express/Connect that sets CORS response headers. These headers tell browsers which origins can read responses from your server.*

### setup 
`npm install cors`

in `app.js` -> 
```
import cors from "cors";

// basic configurations
...
...

// cors configuration
app.use(cors({
    origin: process.env.CORS_ORIGIN?.split(",") || "http://localhost:5173",
    credentials: true,
    methods: ["GET" , "POST" , "PUT" , "PATCH" , "DELETE" , "OPTIONS"],
    allowedHeaders: ["Content-Type" , "Authorization"],
}),
);

```

.env
```
CORS_ORIGIN =*
# ("https://example.com,https://another.com")
```

# Explanation :

# import express from "express"

## Definition
Imports Express framework.

## Use
Used to create backend server.

## Example
```js
const app = express();
```

---

# import cors from "cors"

## Definition
Imports CORS middleware.

## Use
Allows frontend and backend communication.

## Example
```js
app.use(cors())
```

---

# const app = express();

## Definition
Creates Express application.

## Use
Main backend app object.

## Example
```js
app.listen(8000)
```

---

# app.use(express.json({ limit: "16kb"}));

## Definition
Middleware for reading JSON request body.

## Use
Converts incoming JSON into JavaScript object.

## Example
```js
req.body
```

---

# limit: "16kb"

## Definition
Maximum JSON request size.

## Use
Prevents very large request bodies.

## Example
```txt
Max JSON size = 16kb
```

---

# app.use(express.urlencoded({ extended: true , limit: "16kb"}));

## Definition
Middleware for reading form data.

## Use
Handles HTML form submissions.

## Example
```html
<form>
```

---

# extended: true

## Definition
Allows nested objects in form data.

## Use
Supports complex form structures.

## Example
```js
user[name] = "Prashant"
```

---

# app.use(express.static("public"))

## Definition
Serves static files.

## Use
Makes files accessible publicly.

## Example
```txt
images
CSS files
uploads
```

---

# app.use()

## Definition
Registers middleware in Express.

## Use
Runs middleware before routes.

## Example
```js
app.use(cors())
```

---

# cors()

## Definition
CORS middleware function.

## Use
Controls which frontend can access backend.

## Example
```js
http://localhost:5173
```

---

# origin

## Definition
Allowed frontend URL.

## Use
Restricts API access.

## Example
```js
origin: "http://localhost:5173"
```

---

# process.env.CORS_ORIGIN

## Definition
Reads CORS origin from environment variable.

## Use
Stores sensitive/configurable values.

## Example
```env
CORS_ORIGIN=http://localhost:5173
```

---

# ?.split(",")

## Definition
Splits multiple URLs into array.

## Use
Allows multiple frontend origins.

## Example
```env
URL1,URL2
```

---

# || "http://localhost:5173"

## Definition
Fallback/default value.

## Use
Used if env variable is missing.

## Example
```js
localhost frontend
```

---

# credentials: true

## Definition
Allows cookies/auth headers.

## Use
Used for login/authentication.

## Example
```js
JWT cookies
```

---

# methods

## Definition
Allowed HTTP methods.

## Use
Controls API request types.

## Example
```js
GET
POST
DELETE
```

---

# allowedHeaders

## Definition
Allowed request headers.

## Use
Controls which headers frontend can send.

## Example
```js
Authorization
Content-Type
```



