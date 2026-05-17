## Professional Folder Structure

```
Public 
    images

src 
    controllers
    db
    middlewares
    models
    routes
    utils
    validators

```

Move `index.js` inside the `src` folder

# Public

## Definition
Stores static files.

## Use
Images, uploads, PDFs, icons.

## Example
```txt
profile pictures
product images
```

---

# src

## Definition
Main backend source code folder.

## Use
Contains all backend logic.

## Example
```txt
controllers
routes
models
```

---

# controllers

## Definition
Contains route logic/functions.

## Use
Handles request processing.

## Example
```js
loginUser()
registerUser()
```

---

# db

## Definition
Database connection setup.

## Use
Connect backend to database.

## Example
```js
mongoose.connect()
```

---

# middlewares

## Definition
Functions that run before controller.

## Use
Authentication, logging, error handling.

## Example
```js
app.use(authMiddleware)
```

---

# models

## Definition
Database schema structure.

## Use
Defines stored data format.

## Example
```js
const userSchema = new mongoose.Schema({
   name: String
})
```

---

# routes

## Definition
Defines API endpoints.

## Use
Connects URL to controller.

## Example
```js
router.post("/login")
```

---

# utils

## Definition
Reusable helper functions.

## Use
Common utility operations.

## Example
```js
generateToken()
```

---

# validators

## Definition
Checks incoming data validity.

## Use
Validates request data.

## Example
```js
if(password.length < 6)
```