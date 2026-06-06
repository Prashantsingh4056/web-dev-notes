This is a very good set of controllers because it combines:

* Basic CRUD operations
* Relationships between collections
* Aggregation pipelines (`$lookup`, `$match`, `$project`, `$unwind`)
* Role management

Let's go controller by controller.

---

# 1. getProjects()

### Goal

Return all projects in which the currently logged-in user is a member.

---

## Step 1: Start from ProjectMember collection

Suppose ProjectMember collection contains:

```js
{
  user: ObjectId("u1"),
  project: ObjectId("p1"),
  role: "ADMIN"
}

{
  user: ObjectId("u1"),
  project: ObjectId("p2"),
  role: "MEMBER"
}
```

This means user `u1` belongs to two projects.

---

## Step 2: Match current user

```js
{
  $match: {
    user: new mongoose.Types.ObjectId(req.user._id)
  }
}
```

Equivalent SQL:

```sql
WHERE user = currentUserId
```

After this stage:

```js
[
  {
    user: u1,
    project: p1,
    role: "ADMIN"
  },
  {
    user: u1,
    project: p2,
    role: "MEMBER"
  }
]
```

---

## Step 3: Join Projects collection

```js
{
  $lookup: {
    from: "projects",
    localField: "project",
    foreignField: "_id",
    as: "project"
  }
}
```

Think of:

```sql
JOIN projects
ON projectmembers.project = projects._id
```

Result:

```js
{
  role: "ADMIN",
  project: [
    {
      _id: p1,
      name: "Task Manager"
    }
  ]
}
```

Notice:

```js
project: []
```

is always an array.

---

## Step 4: Inside lookup → count members

Another lookup:

```js
$lookup
```

fetches all members of that project.

Example:

```js
projectmembers: [
  {...},
  {...},
  {...}
]
```

Then:

```js
$addFields: {
  members: {
    $size: "$projectmembers"
  }
}
```

`$size` = length of array

Result:

```js
members: 3
```

---

## Step 5: Unwind

```js
{
  $unwind: "$project"
}
```

Converts:

```js
project: [
  {
    name: "Task Manager"
  }
]
```

into

```js
project: {
  name: "Task Manager"
}
```

---

## Step 6: Project

```js
{
  $project: {
    project: {
      _id: 1,
      name: 1,
      description: 1
    },
    role: 1,
    _id: 0
  }
}
```

Controls what fields are returned.

Equivalent to:

```js
Select only these columns
```

---

## Final Output

```js
[
  {
    role: "ADMIN",
    project: {
      name: "Task Manager",
      members: 5
    }
  }
]
```

---

# 2. getProjectById()

### Goal

Fetch one project.

---

```js
const project = await Project.findById(projectId);
```

Equivalent SQL:

```sql
SELECT *
FROM projects
WHERE id = projectId
```

---

If not found:

```js
throw new ApiError(404)
```

Otherwise:

```js
return project
```

---

# 3. createProject()

### Goal

Create project and automatically make creator admin.

---

## Step 1

Receive:

```js
{
  name,
  description
}
```

---

## Step 2

Create project

```js
const project = await Project.create(...)
```

Example:

```js
{
  _id: p1,
  name: "Task Manager",
  description: "...",
  createdBy: u1
}
```

---

## Step 3

Create ProjectMember document

```js
await ProjectMember.create(...)
```

Creates:

```js
{
  user: u1,
  project: p1,
  role: "ADMIN"
}
```

Why?

Because creator should automatically become admin.

---

## Step 4

Return created project.

---

# 4. updateProject()

### Goal

Update project information.

---

Input:

```js
{
  name,
  description
}
```

and

```js
projectId
```

---

```js
Project.findByIdAndUpdate(
  projectId,
  {
    name,
    description
  },
  {
    new: true
  }
)
```

---

Without:

```js
new: true
```

Mongo returns old document.

With:

```js
new: true
```

Mongo returns updated document.

---

Example

Before:

