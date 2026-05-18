*They are just an additional layer in your application.*

*In this case, it's a mongoose layer which allows you to connect with the MongoDB in a much easier and
nicer format.*

![alt text](image-3.png)

## Install 

`npm install mongoose`

```js
"dependencies": { 
    "cors": "^2.8.6",
    "dotenv": "^17.3.1",
    "express": "^5.2.1",
    "mongoose": "^9.3.1"
  }
```

Then to store sensitive information , go to `.env` and write 

```js
MONGO_URI=something
```

create a new file inside `db` -> `index.js` (with any variable name)

```js
∨ db
  index.js
```

Then inside `index.js` , Write :

```js

import mongoose from "mongoose"

// to avoid connection error or any problems , we will put mongoose.connect() inside a try catch block and create a method for that

const connectDB = async() => {

    try {
        await mongoose.connect(process.env.MONGO_URI);
        console.log("✅ MongoDB Connected");
    }catch (error) {
        console.error("❌ MongoDB connection error : " , error);
        process.exit(1);
    }
}

export default connectDB
```

Now , How to get a `MongoDB URI` :

Here we use `MongoDB Atlas`

Get the connection string to connect to cluster 0 

`mongodb+srv://<db_username>:<db_password>@cluster0.7p1tcwv.mongodb.net/`

then add database name at the end : 

`mongodb+srv://<db_username>:<db_password>@cluster0.7p1tcwv.mongodb.net/projmanage`

> Now we need to get `<db_username>` and `<db_password>`

→ Go to `Network Access` 

Now , create an IP Address (if not created) , click on `+ ADD IP ADDRESS`

then create IP Address `0.0.0.0`

![alt text](image-4.png)

> (this is not a production setting , this is only a Development setting so that you can connect to this database from anywhere , once you put your application on production , this is not how we work with that)

After that , we need to have `database access`  

go to `database users` , Click -> `+ ADD NEW DATABASE USER`

![alt text](image-5.png)

Now , replace `<db_username>` and `<db_password>` with the create user `username` and `password`

`MONGO_URI = mongodb+srv://Udemy_user:udemy123456@cluster0.7p1tcwv.mongodb.net/projmanage`

Now go to `index.js` 

We only want listen to port once it is connected to database 

```js
import connectDB from "./db/index.js";

//code...

connectDB()
  .then(() => {
    app.listen(port , () => {
      console.log(`Example app listening on port http://localhost:${port}`);
    })
  })
  .catch((error) => {
    console.error("MongoDB connection error" , error);
    process.exit(1);
  })

```

Now Everything is SET `✅`