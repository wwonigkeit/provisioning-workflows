# Direktiv Internal Provisioning Example (Enterprise Edition)

The repository contains example workflows for Enterprise Edition internal Direktiv provisioning:
- **[setup-master-workflow.yaml](https://github.com/wwonigkeit/provisioning-workflows#setup-master-workflowyaml):** master workflow with the inputs and event sequences
- **[direktiv-user-setup/keycloak-create-user.yaml](https://github.com/wwonigkeit/provisioning-workflows#direktiv-user-setupkeycloak-create-useryaml):** workflow creates the suer and group from the master workflow input in Keycloak
- **[direktiv-namespace-setup/create-namespace.yaml](https://github.com/wwonigkeit/provisioning-workflows#direktiv-namespace-setupcreate-namespaceyaml):** creates the namespace, and associates the Keycloak group name
- **[direktiv-workflow-setup/clone-workflows.yaml](https://github.com/wwonigkeit/provisioning-workflows#direktiv-workflow-setupclone-workflowsyaml):** clone the example workflows into the newly created namespace

### Description

This is an example worklfow on how to provision a user, password and group in Keycloak. The namespace (associate with the group in Keycloak) is then created, the group permissions set and an authentication token is generated. This token is then used to clone an example workflow repository from the public GitHub direktiv workflow example repository.

### NOTE REQUIREMENTS
 - **This will ONLY work with the Enterprise Edition of Direktiv**
 - **The Enterprise Edition has to be version v0.7.0-rc1 or higher**
 - **A new "admin" level token is created during the installation of the Enterprise Edition. For users who already have Enterprise Edition installed and want to upgrade, the following procedure:**

 1. On the Jumphost where the helm charts are stored, run the following command: ```helm repo update```
 ```bash
 $ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "oauth2-proxy" chart repository
...Successfully got an update from the "linkerd" chart repository
...Successfully got an update from the "direktiv" chart repository
...Successfully got an update from the "bitnami" chart repository
```

2. Run the following command to confirm the update: ```helm search repo | grep direktiv```
```bash
$ helm search repo | grep direktiv
direktiv/direktiv                           	0.1.10       	v0.7.0-rc1   	direktiv helm chart                               
direktiv/knative                            	0.4.5        	1.5.0        	knative for direktiv                              
direktiv/pgo                                	0.2.3        	5.0.4        	Installer for PGO, the open source Postgres Ope...
```

3. Edit file created during the initial install: ```vi install/04_direktiv/direktiv_prod.yaml```. Add the ```tag: v0.7.0-rc1``` (depending on the version upgrading to) to the ```ui, flow, functions and api``` items. Do not change any of the other items!! Example file is shown below:
```bash
ui:
  replicas: 1
  image: direktiv/ui-ee
  url: "dev.direktiv.io/api"
  tag: v0.7.0-rc1

flow:
  replicas: 1
  tag: v0.7.0-rc1

functions: 
  replicas: 1
  tag: v0.7.0-rc1

api:
  replicas: 1
  image: direktiv/api-ee
  tag: v0.7.0-rc1
  additionalEnvs:
  - name: EE_DB_CONN
    value: "host=direktiv-primary.postgres.svc port=5432 user=<user> password=<password> dbname=direktiv sslmode=require"
```

4. Run the following command: ```helm upgrade -f direktiv_prod.yaml direktiv direktiv/direktiv```
```bash
$ helm upgrade -f direktiv_prod.yaml direktiv direktiv/direktiv
Release "direktiv" has been upgraded. Happy Helming!
NAME: direktiv
LAST DEPLOYED: Mon Sep  5 15:45:33 2022
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
$
```

5. Get the admin authentication token: ```kubectl get secret direktiv-secrets-api -o json | jq -r ".data.adminkey" | base64 --decode```
```bash
$ kubectl get secret direktiv-secrets-api -o json | jq -r ".data.adminkey" | base64 --decode
<TOKEN HERE>
$
```

6. Restart the Direktiv API pod: ```kubectl delete pod direktiv-api-RANDOM```
```bash
$ kubectl delete pod direktiv-api-56f546d477-686x5
pod "direktiv-api-56f546d477-686x5" deleted
$ kubectl get pods 
NAME                                                 READY   STATUS    RESTARTS   AGE
direktiv-api-56f546d477-hgp8f                        2/2     Running   0          10s
direktiv-flow-d9b8b7d7b-26k9q                        3/3     Running   0          23m
direktiv-functions-85799b8df8-pmnhj                  2/2     Running   0          23m
direktiv-ingress-nginx-controller-6c694bdb6c-65hgx   1/1     Running   0          4d15h
direktiv-prometheus-server-7f49cf58b8-5x6s6          3/3     Running   0          4d15h
direktiv-ui-77d58c7856-wzsjf                         2/2     Running   0          23m
oauth2-oauth2-proxy-57bbf74856-wjf27                 1/1     Running   0          4d15h
redis-master-0                                       1/1     Running   0          4d15h
redis-replicas-0                                     1/1     Running   0          4d15h
$
```

The workflow needs the following inputs:
 - direktivurl: typicall https://dev.direktiv.io
 - namespace: namespace created for the workflows
 - username: Username to configure
 - password: Initial password for the user
 - email: Email address for the user

The following secret needs to be created:
 - **DIREKTIV_ADMIN_TOKEN**: the admin token generated during install (or upgrade process)
 - **KEYCLOAK_ADMIN_USER**: keycloak administartor username
 - **KEYCLOAK_ADMIN_PASSWORD**: keycloak administartor password