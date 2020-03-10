[[_TOC_]]
# What are entitlement sets?
Entitlement sets (sets, henceforth) are used to assign entitlements to tenants/accounts.
Sets are also used to apply its entitlements to the tenant by looking inside the set for entitlements and modify values.
Those with the right permission (cluster admin and operator) can:
- assign sets tenants/accounts
- define and modify sets before or after they are assigned to tenants/accounts
- customize entitlements that belong to a set to create a custom set
- modify entitlements/sets beyond entitlement limits  

## Allowed entitlement set CRUD operations by cluster role
Product managers are responsible for creating and updating entitlement sets. 

|Roles\Operation|Create  |Read  |Update  |Delete  |Assign|
|--|--|--|--|--|--|
|Admin  | Y | Y | Y | Y |Y|
|Operator  | Y | Y | Y | Y |Y|
|Support  |  |  Y|  |  | |

# Design details
The sets table will be stored in the system database and accessed through the system service.
They will have dependencies on entitlements and tenants table in the database. 
Sets table is created to store the information on sets. 
In addition, a joining table is created to support many-to-many relationships between sets and entitlements.

## EntitlementSets table

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|Id  |Nvarchar(128)  |Primary Key  |No  |


## AssignedSetEntitlements table

|Column| Type | Constraints  |Nullable  |
|--|--|--|--|
|TenantId  |Nvarchar(128)  |Foreign key to Sets.Id  |No  |
|EntitlementId  |Nvarchar(128)  |Foreign key to Entitlements.Id |No  |
|Value  |Int  |  |No  |

# Development details 
``TenantEntitlementSetController`` class handles ```api/Tenants/{tenantId}/bulk/entitlements/{SetId}```. 

``EntitlementSetController`` class handles ```api/EntitlementSets```. 
# Entitlement set APIs
## Get entitlement sets 
Returns entitlement sets. 

### Request
```text
GET api/EntitlementSets
```
#### Parameter
None

### Authorization
Cluster support, operator and admin

### Response
A response code and a dictionary of entitlements in a body

|Property| Type | Default  |Optional  |
|--|--|--|--|
|ID  |String  |  |No  |
|Entitlements  |Dictionary (Entitlements)   |Default values of each defined entitlements  |Yes  |


#### Example response body
```json
{
   "ID":"Medium",
   "Entitlements":{
      "NamespaceCount":5,
      "WestEU":true
   }
}
```
## Get an entitlement set 
Returns the specified entitlement set. 

### Request
```text
GET api/EntitlementSets/{SetId}
```
#### Parameter
`string SetId`

The entitlement set identifier

### Authorization
Cluster support, operator and admin

### Response
A response code and a dictionary of entitlements in a body

|Property| Type | Default  |Optional  |
|--|--|--|--|
|ID  |String  |  |No  |
|Entitlements  |Dictionary (Entitlements)   |Default values of each defined entitlements  |Yes  |


#### Example response body
```json
{
   "ID":"Medium",
   "Entitlements":{
      "NamespaceCount":5,
      "WestEU":true
   }
}
```
## Create an entitlement set 
Creates an entitlement set. 

### Request
```text
POST api/EntitlementSets/{SetId}
```
#### Parameter
`string SetId`

The entitlement set identifier

#### Request body
#### Example request body
```json
{
   "ID":"Medium",
   "Entitlements":{
      "NamespaceCount":5,
      "WestEU":true
   }
}
```

### Authorization
Cluster operator and admin

### Response
A response code and a dictionary of entitlement in a body

|Property| Type | Default  |Optional  |
|--|--|--|--|
|ID  |String  |  |No  |
|Entitlements  |Dictionary (Entitlements)   |Default values of each defined entitlements  |Yes  |


#### Example response body
```json
{
   "ID":"Medium",
   "Entitlements":{
      "NamespaceCount":5,
      "WestEU":true
   }
}
```
## Update an entitlement set 
Updates an entitlement set.
The set updates affected ``AssignedSetEntitlements`` but not the ``TenantEntitlements`` entries. 

### Request
```text
PUT api/EntitlementSets/{SetId}
```
#### Parameter
`string SetId`

The entitlement set identifier

#### Request body
#### Example request body
```json
{
   "ID":"Medium",
   "Entitlements":{
      "NamespaceCount":5,
      "WestEU":true
   }
}
```

### Authorization
Cluster operator and admin

### Response
A response code and a dictionary of entitlement in a body

|Property| Type | Default  |Optional  |
|--|--|--|--|
|ID  |String  |  |No  |
|Entitlements  |Dictionary (Entitlements)   |Default values of each defined entitlements  |Yes  |


#### Example response body
```json
{
   "ID":"Medium",
   "Entitlements":{
      "NamespaceCount":5,
      "WestEU":true
   }
}
```
## Delete an entitlement set 
Deletes an entitlement set. 

### Request
```text
DELETE api/EntitlementSets/{SetId}
```
#### Parameter
`string SetId`

The entitlement set identifier

### Authorization
Cluster operator and admin

### Response
A status code and if there's an error, a response body 

# Assigning entitlement sets
In order to assign an entitlement set to a tenant/account, a valid ``SetId``
must be sent through a request. If the Entitlement Set does not exist in the database,
a ``404 not found`` is returned.
A set is assigned to a tenant/account in the following steps:
1. Check the validity of the set
2. Get all entitlements in the set
3. Overwrite all values in the existing tenant entitlements with values in the set.
Default values of the entitlement are assigned unless they're specified in the set. 

When the set applies to the tenant/account, the ``TenantEntitlements`` table will be updated.

## Assign an entitlement set 
Apply an entitlement set to a tenant/account. 

### Request
```text
POST /api/Tenants/{tenantId}/bulk/entitlements/{SetId}
```
#### Parameter
`string tenantId`

The tenant identifier

`string SetId`

The entitlement set identifier

### Authorization
Cluster operator and admin

### Response
A status code and if there's an error, a response body 
