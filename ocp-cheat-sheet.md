
# Openshift cheat sheet

## templates
To process a template
```
$ oc process --parameters <template>
```
creating a new app from template
```
$ oc newapp --template=<template>
```
processing a template with inline parameters
```
$ oc process <template> -p PARAM1=value1 -p PARAM2=valu2
```
processing a template with parameters from a file
```
$ oc process <template> --param-file=<path to param file>
```

## helm charts

to list repos
```
$ help repo list
```
to add a new repo
```
$ help repo add <name> <url> <flags>
```
to search for a repo (with versions)
```
$ help search repo --versions
```
to show chart values
```
$ helm show values <name>
```
to install a chart
```
$ helm install <app> <repo> -f values.yaml --version <ver>
```
to update values
```
$ help upgrade <app> <repo> -f updated-values.yaml --version <new ver>
```
to list installed apps
```
helm list
```
## authentication
Adding htpasswd auth

creating htpasswd secret
```
$ oc create secret generic <secret name> --from-file htpasswd=<path to htpasswd file> -n <namespace>
```
to edit the system oauth
```
$ oc edit oauth
```
adding the htpasswd to oauth:
```
...
  identityProviders:
  - ldap:
	...
  - name: <htpasswd provider name>
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: <htpasswd secret name>
```
watching the authentication pods
```
$ watch "oc get pod -n openshift-authentication"
```
## role management
to add a cluster role to a user or group:
```
$ oc adm policy add-cluster-role-to-<user|group> <role> <user|group>
```

to remove a cluster role from a user/group:
```
$ oc adm policy remove-cluster-role-from-<user|group> <role> <user|group>
```
to add a new group:
```
$ oc adm groups new <group>
```
adding users to group
```
$ oc adm groups add-users <group> <user>
```
adding the self-provisioners role (to create projects)
```
$ oc adm policy add-cluster-role-to-group self-provisioners <group>
```
add *edit* role to group:
```
$ oc policy add-role-to-group edit <group>
```
add *view* role to group:
```
$ oc policy add-role-to-group view <group>
```