## Role
#### Create 
```json
before
//req 
// post -> /api/Role/AddRole
{
  "roleName": "string",
  "permissions": [
    {
      "id": 0,
      "name": "string"
    }
  ]
}
//res (200)
{
  "succeeded": true,
  "errors": []
} 

after 
//req
// post -> /api/role
{
  "nameAr": null,
  "nameEn": null,
  "permissions": [
    {
      "name": null
    }
  ]
}
//res (200)
"01973a94-cc12-7ea3-859c-3f11de06dbca" //roleID

```

#### Get all

```json

before: 
//req
// get -> /api/Role/GetRoles
//res
[
 {
    "id": 1,
    "roleName": "RolesPermisions",
    "isActive": true,
    "createdOn": "2024-06-29T19:14:35.2033333",
    "createdBy": "system",
    "permissions": [
	    {
		    "id": "",
		    "name": ""
	    }
    ]
  },
]

after // supports filteration & pagination
//req
//get -> /api/role
//res
{
  "items": [
    {
      "id": "01973601-4479-7632-8d2a-c827ed931cab",
      "nameAr": null,
      "nameEn": null,
      "isActive": false,
      "createdOn": "2025-06-03T13:35:42.3596946",
      "createdBy": "System",
      "permissions": [
	    {
		    "name": ""
	    }
      ]
    }
  ],
  "page": 1,
  "pageSize": 10,
  "totalCount": 3,
  "totalPages": 1
}


```
#### Update
```json
before 
//req
// post -> /api/Role/UpdateRole
{
  "roleId": 1004,
  "roleName": "updated role name",
  "permissions": [
    {
      "id": 0,
      "name": "string"
    }
  ]
}
//res 200
{
  "succeeded": true,
  "errors": []
}

after (*http verb: put)
//req
// put -> /api/role
{
  "roleId": "01973a94-cc12-7ea3-859c-3f11de06dbca",
  "nameAr": "تم التغيير",
  "nameEn": "updated",
  "permissions": [
    {
      "name": "test permission 2"
    }
  ]
}
//res 
204 No Content
```

#### Delete
```json
before 
//req
//Delete /api/Role/DeleteRole?RoleId=1004'
//res (200)
{
  "succeeded": true,
  "errors": []
}

after (http ver: Delete)
//req 
//Delete /api/Role/{roleId}
//res
204 No Content
```

#### Get by id
```json
before
//req
// get /api/Role/GetRoleById?RoleId=1004
//res
{
  "id": 1,
  "roleName": "RolesPermisions",
  "isActive": true,
  "createdOn": "2024-06-29T19:14:35.2033333",
  "createdBy": "system",
  "permissions": [
	    {
		    "id": "",
		    "name": ""
	    }
  ]
}
after
//req
// get /api/Role/{roleId}
//res
{
  "id": "01973a69-dae8-70e3-ba20-737644e0f801",
  "nameAr": "بلا بلا",
  "nameEn": "ٌtest role",
  "isActive": false,
  "createdOn": "2025-06-04T10:08:25.3585425",
  "createdBy": "System",
  "permissions": [
	    {
		    "name": ""
	    }
  ]
}
```
----
## User
#### Get user roles
```json
befor
//req Get /api/User/UserRoles?UserId=1' 
//res
[
  {
    "id": 1,
    "roleName": "RolesPermisions",
    "isActive": true,
    "createdOn": "2024-06-29T19:14:35.2033333",
    "createdBy": "system",
    "permissions": []
  }
]
after
//req Get /api/User/UserRoles?UserId=""'
//res
[
  {
    "id": "01973ad9-456c-7ca8-aec9-c203f302c88c",
    "nameAr": "مطور",
    "nameEn": "developer",
    "isActive": false,
    "createdOn": "2025-06-04T12:10:07.12295",
    "createdBy": "System",
    "permissions": [
      {
        "name": "p-1"
      },
      {
        "name": "p-2"
      }
    ]
  }
```

#### Add user to a role
```json
before
//req Post /api/User/AddToRole
{
  "userId": 0,
  "roleName": "string"
}
//res 
{
  "succeeded": true,
  "errors": [
    "string"
  ]
}
after *add to role/roles*
//req Post /api/User/AddToRoles
{
  "userId": "01973AD0-649C-74DF-87C0-0F75EAE9F2A6",
  "roles": [
    "developer"
  ]
}
//res 
204 No Content
```


#### remove from role
````json
before
//req Delete /api/User/RemoveFromRole
{
  "userId": 0,
  "roleName": "string"
}
//res
{
  "succeeded": true,
  "errors": [
    "string"
  ]
}
after *remove from role/roles*
//req
{
  "userId": "01973AD0-649C-74DF-87C0-0F75EAE9F2A6",
  "roles": [
    "developer"
  ]
//res
204 No Content
````


## Account
#### Register
```json
before: 
//req 
//res
{
  "succeeded": true,
  "errors": []
}
after: 
//req  //no changes
//res
201 Created
```

#### Login
```json
before:
//req
{
  "username": "string",
  "password": "string"
}
//res
{
  "isAuthenticated": true,
  "token": "",
  "expiresOn": null
}
after:
//req
{
  "userIdentifier": "",
  "password": "",
  "rememberMe": true
}
//res
{
  "succeeded": true,
  "token": "",
  "userId": "",
  "email": "",
  "firstName": "",
  "lastName": "",
  "roles": []
}
```
#### Change-Password
```json
before:
//req
//res
{
  "succeeded": true,
  "errors": [
    "string"
  ]
}
after:
//req (no changes)
//res
204 No Content
```