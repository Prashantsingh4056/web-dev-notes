Now , Lets write the routes for each controller 

Create a file `project.routes.js` inside `Routes` Folder

```js

import { Router } from "express";
import {
  addMembersToProject,
  updateMemberRole,
  updateProject,
  getProjectById,
  deleteMember,
  deleteProject,
  getProjectMembers,
  getProjects,
  createProject,
} from "../controllers/auth.controllers.js";
import { login } from "../controllers/project.controllers.js";

import { validate } from "../middlewares/validator.middleware.js";

import {
    createProjectValidator , addMemberToProjectValidator
} from "../validators/index.js";

import { verifyJWT , validateProjectPermission} from "../middlewares/auth.middleware.js";

import { logoutUser } from "../controllers/auth.controllers.js";
import { AvailableUserRole, UserRolesEnum } from "../utils/constants.js";

const router = Router();

// middleware 
router.use(verifyJWT);    // whatever we write after this line , will all have a verifyJWT

router
    .route("/")
    .get(getProjects)
    .post(createProjectValidator() , validate , createProject)


router
    .route("/:projectId")   // req.params
    .get(validateProjectPermission(AvailableUserRole) , getProjectById)  // here everyone is allowed to access this , if we want only for certain roles , we can do that so
    .put(
        validateProjectPermission([UserRolesEnum.ADMIN]),
        createProjectValidator,
        validate,
        updateProject
    ).delete(
        validateProjectPermission([UserRolesEnum.ADMIN]),
        deleteProject
    );


router  
    .route("/:projectId/members")   // if we put : , it will be considered as params
    .get(getProjectMembers)
    .post(
        validateProjectPermission([UserRolesEnum.ADMIN]),
        addMemberToProjectValidator(),
        validate,
        addMembersToProject
    )


router
    .route("/:projectId/members/:userId")
    .put(validateProjectPermission([UserRolesEnum.ADMIN]) , updateMemberRole)
    .delete(validateProjectPermission([UserRolesEnum.ADMIN]) , deleteMember)

export default  router
```


Then Go to `app.js` and write : 

```js

import projectRouter from "./routes/project.routes.js";
app.use("/api/v1/projects" , projectRouter)

```

---
# Explanation
---

This file is the **Project Routes Layer** of your backend.

Remember the architecture:

```text
Client
  ↓
Route
  ↓
Middleware
  ↓
Controller
  ↓
Database
```

This file is responsible for:

* Matching URLs
* Running middlewares
* Calling the correct controller

---

# Step 1: Imports

```js
const router = Router();
```

Creates an Express router.

Think of it as:

```text
Project Module
     |
     ├── GET /projects
     ├── POST /projects
     ├── GET /projects/:projectId
     ├── PUT /projects/:projectId
     └── DELETE /projects/:projectId
```

---

# Step 2: Global Middleware

```js
router.use(verifyJWT);
```

Very important.

This means:

```text
Every route below this line
        |
        ▼
verifyJWT runs first
```

So:

```js
GET /projects
```

actually becomes:

```text
Request
   |
verifyJWT
   |
getProjects
```

---

## What verifyJWT does?

It:

```js
1. Reads JWT
2. Verifies JWT
3. Finds User
4. Adds:

req.user = user
```

Now every controller can use:

```js
req.user._id
```

without finding user again.

---

# Route 1

```js
router
  .route("/")
```

Represents:

```text
/projects
```

---

## GET /projects

```js
.get(getProjects)
```

Flow:

```text
GET /projects

verifyJWT
    |
    ▼
getProjects()
```

Controller:

```js
getProjects
```

returns all projects where current user is a member.

---

## POST /projects

```js
.post(
   createProjectValidator(),
   validate,
   createProject
)
```

Flow:

```text
POST /projects
      |
      ▼
createProjectValidator()
      |
      ▼
validate
      |
      ▼
createProject
```

---

### Example Request

```json
{
  "name": "Task Manager",
  "description": "Project description"
}
```

---

### Validator

Checks:

```js
name exists
description exists
```

If invalid:

```text
400 Bad Request
```

Controller never runs.

---

### Controller

```js
createProject()
```

Creates:

```text
Project
ProjectMember
```

and makes creator:

```text
ADMIN
```

---

# Route 2

```js
.route("/:projectId")
```

Represents:

```text
/projects/123
```

where:

```js
req.params.projectId
```

contains:

```text
123
```

---

# GET /projects/:projectId

```js
.get(
   validateProjectPermission(
      AvailableUserRole
   ),
   getProjectById
)
```

---

## Flow

```text
Request
   |
verifyJWT
   |
validateProjectPermission()
   |
getProjectById()
```

---

## Why?

Suppose:

```text
User A
```

tries:

```text
GET /projects/xyz
```

We first check:

```text
Is User A actually a member of project xyz?
```

If yes:

```text
Allow
```

Otherwise:

```text
403 Forbidden
```

---

## AvailableUserRole

Likely:

```js
[
  "ADMIN",
  "PROJECT_ADMIN",
  "MEMBER"
]
```

Meaning:

```text
Any project member
can view project details
```

---

# PUT /projects/:projectId

```js
.put(
   validateProjectPermission([
      UserRolesEnum.ADMIN
   ]),
   createProjectValidator,
   validate,
   updateProject
)
```

---

## Flow

```text
verifyJWT
     |
validateProjectPermission
     |
validator
     |
validate
     |
updateProject
```

---

### Permission Check

Only:

```text
ADMIN
```

allowed.

---

Example:

```text
ADMIN -> allowed
MEMBER -> denied
```

---

Controller:

```js
updateProject()
```

updates:

```js
name
description
```

---

# DELETE /projects/:projectId

