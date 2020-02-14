Draft after incorporating Faith's comments.

# What is an entitlement?
Entitlement refers to the rights given to a user on licensing and usage. OSIsoft's Product and Support need to be able to create and impose an entitlement scheme to  grant or revoke rights, resolve issues and manage customers' use of the software. 

Using the APIs below, Product can create, update and delete entitlements or get information on existing entitlements. The APIs are also used to assign entitlements to tenants/accounts, modify entitlement specifications for a particular tenant/account or to manually enforce soft limits on account/tenants. 

Note that once an entitlement is created, its default value is automatically assigned to all tenants within. Be extra careful when entitlements are being created for tenants that already exist so as not to accidentally impose limits.

## Fields in entitlement storage
|Property|Type  |
|--|--|
|Id  |String  |
|Entitlement type  |Enum (Feature, Resource, Usage)  |
|Limit type  |Enum (Hard, Soft)  |
|Default Value  |Int or Boolean  |

## Components of entitlement storage
|Property| Type | Definition  |Examples  |Note  |
|--|--|--|--|--|
|Feature  | Boolean | Add-on functionality that can be turned on or off per account | Region support  |  |
|Resource  | Int  | Restrictions on the number of resources | Number of namespace or streams |  |
|Usage  | Int | Restrictions on resource usage | Throughput, GB egressed |None defined to date  |

## Entitlement scope
1. Account/Tenant: resources and features are managed at the tenant level, for example, region, namespaces, and users.
1. Namespace: resources and features are managed at the namespace level, for example, streams and data views.

## Types of limits on entitlement
1. Soft limit
- Applies to namespace-scoped entitlement
- Resource and usage
- Example: count of streams
- Manually reject attempts
2. Hard limit
- Applies to account-scoped entitlement
- Features, resource and usage
- Example: regions, count of namespace/users
- Automatically reject attempts to create more resources than the limit

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
A new entitlement table is created in the System Service database to store entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|Id  |Nvarchar(128)  |Primary Key  |No  |
|EntitlementType  |Int  |  |No  |
|LimitType  |Int  |  |No  |
|DefaultValue  |Int  |  |No  |

##Tenant entitlement table
A tenant entitlement table will be created in the database to store tenant entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|TenantId  |Nvarchar(128)  |Foreign key to Tenant.Id  |No  |
|EntitlementId  |Nvarchar(128)  |Foreign key to Entitlement.Id |No  |
|Value  |Int  |  |No  |

