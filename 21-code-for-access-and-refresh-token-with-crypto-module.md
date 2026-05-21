So let's go ahead and build three types of token 

Two are going to be with data token known as the **JWT tokens**.

And one is going to be without data Token

Open `.env` file and write : 

`.env` :

```js
ACCESS_TOKEN_SECRET=test
ACCESS_TOKEN_EXPIRY=1d

REFRESH_TOKEN_SECRET=test
REFRESH_TOKEN_EXPIRY=10d
```

*Now in order to generate the refresh token and access token.*

*We do have a library that we need to get this one and there are many others as well.*

*But the most common one is JSON web token.*

Install -> `npm i jsonwebtoken`

Go to `ueser.models.js` : 

```js
//code ...
import jwt from "jsonwebtoken"

// code



```