```js
{
  name: "Old"
}
```

After update:

```js
{
  name: "New"
}
```

Returned:

```js
{
  name: "New"
}
```

---

# 5. deleteProject()

### Goal

Delete project.

---

```js
Project.findByIdAndDelete(projectId)
```

Equivalent SQL:

```sql
DELETE FROM projects
WHERE id = projectId
```

---

Returns deleted document.

If not found:

```js
404
```

---

# 6. addMembersToProject()

### Goal

Add a user to project.

---

## Step 1

Receive:

```js
{
  email,
  role
}
```

---

## Step 2

Find user

```js
User.findOne({ email })
```

---

Suppose found:

```js
{
  _id: u2
}
```

---

## Step 3

Upsert ProjectMember

```js
ProjectMember.findOneAndUpdate(...)
```

(Your code says `findByIdAndUpdate`, but logically this should be `findOneAndUpdate`.)

---

Search:

```js
{
  user: u2,
  project: p1
}
```

---

If found:

Update role.

If not found:

Create new document.

Because:

```js
upsert: true
```

means

> Update if exists, otherwise insert.

---

Result:

```js
{
  user: u2,
  project: p1,
  role: "MEMBER"
}
```

---

# 7. getProjectMembers()

### Goal

Get all members of a project.

---

## Step 1

Find all ProjectMember docs

```js
$match
```

```js
project: projectId
```

---

Result:

```js
[
  {
    user: u1,
    role: "ADMIN"
  },
  {
    user: u2,
    role: "MEMBER"
  }
]
```

---

## Step 2

Lookup users

```js
$lookup
```

Join with users collection.

---

Equivalent SQL:

```sql
JOIN users
ON projectmembers.user = users.id
```

---

## Step 3

Project only required fields

```js
{
  username: 1,
  fullname: 1,
  avatar: 1
}
```

Avoids exposing:

```js
password
refreshToken
emailVerificationToken
```

etc.

---

## Step 4

Extract first element

Lookup always returns array.

```js
user: [
  {...}
]
```

Convert:

```js
{
  user: {
    ...
  }
}
```

using:

```js
$arrayElemAt
```

---

## Step 5

Final projection

Return:

```js
{
  user,
  role,
  createdAt,
  updatedAt
}
```

---

# 8. updateMemberRole()

### Goal

Change role of existing member.

---

Example:

```js
MEMBER → ADMIN
```

---

## Step 1

Validate role

```js
if(!AvailableUserRole.includes(newRole))
```

Reject invalid roles.

---

## Step 2

Find member

```js
ProjectMember.findOne(...)
```

Search by:

```js
projectId
userId
```

---

## Step 3

Update role

```js
findByIdAndUpdate(
  memberId,
  {
    role: newRole
  }
)
```

---

Result:

```js
{
  role: "ADMIN"
}
```

---

# 9. deleteMember()

### Goal

Remove user from project.

---

## Step 1

Find membership

```js
ProjectMember.findOne({
  project,
  user
})
```

---

Example:

```js
{
  user: u2,
  project: p1
}
```

---

## Step 2

Delete it

```js
ProjectMember.findByIdAndDelete(
  projectMember._id
)
```

---

After deletion:

Before:

```js
u2 -> p1
```

After:

```js
No relationship exists
```

User is no longer a member of that project.

---

# Big Picture

You now have a complete project-management workflow:

### Project CRUD

* `createProject()`
* `getProjectById()`
* `updateProject()`
* `deleteProject()`

### Membership Management

* `addMembersToProject()`
* `getProjectMembers()`
* `updateMemberRole()`
* `deleteMember()`

### Aggregation Concepts Used

* `$match` → filter documents
* `$lookup` → join collections
* `$unwind` → flatten arrays
* `$project` → choose fields
* `$addFields` → create new fields
* `$size` → count array elements
* `$arrayElemAt` → pick specific array item

These are the same core aggregation operators you'll repeatedly use in real-world backend projects.
