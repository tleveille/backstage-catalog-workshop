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

/!\ Note that we use $text descriptor with a json input as the API kind will interpret it on its own.

## Describing relations between entities ##