```js
.delete(
   validateProjectPermission([
      UserRolesEnum.ADMIN
   ]),
   deleteProject
)
```

---

Flow:

```text
verifyJWT
    |
permission check
    |
deleteProject
```

Only admins can delete project.

---

# Route 3

```js
.route("/:projectId/members")
```

Represents:

```text
/projects/123/members
```

---

# GET Members

```js
.get(getProjectMembers)
```

Flow:

```text
verifyJWT
     |
getProjectMembers
```

Controller runs aggregation:

```text
ProjectMember
     +
Users
```

Returns:

```json
[
  {
    "user": {
      "username": "prashant"
    },
    "role": "ADMIN"
  }
]
```

---

# POST Member

```js
.post(
   validateProjectPermission([
      UserRolesEnum.ADMIN
   ]),
   addMemberToProjectValidator(),
   validate,
   addMembersToProject
)
```

---

Flow

```text
verifyJWT
      |
permission
      |
validator
      |
validate
      |
controller
```

---

Example Request

```json
{
  "email": "john@gmail.com",
  "role": "MEMBER"
}
```

Controller:

```js
addMembersToProject()
```

does:

```text
Find user by email
      |
Create ProjectMember
```

or update existing role.

---

# Route 4

```js
.route("/:projectId/members/:userId")
```

Represents:

```text
/projects/123/members/456
```

where:

```js
projectId = 123
userId = 456
```

---

# Update Role

```js
.put(
   validateProjectPermission([
      UserRolesEnum.ADMIN
   ]),
   updateMemberRole
)
```

---

Flow

```text
verifyJWT
     |
Admin Check
     |
updateMemberRole
```

---

Example

```json
{
   "newRole": "PROJECT_ADMIN"
}
```

Controller:

```text
Find member
Update role
Return updated document
```

---

# Delete Member

```js
.delete(
   validateProjectPermission([
      UserRolesEnum.ADMIN
   ]),
   deleteMember
)
```

---

Flow

```text
verifyJWT
     |
Admin Check
     |
deleteMember
```

Deletes:

```text
ProjectMember document
```

which effectively removes the user from the project.

---

# Complete Flow Diagram

```text
POST /projects

Client
  |
  ▼
verifyJWT
  |
  ▼
createProjectValidator
  |
  ▼
validate
  |
  ▼
createProject Controller
  |
  ▼
Project Model
  |
  ▼
MongoDB
  |
  ▼
Response
```

---

# Permission Flow Diagram

```text
PUT /projects/:projectId

Client
  |
verifyJWT
  |
validateProjectPermission(["ADMIN"])
  |
  ├── role = ADMIN
  |       |
  |       ▼
  |   updateProject
  |
  └── role = MEMBER
          |
          ▼
      403 Forbidden
```

The most important thing to understand from this file is that **routes don't contain business logic**. They only decide:

```text
Which URL?
     ↓
Which middleware?
     ↓
Which controller?
```

The real work happens inside controllers, while middleware handles authentication, validation, and permissions before the controller gets a chance to run.


---

in `app.js` : 



This line connects your **project routes** to your Express application.

```js
import projectRouter from "./routes/project.routes.js";

app.use("/api/v1/projects", projectRouter);
```

Think of it like this:

```text
app.js
  |
  |---- /api/v1/projects ----> projectRouter
```

---

## What happens when a request comes?

Suppose the client sends:

```http
GET /api/v1/projects
```

### Step 1

Express sees:

```js
app.use("/api/v1/projects", projectRouter);
```

and forwards the request to `projectRouter`.

```text
GET /api/v1/projects
        |
        ▼
   projectRouter
```

---

### Step 2

Inside `project.routes.js`

```js
router.route("/")
      .get(getProjects)
```

Since the base path is already:

```text
/api/v1/projects
```

and route path is:

```text
/
```

Final URL becomes:

```text
/api/v1/projects
```

So Express executes:

```js
getProjects()
```

---

## Example 2

Route:

```js
router.route("/:projectId")
      .get(getProjectById)
```

Request:

```http
GET /api/v1/projects/123
```

Flow:

```text
GET /api/v1/projects/123
            |
            ▼
app.use("/api/v1/projects", projectRouter)
            |
            ▼
router.route("/:projectId")
            |
            ▼
projectId = "123"
            |
            ▼
getProjectById()
```

Inside controller:

```js
req.params.projectId
```

will contain:

```js
"123"
```

---

## Example 3

Route:

```js
router.route("/:projectId/members")
      .get(getProjectMembers)
```

Request:

```http
GET /api/v1/projects/123/members
```

Flow:

```text
/api/v1/projects/123/members
            |
            ▼
projectRouter
            |
            ▼
"/:projectId/members"
            |
            ▼
projectId = 123
            |
            ▼
getProjectMembers()
```

---

## Complete Picture

```text
index.js
   |
   ▼
 app.js
   |
   ▼
app.use("/api/v1/projects", projectRouter)
   |
   ▼
project.routes.js
   |
   ├── GET    /               -> getProjects
   ├── POST   /               -> createProject
   ├── GET    /:projectId     -> getProjectById
   ├── PUT    /:projectId     -> updateProject
   ├── DELETE /:projectId     -> deleteProject
   ├── GET    /:projectId/members
   ├── POST   /:projectId/members
   ├── PUT    /:projectId/members/:userId
   └── DELETE /:projectId/members/:userId
```

So the **full URL** is formed by:

```text
app.use("/api/v1/projects", projectRouter)
               +
router.route(...)
```

For example:

```text
"/api/v1/projects"
+
"/:projectId/members"

=
"/api/v1/projects/:projectId/members"
```

This is why in large projects, `app.js` stays clean and each feature (auth, projects, tasks, users) manages its own routes in separate router files.
