here , everything will revolve around **saving** of the **Data** 

Anything that happens just before saving the data as a **Schema** known as `pre-hook` and just after saving the Data is known as `post-hook` 


So passwords are usually hashed and there are many libraries through which you can hash this 

Example : `bcrypt` or `bcrypt.js`

> A library to help you hash passwords.

Then , install `npm i bcrypt`

and we can encrypt any particular fields using this 

## How to use : 

```js
import bcrypt from "bcrypt";

const userSchema = new Schema(
    // code
);

// ------

userSchema.pre("save" , async function(next){
    // condition check so that password is encrypted only when someone changes or resets 
    // password and not the whole data

    // if the modified field is not the password , then just return next()
    if(!this.isModified("password")) return next()


    this.password = await bcrypt.hash(this.password , 10)
    next()
})

export const User = mongoose.model("User", userSchema);
```