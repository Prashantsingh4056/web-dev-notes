Now we will write project Validators :

```js

// ------------------------------- Project Validators ----------------------------

const createProjectValidator = () => {
  return [
    body("name")
              .notEmpty()
              .withMessage("Name is required"),

    body("description")
              .optional(),
  ]
}


const addMemberToProjectValidator = () => {

    return [
      body("email")
              .trim()
              .notEmpty()
              .withMessage("Email is Required")
              .isEmail()
              .withMessage("Email is invalid"),

      body("role")
              .notEmpty()
              .withMessage("Role is required")
              .isIn(AvailableUserRole)
              .withMessage("Role is invalid")

    ]
}
 
export {
  userRegisterValidator,
  userLoginValidator,
  userChangeCurrentPasswordValidator,
  userForgotPasswordValidator,
  userResetForgotPasswordValidator,
  createProjectValidator,
  addMemberToProjectValidator
};

```