# Building a catalog #

## Our example company ##

Our example company Acme Corp

<div hidden>
```
@startuml ch2-organigram
!include <awslib/AWSCommon>
!include <awslib/AWSSimplified.puml>
!include <awslib/Compute/all.puml>
!include <awslib/mobile/all.puml>
!include <awslib/general/all.puml>
!include <awslib/GroupIcons/all.puml>

package non-technical {
  Users(management,"Management","management@acme.corp")
  Users(accounting,"Accounting","Accounting@acme.corp")
  Users(hr,"Human Resources","hr@acme.corp")
}

package dev {
  Users(mobiledev,"Mobile","dev.mobile@acme.corp")
  Users(frontenddev,"Frontend","dev.frontend@acme.corp")
  Users(backenddev,"Backend","dev.backend@acme.corp")
}

package operations {
  Users(opdba,"Database Administrators","op.dba@acme.corp")
  Users(opsys,"System Administrators","op.sys@acme.corp")
}

management -[hidden]r- accounting
accounting -[hidden]r- hr
management -[hidden]d- mobiledev
mobiledev -[hidden]r- frontenddev
frontenddev -[hidden]r- backenddev
opdba -[hidden]r- opsys

@enduml
```
</div>

![architecture example](ch2-organigram.svg)

## Organization skeleton ##

We create a skeleton for our organization. Ideally the group/users would come from ingestion of the corporate identity management software. Note the Location Kind to target different catalog files. We point for individual catalog files for each team as well as a domain specific file that we will use as a basis for the rest of our workshop. We could technically fill the (mandatory) children field of our organization but it is easier to use the parent field of the sub groups.

```
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  name: acmecorp
  title: ACME corp
spec:
  type: organization
  children: []
---
apiVersion: backstage.io/v1alpha1
kind: Location
metadata:
  name: teams
spec:
  targets:
    - ./teams/dev/catalog-info.yaml
    - ./teams/operations/catalog-info.yaml
    - ./domains/catalog-info.yaml
```

## Our example architecture ##

Here is an examples of the architecture of our webshop.

<div hidden>
```
@startuml ch2-architecture-diagram
!include <awslib/AWSCommon>
!include <awslib/AWSSimplified.puml>
!include <awslib/Compute/all.puml>
!include <awslib/mobile/all.puml>
!include <awslib/general/all.puml>

'skinparam linetype ortho

Users(Users, "Users", "")
Mobile(MobileApp, "Mobile App", "")
Client(Browser, "Web Browser", "")
GenericDatabase(AccountDB, "Account DB", "")
GenericDatabase(InventoryDB, "Inventory DB", "")
GenericDatabase(SaleDB, "Sale DB", "")
GenericDatabase(ShippingDB, "Shipping DB", "")
ECSContainer1(WebApp, "Web frontend", "")
APIGateway(APIgw, "API Gateway", "")
LambdaLambdaFunction(Accountsvc, "Account Service", "")
LambdaLambdaFunction(Inventorysvc, "Inventory Service", "")
LambdaLambdaFunction(Shippingsvc, "Shipping Service", "")
ECSContainer1(Salesvc, "Sale Service", "")

MobileApp -[hidden]r- Browser
APIgw -[hidden]r- WebApp
Accountsvc -[hidden]r- Inventorysvc
Inventorysvc -[hidden]r- Salesvc
Salesvc -[hidden]r- Shippingsvc
AccountDB -[hidden]r- InventoryDB
InventoryDB -[hidden]r- SaleDB
SaleDB -[hidden]r- ShippingDB

Users -- MobileApp
Users -- Browser
MobileApp -- APIgw
Browser -- WebApp
APIgw -- Accountsvc
WebApp -- Accountsvc
APIgw -- Inventorysvc
WebApp -- Inventorysvc
APIgw -- Salesvc
WebApp -- Salesvc
APIgw -- Shippingsvc
WebApp -- Shippingsvc
Accountsvc -- AccountDB
Inventorysvc -- InventoryDB
Shippingsvc -- ShippingDB
Salesvc -- SaleDB

@enduml
```
</div>

![architecture example](ch2-architecture-diagram.svg)

## Identifying components and resources ##

End users and Browser are not software entities of our webshop product, we will ignore them in the next parts.
DB and amazon api gateway are resources, as the code is not managed by a development team.
All others entities are components.

