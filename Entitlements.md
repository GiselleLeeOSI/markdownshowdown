[[_TOC_]]

# What is an entitlement?
Entitlement refers to the rights given to a user on licensing and usage. OSIsoft's Product and Support need to be able to create and impose an entitlement scheme to  grant or revoke rights, resolve issues and manage customers' use of the software. 

Using the APIs below, Product can create, update and delete entitlements or get information on existing entitlements. The APIs are also used to assign entitlements to tenants/accounts, modify entitlement specifications for a particular tenant/account or to manually enforce soft limits on tenants/accounts. 

Note that once an entitlement is created, its default values are automatically assigned to all tenants within. Be extra careful when entitlements are being created for tenants that already exist so as not to accidentally impose limits.

- Fields in entitlement storage

|Property|Type  |
|--|--|
|Id  |String  |
|Entitlement type  |Enum (Feature=0; Resource=1; Usage=2)  |
|Limit type  |Enum (Hard=0; Soft=1)  |
|Default Value  |Int or Boolean  |

- Components of entitlement storage

|Property| Type | Definition  |Examples  |Note  |
|--|--|--|--|--|
|Feature  | Boolean | Add-on functionality that can be turned on or off per account | Region support  |  |
|Resource  | Int  | Restrictions on the number of resources | Number of namespace or streams |  |
|Usage  | Int | Restrictions on resource usage | Throughput, GB egressed |None defined to date  |

- Entitlement scope
1. Tenant/Account: resources and features are managed at the tenant level, for example, region, namespaces, and users.
1. Namespace: resources and features are managed at the namespace level, for example, streams and data views.


 Types of limits on entitlement
1. Hard limit
- Applies to account-scoped entitlement
- Features, resource and usage
- Example: regions, count of namespace/users
- Automatically reject attempts to create more resources than the limit
2. Soft limit
- Applies to namespace-scoped entitlement
- Resource and usage
- Example: count of streams
- Manually reject attempts

Allowed entitlement CRUD operations by cluster role

Product managers and support engineers are responsible for creating and updating entitlements. 

|Roles\Operation|Create  |Read  |Update  |Delete  |
|--|--|--|--|--|
|Admin  | Y | Y | Y | Y |
|Operator  | Y | Y | Y |  |
|Service  |  Y| Y | Y | Y |
|Support  |  |  Y|  |  |

Design details

Entitlement table

A new entitlement table is created in the System Service database (SystemStorage) to store entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|Id  |Nvarchar(128)  |Primary Key  |No  |
|EntitlementType  |Int  |  |No  |
|LimitType  |Int  |  |No  |
|DefaultValue  |Int  |  |No  |

Tenant entitlement table

A tenant entitlement table will be created in the database to store tenant entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|TenantId  |Nvarchar(128)  |Foreign key to Tenant.Id  |No  |
|EntitlementId  |Nvarchar(128)  |Foreign key to Entitlement.Id |No  |
|Value  |Int  |  |No  |

Development details
 
Entitlement CRUD operations will be defined in the EntitlementController.
The TenantEntitlement CRUD operations will be exposed in the TenantEntitlementController. 
When you delete an entitlement, all tenant entitlements that reference the entitlement will also be deleted. 

# Entitlement APIs
## Get entitlements
Returns a collection of entitlements. 

**Request**
```text
GET api/entitlements 
```
**Parameter**

None

**Authorization**

Support, cluster service, operator and admin

**Response**

A status code and entitlement definition in a body

**Example response body**
```json
[ 
   { 
      "id":"WestUS",
      "defaultValue":true,
      "entitlementType":0,
      "limitType":0
   },
   { 
      "id":"NamespaceCount",
      "defaultValue":5,
      "entitlementType":1,
      "limitType":0
   },
   { 
      "id":"StreamCount",
      "defaultValue":10000,
      "entitlementType":1,
      "limitType":1
   },
   { 
      "id":"EgressRate",
      "defaultValue":200,
      "entitlementType":2,
      "limitType":1
   }
]
```
## Get entitlement
Returns an entitlement. 

**Request**
```text
GET api/entitlements/{entitlementId}
```
**Parameter**

`string entitlementId`

The entitlement identifier

**Authorization**

Support, cluster service, operator and admin

**Response**

A status code and entitlement definition in a body

**Example response body**
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":1,
  "limitType":1
}
```

## Create entitlement
Creates an entitlement.

**Request**
```text
POST api/entitlements/{entitlementId}
```
**Parameter**

`string entitlementId`

The entitlement identifier

**Example request body**
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":1,
  "limitType":1
}
```

**Authorization**

Cluster operator, service and admin

**Response**

A status code and entitlement definition in a body

**Example response body**
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":1,
  "limitType":1
}
```

## Update entitlement
Updates the definition of entitlement. Update does not propagate to the entitlement that is already assigned to tenants.
You can only update the default value of the entitlement but not other fields. 

**Request**
```text
PUT api/entitlements/{entitlementId}
```
**Parameter**

`string entitlementId`

The entitlement identifier

**Example request body**
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":0,
  "limitType":1
}
```
**Authorization**