# Development details 
Entitlement CRUD operations will be defined in the EntitlementController. The TenantEntitlement CRUD operations will be exposed in the TenantEntitlementController. When you delete an entitlement, all tenant entitlements that reference the entitlement will also be deleted. 

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
      "entitlementType":"Feature",
      "limitType":"Hard"
   },
   { 
      "id":"NamespaceCount",
      "defaultValue":5,
      "entitlementType":"Resource",
      "limitType":"Hard"
   },
   { 
      "id":"StreamCount",
      "defaultValue":10000,
      "entitlementType":"Resource",
      "limitType":"Soft"
   },
   { 
      "id":"EgressRate",
      "defaultValue":200,
      "entitlementType":"Usage",
      "limitType":"Soft"
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
  "entitlementType":"Resource",
  "limitType":"Soft"
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
  "entitlementType":"Resource",
  "limitType":"Soft"
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
  "entitlementType":"Resource",
  "limitType":"Soft"
}
```

## Update entitlement
Updates the definition of entitlement. Update does not propogate to the entitlement that is already assigned to tenants. 

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
  "entitlementType":"Resource",
  "limitType":"Soft"
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
  "entitlementType":"Resource",
  "limitType":"Soft"
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
When a tenant is created, the system will create tenant entitlements with the default entitlement values. This design allows for modifications to the default values of entitlements without changing the values already assigned to tenants. However, if there are M tenants and N entitlements, this will take up M * N rows in the database, similar to how roles are currently handled in the system database. 

# Tenant entitlements API
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

 
====
Draft shared with Faith

[[_TOC_]]

# What is an entitlement?
Entitlement is the rights given to a user on licensing and usage. As a company, Product and Support need to be able to create and impose entitlement scheme to  grant or revoke rights, resolve issues and manage customer's use of the software. 

Using the APIs below, Product can create, update and delete entitlements or get information on existing entitlements. The APIs are also used for assigning entitlements to tenants/accounts, modifying entitlement specifications for a particular tenant/account or to manually enforce soft limits for account/tenants. 

Note that once an entitlement is created, its default value is automatically assigned to all tenants within. Be extra careful when entitlements are being created for tenants that already exist so as not to accidentally impose limits.

## Fields in entitlement storage
|Property|Type  |
|--|--|
|Id  |String  |
|Entitlement type  |Enum (Feature, Resource, Usage)  |
|Limit type  |Enum (Hard, Soft)  |
|Default Value  |Int or Boolean  |

## Components of entitlement storage
|Property| Type | Definition  |Examples  |Note  |
|--|--|--|--|--|
|Feature  | Boolean | Add-on functionality that can be turned on or off per account | Region support  |  |
|Resource  | Int  | Restrictions on the number of resources | Number of namespace or streams |  |
|Usage  | Int | Restrictions on resource usage | Throughput, GB egressed |None defined to date  |

## Entitlement scope
1. Account/Tenant: resources and features are managed at the tenant level, for example, region, namespaces, and users.
1. Namespace: resources and features are managed at the namespace level, for example, streams and data views.

## Types of limits on entitlement
1. Soft limit
- Applies to namespace-scoped entitlement
- Resource and usage
- Example: count of streams
- Manually reject attempts
2. Hard limit
- Applies to account-scoped entitlement
- Features, resource and usage
- Example: regions, count of namespace/users
- Automatically reject attempts to create more resources than the limit

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
A new entitlement table is created in the System Service database to store entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|Id  |Nvarchar(128)  |Primary Key  |No  |
|EntitlementType  |Int  |  |No  |
|LimitType  |Int  |  |No  |
|DefaultValue  |Int  |  |No  |

##Tenant entitlement table
A tenant entitlement table will be created in the database to store tenant entitlement definition.

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|TenantId  |Nvarchar(128)  |Foreign key to Tenant.Id  |No  |
|EntitlementId  |Nvarchar(128)  |Foreign key to Entitlement.Id |No  |
|Value  |Int  |  |No  |

# Development details 
Entitlement CRUD operations will be defined in the EntitlementController. The TenantEntitlement CRUD operations will be exposed in the TenantEntitlementController. When deleting an entitlement, the system will cascade delete to all tenant entitlements that reference the entitlement. 

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
The response includes a status code and entitlement definition in a body

#### Example response body
```json
[ 
   { 
      "id":"WestUS",
      "defaultValue":true,
      "entitlementType":"Feature",
      "limitType":"Hard"
   },
   { 
      "id":"NamespaceCount",
      "defaultValue":5,
      "entitlementType":"Resource",
      "limitType":"Hard"
   },
   { 
      "id":"StreamCount",
      "defaultValue":10000,
      "entitlementType":"Resource",
      "limitType":"Soft"
   },
   { 
      "id":"EgressRate",
      "defaultValue":200,
      "entitlementType":"Usage",
      "limitType":"Soft"
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
The response includes a status code and entitlement definition in a body

#### Example response body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":"Resource",
  "limitType":"Soft"
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
  "entitlementType":"Resource",
  "limitType":"Soft"
}
```

### Authorization
Cluster operator, service and admin

### Response
The response includes a status code and entitlement definition in a body

#### Example response body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":"Resource",
  "limitType":"Soft"
}
```

## Update entitlement
Updates an entitlement. When an entitlement is updated, it will not update any entitlements already given to tenants but only the definition of the entitlement. 

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
  "entitlementType":"Resource",
  "limitType":"Soft"
}
```
### Authorization
Cluster operator, service and admin

### Response
The response includes a status code and entitlement definition in a body

#### Example response body
```json
{ 
  "id":"StreamCount",
  "defaultValue":10000,
  "entitlementType":"Resource",
  "limitType":"Soft"
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
The response includes a status code and if there's an error, a response body

# Tenant entitlements
## Tenant entitlements creation 
When a tenant is created, the system will create tenant entitlements with the default entitlement values. This design allows for modifications to the default values of entitlements without changing the values already assigned to tenants. However, if there are M tenants and N entitlements, this will take up M * N rows in the database, similar to how roles are currently handled in the system database. 

# Tenant entitlements API
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
Dictionary of entitlements with ``entitlementId`` as key and ``entitlement value`` as value, where each key/value pair represents a tenantEntitlement

### Authorization
Cluster operator, service, admin and tenant members

### Response
The response includes a status code and a response body containing a serialized event

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
The response includes a status code and a response body containing a serialized event

#### Example response body
```json
{ 
   "NamespaceCount":10,
   "WestEU":true,
   "WestUS":true
} 
```
### Note
The request body accepts both Boolean and Int in the values. All values will be mapped to an Int before it is stored in the database.

 

``string** **``
`string*space*`
Some Markdown text with <span style="color:blue">some *blue* text</span>.

|heading| description|
--------|----------
`device!!`*space* | space after device!! 

Management > Namespace
Think of namespace as a logical database where streams and other objects are stored by account.

Management > Tenants
(Account-level should be hyphenated)

Sequential Data Store >
The Sequential Data Store (SDS) is a cloud-based streaming data storage that is optimized for storing sequential data, usually a time-series, but anything that is indexed by an ordered sequence. You use SDS to store, retrieve, and analyze data. 

An SdsType (used interchangeably with type throughout documentation) defines the shape of a single measured event or object. A type gives structure to your data. For example, if you're measuring three things (longitute, latitude, speed) from a device at the same time, then you want those three properties to be included in your tytpe. An SdsStream (used interchangeably with stream throughout documentation) is an instance of the type you have defined. Each stream is made up of a series of instances (or events). 


===

Types are equivalent to int32, float32, string. Can be defined in any way you like. Things that are measured at the same time (index), at the same place on a same device. In a sensor in an application, you want to give it a structure. You define a type with the way the data should be represented in mind. Do it for the application reason, not for the storage persistence reason. If the sensor is measuring three things at the same time, you should store those in the same type. 

Streams are instance of the type. Each stream will have a series of events. 

Namespace a container for types and streams. 

 [Indexes](xref:sdsIndexes)


Some Markdown text with <span style="color:blue">some *blue* text</span>.

# markdownshowdown
## This would be the second heading
### Am I the third heading?
#### Me fourth?
Hello world
===========
Hello world mini
-----------------
Emphasis in *italitcs*  
Emphasis in **bold**  
Combined in **_bold italics_**  
Strikethrough is ~~Scratch this~~  

<table>
  <tr>
    <td>Hello</td>
  </tr>
  <table>
    
>Block Quotes
>> Nested block quotes

unblocked
quotes

* for list items, use asterisk with space
* list
* list

- same with a hyphen for a list
- list
- list

+ or a plus sign
+ another plus sign for a list

1. numbers mark a ordered list
1. any number will do
1. numbers are awesome

* A list with a code block:
    Code block with two tabs or eight spaces
