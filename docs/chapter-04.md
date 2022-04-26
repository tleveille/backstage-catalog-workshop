# Links, Tags, Labels and annotations #

## Links ##

List of hyperlinks, typically used to point to external urls.

There are 3 fields:
- url [required]
- title [optionnal]
- icon [optionnal]: map to an icon library, by default [material-ui](https://mui.com/material-ui/material-icons/)

```
# Example:
metadata:
  links:
     - url: https://shop.acmecorp
       title: Acme Web shop [production]
       icon: Home
     - url: https://dev.shop.acmecorp
       title: Acme Web shop [dev - vpnonly]
       icon: Construction
```

See: https://backstage.io/docs/features/software-catalog/descriptor-format#links-optional

## Tags ##

metadata.tags are an easy metadata to add to your entities and provide immediate filtering capabilities within the backstage ui.

It is a practical way to reference things like:
- technology
- programming language
- cloud vendor
- cloud service

```
# Example:
- aws
- lambda
- nodejs-16
```

limitations: single word, [a-z0-9] separated by -, max 63 characters

## Labels

Optionnal key/value pairs attached to the entities, similar to Kubernetes labels.

Name and value has same limitations as tag, prefix can be up to 253 characters lowercase domain name.

We advise not to use them at this point as it is currently not used in the backstage UI

## Annotations

Again similar to Kubernetes annotations, prefix and name similar to labels but value can be of any length.

Annotations are mostly used by plugins to point to the correct ressources (jira project name, ci/cd pipelines, pagerduty token...)

```
# Example:
metadata:
  annotations:
    sonarqube.org/project-key: acmeweb
```