Cluster operator, service and admin

**Response**

A status code and entitlement definition in a body

**Example response body**
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":0,
  "limitType":1
}
```

## Delete entitlement
Deletes an entitlement. Removes the entitlement from all tenants. 

**Request**

```text
DELETE api/entitlements/{entitlementId}
```
**Parameter**

`string entitlementId`

The entitlement identifier

**Authorization**

Cluster service and admin

**Response**

A status code and if there's an error, a response body 

# Tenant entitlements
## Tenant entitlements creation 
When a tenant is created, the system will create tenant entitlements with the default entitlement values that are already defined.
When you update the default values in an entitlement, it will not affect the ones already assigned to tenants.
However, if there are M tenants and N entitlements, this will take up M * N rows in the database.
This is similar to how roles are currently handled in the system storage database where entitlement schema is implemented. 

# Tenant entitlements APIs
## Get tenant entitlements
Returns tenant entitlements.  

**Request**
```text
GET api/tenants/{tenantId}/entitlements
```
**Parameter**

`string tenantId`

The tenant identifier

**Example request body**

Dictionary of entitlements with ``entitlementId`` as key and ``entitlement value`` as value, where each key/value pair represents a tenant entitlement

**Authorization**

Cluster operator, service, admin and tenant members

**Response**

A status code and a response body containing a serialized event

**Example response body**
```json
{ 
   "NamespaceCount":10,
   "WestEU":true,
   "WestUS":true
} 
```

**Note**

The request body accepts both Boolean and Int as values. All values will be mapped to an Int before it is stored in the database.

## Update tenant entitlements
Updates tenant entitlements.  

**Request**
```text
PUT api/tenants/{tenantId}/entitlements
```
**Parameter**

`string tenantId`

The tenant identifier

**Request body**

Dictionary of entitlements with ``entitlementId`` as key and ``entitlement value`` as value, where each key/value pair represents a tenantEntitlement

**Example response body**
```json
{ 
   "NamespaceCount":10,
   "WestEU":true,
   "WestUS":true
} 
```

**Authorization**

Cluster operator, service and admin

**Response**

A status code and a response body containing a serialized event

**Example response body**
```json
{ 
   "NamespaceCount":10,
   "WestEU":true,
   "WestUS":true
} 
```
**Note**

The request body accepts both Boolean and Int in the values. All values will be mapped to an Int before they are stored in the database.

 

[[_TOC_]]

# What is an entitlement?
Entitlement refers to the rights given to a user on licensing and usage. OSIsoft's Product and Support need to be able to create and impose an entitlement scheme to  grant or revoke rights, resolve issues and manage customers' use of the software. 

Using the APIs below, Product can create, update and delete entitlements or get information on existing entitlements. The APIs are also used to assign entitlements to tenants/accounts, modify entitlement specifications for a particular tenant/account or to manually enforce soft limits on tenants/accounts. 

Note that once an entitlement is created, its default values are automatically assigned to all tenants within. Be extra careful when entitlements are being created for tenants that already exist so as not to accidentally impose limits.

## Fields in entitlement storage
|Property|Type  |
|--|--|
|Id  |String  |
|Entitlement type  |Enum (Feature=0; Resource=1; Usage=2)  |
|Limit type  |Enum (Hard=0; Soft=1)  |
|Default Value  |Int or Boolean  |

## Components of entitlement storage
|Property| Type | Definition  |Examples  |Note  |
|--|--|--|--|--|
|Feature  | Boolean | Add-on functionality that can be turned on or off per account | Region support  |  |
|Resource  | Int  | Restrictions on the number of resources | Number of namespace or streams |  |
|Usage  | Int | Restrictions on resource usage | Throughput, GB egressed |None defined to date  |

## Entitlement scope
1. Tenant/Account: resources and features are managed at the tenant level, for example, region, namespaces, and users.
1. Namespace: resources and features are managed at the namespace level, for example, streams and data views.

## Types of limits on entitlement
1. Hard limit
- Applies to account-scoped entitlement
- Features, resource and usage
- Example: regions, count of namespace/users
- Automatically reject attempts to create more resources than the limit
2. Soft limit
- Applies to namespace-scoped entitlement
- Resource and usage
- Example: count of streams
- Manually reject attempts

## Allowed entitlement CRUD operations by cluster role
Product managers and support engineers are responsible for creating and updating entitlements. 

|Roles\Operation|Create  |Read  |Update  |Delete  |
|--|--|--|--|--|
|Admin  | Y | Y | Y | Y |
|Operator  | Y | Y | Y |  |
|Service  |  Y| Y | Y | Y |
|Support  |  |  Y|  |  |

# Design details
## Entitlement table
A new entitlement table is created in the System Service database (SystemStorage) to store entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|Id  |Nvarchar(128)  |Primary Key  |No  |
|EntitlementType  |Int  |  |No  |
|LimitType  |Int  |  |No  |
|DefaultValue  |Int  |  |No  |

## Tenant entitlement table
A tenant entitlement table will be created in the database to store tenant entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|TenantId  |Nvarchar(128)  |Foreign key to Tenant.Id  |No  |
|EntitlementId  |Nvarchar(128)  |Foreign key to Entitlement.Id |No  |
|Value  |Int  |  |No  |

# Development details 
Entitlement CRUD operations will be defined in the EntitlementController.
The TenantEntitlement CRUD operations will be exposed in the TenantEntitlementController. 
When you delete an entitlement, all tenant entitlements that reference the entitlement will also be deleted. 

# Entitlement APIs
## Get entitlements
Returns a collection of entitlements. 

### Request
```text
GET api/entitlements 
```
#### Parameter
None

### Authorization
Support, cluster service, operator and admin

### Response
A status code and entitlement definition in a body

#### Example response body
```json
[ 
   { 
      "id":"WestUS",
      "defaultValue":true,
      "entitlementType":0,
      "limitType":0
   },
   { 
      "id":"NamespaceCount",
      "defaultValue":5,
      "entitlementType":1,
      "limitType":0
   },
   { 
      "id":"StreamCount",
      "defaultValue":10000,
      "entitlementType":1,
      "limitType":1
   },
   { 
      "id":"EgressRate",
      "defaultValue":200,
      "entitlementType":2,
      "limitType":1
   }
]
```
## Get entitlement
Returns an entitlement. 

### Request
```text
GET api/entitlements/{entitlementId}
```
#### Parameter
`string entitlementId`

The entitlement identifier

### Authorization
Support, cluster service, operator and admin

### Response
A status code and entitlement definition in a body

#### Example response body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":1,
  "limitType":1
}
```