<div hidden>
```
@startuml ch2-components-resources
!include <awslib/AWSCommon>
!include <awslib/Compute/all.puml>
!include <awslib/mobile/all.puml>
!include <awslib/general/all.puml>

'skinparam linetype ortho

Users(Users, "Users", "")
Mobile(MobileApp, "Mobile App", "Component")
Client(Browser, "Web Browser", "")
GenericDatabase(AccountDB, "Account DB", "Resource")
GenericDatabase(InventoryDB, "Inventory DB", "Resource")
GenericDatabase(SaleDB, "Sale DB", "Resource")
GenericDatabase(ShippingDB, "Shipping DB", "Resource")
ECSContainer1(WebApp, "Web frontend", "Component")
APIGateway(APIgw, "API Gateway", "Resource")
LambdaLambdaFunction(Accountsvc, "Account Service", "Component")
LambdaLambdaFunction(Inventorysvc, "Inventory Service", "Component")
LambdaLambdaFunction(Shippingsvc, "Shipping Service", "Component")
ECSContainer1(Salesvc, "Sale Service", "Component")

MobileApp -[hidden]r- Browser
APIgw -[hidden]r- WebApp
Accountsvc -[hidden]r- Inventorysvc
Inventorysvc -[hidden]r- Salesvc
Salesvc -[hidden]r- Shippingsvc
AccountDB -[hidden]r- InventoryDB
InventoryDB -[hidden]r- SaleDB
SaleDB -[hidden]r- ShippingDB

Users -- MobileApp
Users -- Browser
MobileApp -- APIgw
Browser -- WebApp
APIgw -- Accountsvc
WebApp -- Accountsvc
APIgw -- Inventorysvc
WebApp -- Inventorysvc
APIgw -- Salesvc
WebApp -- Salesvc
APIgw -- Shippingsvc
WebApp -- Shippingsvc
Accountsvc -- AccountDB
Inventorysvc -- InventoryDB
Shippingsvc -- ShippingDB
Salesvc -- SaleDB

@enduml
```
</div>

![architecture example](ch2-components-resources.svg)
 
## Creating our first Components ##

Types are dynamically expandable we'll make use of that to easily filter out compnents

```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: webapp
  title: Webshop Frontend
spec:
  type: website
  owner: frontend
```

```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: Accountsvc
  title: Account Service
spec:
  type: lambda-function
  owner: backend
```

## Creating our first resources ##

Here we will define our resources, here is an example of it.
Note that the owner is the backend team, who uses the resource, instead of the operationnal team (dba) that might do the operationnal management.

```
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: AccountDB
  title: Account Database
spec:
  type: Postgresql-database
  owner: backend
```

## System and Domain ##

Our Domain will comprise the entire webshop. We can separate the Account, Inventory, Sale and Shipping logical functions as separate systems.

<div hidden>
```
@startuml ch2-system-domain
!include <awslib/AWSCommon>
!include <awslib/AWSSimplified.puml>
!include <awslib/Compute/all.puml>
!include <awslib/mobile/all.puml>
!include <awslib/general/all.puml>
!include <awslib/GroupIcons/all.puml>
!include <awslib/Storage/all.puml>
!include <awslib/ManagementAndGovernance/all.puml>
!include <awslib/CustomerEngagement/all.puml>
!include <awslib/MachineLearning/all.puml>
!include <awslib/NetworkingAndContentDelivery/all.puml>
!include <awslib/Database/all.puml>
!include <awslib/ApplicationIntegration/all.puml>

'skinparam linetype ortho

Package WebShop {
  Mobile(MobileApp, "Mobile App", "Component")
  Package Account {
    LambdaLambdaFunction(Accountsvc, "Account Service", "Component")
    GenericDatabase(AccountDB, "Account DB", "Resource")
  }
  Package Inventory {
    LambdaLambdaFunction(Inventorysvc, "Inventory Service", "Component")
    GenericDatabase(InventoryDB, "Inventory DB", "Resource")
  }
  Package Sale {
    ECSContainer1(Salesvc, "Sale Service", "Component")
    GenericDatabase(SaleDB, "Sale DB", "Resource")
  }
  Package Shipping {
    LambdaLambdaFunction(Shippingsvc, "Shipping Service", "Component")
    GenericDatabase(ShippingDB, "Shipping DB", "Resource")
  }
  
  ECSContainer1(WebApp, "Web frontend", "Component")
  APIGateway(APIgw, "API Gateway", "Resource")
}

APIgw -[hidden]l- WebApp
Accountsvc -[hidden]r- Inventorysvc
Inventorysvc -[hidden]r- Salesvc
Salesvc -[hidden]r- Shippingsvc
AccountDB -[hidden]r- InventoryDB
InventoryDB -[hidden]r- SaleDB
SaleDB -[hidden]r- ShippingDB

MobileApp -- APIgw
Browser --- WebApp
APIgw -- Accountsvc
WebApp -- Accountsvc
APIgw -- Inventorysvc
WebApp -- Inventorysvc
APIgw -- Salesvc
WebApp -- Salesvc
APIgw -- Shippingsvc
WebApp -- Shippingsvc
Accountsvc -- AccountDB
Inventorysvc -- InventoryDB
Shippingsvc -- ShippingDB
Salesvc -- SaleDB

@enduml
```
</div>

![architecture example](ch2-system-domain.svg)

## Domain example ##

```
apiVersion: backstage.io/v1alpha1
kind: Domain
metadata:
  name: webshop
spec:
  owner: dev-team
```

## System example ##

```
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: mobile-app
spec:
  owner: mobile
```

