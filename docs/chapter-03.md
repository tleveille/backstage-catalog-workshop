# APIs and dependencies #

## The API Kind ##

The API kind allow to create an entity describing an API provided by a system and its components. It supports out of the box different kinds of API definition:

- openapi
- asyncapi
- graphql
- grpc

The actual definition of the API can be described in the spec.definition field.

## Describing an API inline ##

Here is an example of an openapi API whose YAML is described inline in the spec.definition field:

```
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: acme.webshop.shipping.api
  description: Shipping API
spec:
  type: openapi
  lifecycle: production
  owner: backend
  system: acme.webshop.shipping
  definition: |
    openapi: "3.0.0"
    info:
      version: 1.0.0
      title: Shipping API
    servers:
      - url: http://webshopt.acmecorp/shipping/v1
    paths:
      /Parcel:
        get:
          summary: List all parcels
    ...
```

## Describing an API using the descriptor format ##

The descriptor format allow substitutions from a link to the actual content of the linked file. There are 3 descriptor format supported:

- $text Interprets the contents of the referenced file as plain text and embeds it as a string.
- $json Interprets the contents of the referenced file as JSON and embeds the parsed structure.
- $yaml Interprets the contents of the referenced file as YAML and embeds the parsed structure. 

Reference link can be absolute or relative to the folder of the actual entity catalog-info.yaml

Example in the case of an API definition:
```
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: acme.webshop.account.api
  description: Account API
spec:
  type: openapi
  lifecycle: production
  owner: backend
  definition:
    $text: https://petstore.swagger.io/v2/swagger.json
```
/!\ Note that we use `$text` descriptor with a json input as the API kind will interpret ddit on its own.

We can create all our apis linking to the same json file unless you have one available.


## Describing relations between entities ##

We already used some relationnal fields when we dealt with ownership and group membership. Here are all the relationnal fields possible.

| Field               | Description                                                                                   | Applicable Kind |
| ------------------- | --------------------------------------------------------------------------------------------- | --------------- |
| spec.owner          | Reference to the owner of the entity. Value kinds:<br>group<br>user                           | Domain<br>Component<br>Template<br>API<br>Resource |
| spec.parent         | An array of components and/or resource a component depends on.                                | Group |
| spec.children       | The reverse of spec.parent. Must be an array.<br>Mandatory field for groups.<br>Can be empty. | Group | 
| spec.memberOf       | An array of components and/or resource a component depends on.                                | User |
| spec.members        | An array of direct members of a group.<br>Optionnal, easier to use membersOf.                 | Group |
| spec.system         | The system the component belongs to.<br>Optionnal.                                            | Component<br>API<br>Resource |
| spec.dependsOn      | An array of components and/or resources a component depends on.                               | Component<br>Resource |
| spec.dependencysOf  | Reverse of spec.dependsOn but only for resource.<br>Best to stick with dependsOn only.        | Resource  |
| spec.subcomponentOf | Reference another component of which it is part of.<br>Should we use it?                      | Component |
| spec.consumesApi    | An array of APIs consumed by a  component.                                                    | Component |
| spec.providesApi    | An array of APIs a component provides.                                                        | Component |

## Relations in our example ##

Looking back at our domain:

![architecture example](ch2-system-domain.svg)

- We will use the field `spec.system` to reference which system our components, APIs and resources are part of.
- We will use the field `spec.providesApi` on our 4 services to reference the APIs they provide.
- We will use the field `spec.consumesApi` to the webserver component and MobileApp.
- We will use the field `spec.dependsOn` to reference the DB resources used by the services.

## Our new systems ##

Here is how our account.yaml describing the account system should look:

```
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: acme.webshop.account
  title: ACME Shop account
spec:
  owner: backend
---
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: acme.webshop.account.service
  title: ACME Shop account service
spec:
  system: acme.webshop.account
  type: service
  owner: backend
  lifecycle: production
  providesApi: acme.webshop.account.api
---
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: acme.webshop.account.db
  title: ACME Shop account database
spec:
  system: acme.webshop.account
  owner: backend
  type: database
---
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: acme.webshop.account.api
  description: Account API
spec:
  system: acme.webshop.account
  type: openapi
  lifecycle: production
  owner: backend
  definition:
    $text: https://petstore.swagger.io/v2/swagger.json
```

## consumesAPI example ##

Our Mobile app components will look like that. The Web frontend will have the same consumesApi array but no the dependsOn statement:

```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: acme.webshop.android.app
  title: ACME Shop android app
spec:
  system: acme.webshop
  owner: mobile
  type: app
  lifecycle: experimental
  consumesApi:
    - acme.webshop.account.api
    - acme.webshop.inventory.api
    - acme.webshop.sale.api
    - acme.webshop.shipping.api
  dependsOn: acme.webshop.apigw
```