## Create entitlement
Creates an entitlement.
### Request
```text
POST api/entitlements/{entitlementId}
```
#### Parameter
`string entitlementId`

The entitlement identifier

#### Request body
#### Example request body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":1,
  "limitType":1
}
```

### Authorization
Cluster operator, service and admin

### Response
A status code and entitlement definition in a body

#### Example response body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":1,
  "limitType":1
}
```

## Update entitlement
Updates the definition of entitlement. Update does not propagate to the entitlement that is already assigned to tenants.
You can only update the default value of the entitlement but not other fields. 

### Request
```text
PUT api/entitlements/{entitlementId}
```
#### Parameter
`string entitlementId`

The entitlement identifier

#### Example request body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":0,
  "limitType":1
}
```
### Authorization
Cluster operator, service and admin

### Response
A status code and entitlement definition in a body

#### Example response body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":0,
  "limitType":1
}
```

## Delete entitlement
Deletes an entitlement. Removes the entitlement from all tenants. 

### Request
```text
DELETE api/entitlements/{entitlementId}
```
#### Parameter
`string entitlementId`

The entitlement identifier

### Authorization
Cluster service and admin

### Response
A status code and if there's an error, a response body 

# Tenant entitlements
## Tenant entitlements creation 
When a tenant is created, the system will create tenant entitlements with the default entitlement values that are already defined.
When you update the default values in an entitlement, it will not affect the ones already assigned to tenants.
However, if there are M tenants and N entitlements, this will take up M * N rows in the database.
This is similar to how roles are currently handled in the system storage database where entitlement schema is implemented. 

# Tenant entitlements APIs
## Get tenant entitlements
Returns tenant entitlements.  

### Request
```text
GET api/tenants/{tenantId}/entitlements
```
#### Parameter
`string tenantId`

The tenant identifier

#### Request body
Dictionary of entitlements with ``entitlementId`` as key and ``entitlement value`` as value, where each key/value pair represents a tenant entitlement

### Authorization
Cluster operator, service, admin and tenant members

### Response
A status code and a response body containing a serialized event

#### Example response body
```json
{ 
   "NamespaceCount":10,
   "WestEU":true,
   "WestUS":true
} 
```

### Note
The request body accepts both Boolean and Int as values. All values will be mapped to an Int before it is stored in the database.

## Update tenant entitlements
Updates tenant entitlements.  

### Request
```text
PUT api/tenants/{tenantId}/entitlements
```
#### Parameter
`string tenantId`

The tenant identifier

#### Request body
Dictionary of entitlements with ``entitlementId`` as key and ``entitlement value`` as value, where each key/value pair represents a tenantEntitlement

#### Example response body
```json
{ 
   "NamespaceCount":10,
   "WestEU":true,
   "WestUS":true
} 
```

### Authorization
Cluster operator, service and admin

### Response
A status code and a response body containing a serialized event

#### Example response body
```json
{ 
   "NamespaceCount":10,
   "WestEU":true,
   "WestUS":true
} 
```
### Note
The request body accepts both Boolean and Int in the values. All values will be mapped to an Int before they are stored in the database.

 
