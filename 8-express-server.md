## Install Express

`npm install express`

**After that →** 
```
"dependencies": { 
    "dotenv": "^17.3.1",
    "express": "^5.2.1",
  }
```

to install latest Version -> `npm install express@latest`

How to use it -> 
`import express from "express"`

example : 

```
import express from "express"

const app = express();
const port = process.env.PORT || 3000;

app.get("/instagram" , (req , res) => {
    res.send("this is an instagram page")
})

app.listen(port , () => {
    console.log(`Example app listening on port https://localhost:${port}`);
})
```
