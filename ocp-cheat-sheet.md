
# Openshift cheat sheet
Openshift commands for day to day usage
# Index
- [Templates](#templates)
- [Helm charts](#helm-charts)
- [Authentication](#authentication)
- [Role management](#role-management)
- [TLS security](#tls-security)
	- [external traffic](#external-traffic)
	- [internal traffic](#internal-traffic)
- [Network Policies](#network-policies)
- [Load Balancers](#load-balancers)
- [Quota and Resources](#quota-and-resources)
- [Limit Ranges](#limit-ranges)
- [Operators](#operators)

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
creating a bootstrap template
```
$ oc adm create-bootstrap-project-template -o yaml >my-template.yaml
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

## tls security

### external traffic
creating a secret with TLS certs:
```
$ oc create secret tls <name> --cert <path-to-cert> --key <path-to-key>
```
insecure service exposure:
```
$ oc expose svc <pod name> --hostname <hostname>
```
exposing a service with the edge termination (encryption only in the edge)
```
$ oc create route edge <name> --service <svc> --hostname <hostname>
```
exposing aservice with the passthrough termination (encryption in the pod level)
```
$ oc create route passthrough <name> --service <svc> --port 8443 --hostname <hostname>
```
### internal traffic
create certs and secret using the `service-ca` controller:
```
$ oc annotate service <svc> service.beta.openshift.io/serving-cert-secret-name=<svc>
```

saving CA in a configmap
```
$ oc annotate configmap <existing configmap> service.beta.openshift.io/inject-cabundle=true
```
_both certs secret and CA configmap can be used in deployments or Pod declarations_

to rotate secrets using the `service-ca` controller:
```
$ oc delete secret <certificate secret created by service-ca>
```
to rotate the CA
```
$ oc delete secret/signing-key -n openshift-service-ca
```

## network policies

creating a deny all policy - all pods unreacheable
```
$ cat > deny-all.yaml <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all
spec:
  podSelector:{}
  ingress: []
EOF

$ oc create -f deny-all.yaml
```
allowing specific traffic to the pod labeled `ingress-allowed: true` from pods/namespaces labeled `network: internal-network` and
from the pod labeled `allowed-specific: true`
to the `tcp` port `80`
```
$ cat > allow-specific.yaml <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-specific
spec:
  podSelector:
    matchLabels:
      ingress-allowed: true
  ingress:
    - from
      - namespaceSelector:
          matchLabels:
            network: internal-network
          podSelector:
            matchLabels:
              allowed-specific: true
      ports:
      - port: 80
        protocol: TCP
EOF

$ oc create -f allow-specific.yaml
```

list network policies
```
$ oc get networkpolicies -n <namespace>
```

allow traffic from openshift ingress policy group
```
$ cat > allow-from-ingress.yaml <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-from-ingress
spec:
  podSelector: {}
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            network.openshift.io/policy-group: ingress
EOF

$ oc create -f allow-from-ingress.yaml
```

## load balancers

create a service named `myservice`of type `LoadBalancer` listening in the port `80` and targeting the Pod port `8080`
using the `dry-run='client'` option, so it can be customized later with proper labels:

```
$ oc create service loadbalancer myservice --tcp=80:8080 --dry-run='client' -o yaml > myservice.yaml
_edit the file_
$ oc create -f myservice.yaml
```

expose a existing service using the `LoadBalancer` type
```
$ oc expose <pod/deployment name> --name=<desired service name> --type=LoadBalancer
```

## quota and resources
set requests cpu quota to 1 for a pod
```
$ oc set resources deployment <name> --requests=cpu=1
```
scaling deployment
```
$ oc scale deployment <name> --replicas=<desired number of replicas>
```
getting events for the current namespace sorted by creation time
```
$ oc get event --sort-by .metadata.creationTimeStamp
```

to check system resource usage
```
$ oc adm top node
```
too check the node resources
```
$ oc describe node/master01
```

creating a new quota object with hard requests.cpu=2
```
$ oc create quota <name> --hard=requests.cpu=2
```
## limit ranges
creating a LimitRange
```
$ cat > limit-range.yaml <<EOF
apiVersion: v1
kind: LimitRange
metadata:
  name: <limit range name>
  namespace: <namespace>
spec:
  limits:
  - min:
      memory: 128Mi
    defaultRequests:
      memory: 256Mi
    default:
      memory: 512Mi
    max:
      memory: 1Gi
    type: Container
EOF

$ oc create -f limit-range.yaml
```
describing a limit range
```
$ oc describe limitrange <name>
```

## operators
getting the package manifests list
```
$ oc get packagemanifests
```
describing a package manifest
```
$ oc describe packagemanifest <name>
```
creating a subscription:
```
$ cat >subscription.yaml <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: subscription
metadata:
  name: <operator-name>
  metadata: <operator namespace to run>
spec:
  channel: "stable"
  installPlanApproval: Manual
  name: <operator name>
  source: <package manifest catalog source name>
  sourceNamespace: openshift-marketplace

EOF
$ oc create -f subscription.yaml
```
getting the operator installplan:
```
$ oc get installplan -n <operator install namespace> -o jsonpath='{.spec}{"\n"}'
```
approve manually the installplan:
```
$ oc patch installplan --type merge -p '{"spec": {"approved": true}}' -n <operator install namespace>
```
checking the operator install progress
```
$ oc describe operator <name>
```
get operator subscription info for _web-terminal_ operator
```
$ oc get sub -n openshift-operators web-terminal
```

get installplan for the operator
```
$ oc get installplan -n openshift-operators install-xxx
```

approve the installplan
```
$ oc patch installplan install-xxx --type merge \
--patch '{"spec": {"approved":  true}}'
```

## cluster updates
getting the cluster update status
```
$ oc get clusterversion
```
channels:
```
candidate-4.x
fast-4.x
stable-4.x
```
changing the channel in command line:
```
$ oc patch clusterversion version --type="merge" --patch '{"spec": {"channel": "stable-4.14"}}'
```
pausing machine healthcheck
```
$ oc annotate machinehealthcheck -n openshift-machine-api machine-api-termination-handler cluster.x-k8s.io/paused=""
```
unpausing
```
$ oc annotate machinehealthcheck -n openshift-machine-api machine-api-termination-handler cluster.x-k8s.io/paused-
```
to start the upgrade
```
$ oc adm upgrade
```
force upgrade to the latest version
```
$ oc adm upgrade --to-latest=true
```
upgrading for a specific version
```
$ oc adm upgrade --to=<version>
```

get detailed information on the upgrade status
```
$ oc describe clusterversion
```

## deprecated apis
```
$ oc api-resources |grep "^NAME|<api name>"
```

get api usage per version
```
oc get apirequestcounts | awk '{if(NF==4){print $0}}'
```
manual acknownledgement of an api upgrade
```
$ oc patch configma admin-acks -n openshift-config --type=merge \
--patch '{"data": {"ack-4.13-kube-1.27-api-removals-in-4.14": "true"}}'
```
