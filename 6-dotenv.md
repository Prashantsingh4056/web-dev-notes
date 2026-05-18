# dotenv

*Dotenv is a zero-dependency module that loads environment variables from a .env file into process.env*

# Setup

 Install → `npm i dotenv`

 ```
 "dependencies": { 
    "dotenv": "^17.3.1"
  }
 ```

 ## Make a `.env` file in the home dir.
 .env

 ```
port=8000
 ```

 ## How to use this : 

```
 import dotenv from "dotenv"

 // This loads variables from .env into process.env

dotenv.config({
  path: "./.env",
});

const port = process.env.PORT || 3000;

```
