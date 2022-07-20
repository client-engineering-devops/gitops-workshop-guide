
# GitOps Workshop Guide for **Sterling File Gateway/B2B Integrator**

## Overview  

<!--- cSpell:ignore gitorg YAMLs -->

The GitOps concept originated from [Weaveworks](https://www.weave.works/) back in 2017 and the goal was to automate the operations of a Kubernetes (K8s) system using a model external to the system as the source of truth ([History of GitOps](https://www.weave.works/blog/the-history-of-gitops)).

There are various GitOps structure and workflows.  This is our opinionated point of view (PoV) on how `GitOps` can be used to manage the infrastructure, services and application layers of K8s based systems.  It takes into account the various personas interacting with the system and accounts for separation of duties.

In this workshop you will learn the following: 
-   Overview of IBM's GitOps structure and workflow.  During the presentation in part one of the workshop, the instructors will present the overview of IBM's GitOps structure and workflow.  The information that is covered can be found in the [IBM GitOps Deployment Production guide](https://production-gitops.dev/gitops/structure/#gitops-principles).
-   IBM's GitOps Receipe for deploying IBM Sterling File Gateway/B2B Integrator.  During part two of this workshop presentation, the instructors will cover an overview of [IBM GitOps Receipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/sfg-recipe.md) for deploying IBM Sterling File Gateway/B2B Integrator. 
-   Lab 1 - Deploy the IBM Sterling File Gateway/B2B Integrator using the [IBM GitOps Recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/sfg-recipe.md)
-   Lab 2 - Validating Use case Requirements for Self-Healing, Upgrade/Rollback and automatic Pod Scaling.

## Lab Prerequsites - Client Environment Setup 
This part of the workshop is a hands-on lab that the instructors will walk you through to deploy an instances of IBM Sterling File Gateway/B2B Integrator.  You will be assigned a RedHat OpenShift Environment and GitHub Organization in which to run the lab.  You will need to have your IBM Cloud ID and Public GitHub ID, that you provided to sign up for the lab, available.  If you have any issues accessing the environment with your IBM ID and GitHub ID, please consult with your lab instructor.

### Environment Assignment
You should have received an e-mail from the IBM instructor with your assigned environment access based on your IBM ID and GitHub ID. 

### Login and Setup the IBM Cloud Shell Environment

1. Login to your IBM Cloud account and access the [IBM Cloud Shell](https://cloud.ibm.com/shell)

*Note that the shell session's [IBM Cloud Shell workspace](https://cloud.ibm.com/docs/cloud-shell?topic=cloud-shell-files#file-persistence) is deleted one hour after the shell session is closed.  If you loose the shell workspace, follow the steps above to re-setup the environment.*

2. Install and setup the prequiste CLIs 
```bash
mkdir bin
cd bin
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.17.4/kubeseal-0.17.4-linux-amd64.tar.gz
tar -xvf kubeseal-0.17.4-linux-amd64.tar.gz
chmod 755 kubeseal
export PATH=~/bin:$PATH
echo $PATH
```

3. Setup the environement variables
```bash
# Your-Github-Org should be the name of the github org that was created for this Lab
export GIT_ORG=Your-Github-Org
```
```bash
#Validate that GIT_ORG has the correct value.
echo $GIT_ORG 
```
4. Clone your GitOps repositories from your Github Organization 
```bash
cd ~
mkdir $GIT_ORG
cd $GIT_ORG
git clone https://github.com/$GIT_ORG/multi-tenancy-gitops.git
git clone https://github.com/$GIT_ORG/multi-tenancy-gitops-infra.git
git clone https://github.com/$GIT_ORG/multi-tenancy-gitops-services.git
ls -l
```

5. Setup your GitHub profile
```bash
# Your e-mail should be the e-mail you used to sign up for github
git config --global user.email "Your e-mail"
```
```bash
# Your Name should be the name you used to sign up for github
git config --global user.name "Your Name"
```

6. Launch the `OpenShift Web Console`. From the dropdown menu in the upper right of the page, click Copy Login Command.  Paste the copied command in your IBM Cloud shell.

![openshift_web_console](images/openshift-web-console.png "Screenshot of Openshift Web Console")

]

7. Lauch and login in to your Argo instance with the credentials provided in the environment e-mail you received from IBM TechZone.

 ![argocd_login](images/argo-launch-login-page.png "Screenshot of  ArgoCD login page")

 and verify the Argo applications 
 
 ![argocd_startpage](images/argo-start-page.png "Screenshot of  ArgoCD start page")
 
---

### Pre-Lab 1 - Create a Personal Access Token

1. Click on your user at the right top of the page and select Settings

![github_userprofile](images/github-user-profile.png "Screenshot of  GitHub User profile")

2. Scroll down until you see **Developer settings** in the left sidebar and click on it.

3. Click on Personal access tokens.
![github_pat](images/github-pat.png "Screenshot of  Personal Access Token")

4. Click on **Generate new token**
![github_pat2](images/github-pat2.png "Screenshot of  Personal Access Token2")

5. Enter the name of the token and ensure the **repo** box is checked
![github_pat_name](images/github-pat-name.png "Screenshot of  Enter PAT Name")
Scroll down and create the new token.

6. The token is displayed only once; make sure you copy it. You will need it multiple times
during the following steps.
![github_pat_token](images/github-token.png "Screenshot of  GitHub PAT Token")

---

## Lab 1 - Deploy the  IBM Sterling File Gateway/B2B Integrator using IBM's GitOps Recipe
----


### Deploy the Pre-Requisite Infrastructure Components 
In the first section of this lab, you will review the  `Infrastructure` layer in the [kustomization.yaml](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/0-bootstrap/single-cluster/1-infra/kustomization.yaml) file and un-comment the resources  based on the  [IBM SFG recipe](https://github.com/cloud-native-toolkit/multi-tenancy-gitops/blob/master/doc/sfg-recipe.md) as below.

### 1. Edit the Infrastructure layer - Kustomization.yaml file

Open the kustomization.yaml file for the infra layer as follows
```bash
vi ~/$GIT_ORG/multi-tenancy-gitops/0-bootstrap/single-cluster/1-infra/kustomization.yaml
```

You'll need to un-comment some of the k8s resources under the 'resources' field in the `kustomization.yaml` file in order to deploy the kubernetes infrastructure level resources required for Sterling B2Bi. The resources you'll need to uncomment are shown below:

```Markdown
resources:
#- argocd/consolelink.yaml
- argocd/consolenotification.yaml
#- argocd/namespace-ibm-common-services.yaml
#- argocd/namespace-ci.yaml
#- argocd/namespace-dev.yaml
#- argocd/namespace-staging.yaml
#- argocd/namespace-prod.yaml
#- argocd/namespace-cloudpak.yaml
#- argocd/namespace-istio-system.yaml
#- argocd/namespace-openldap.yaml
- argocd/namespace-sealed-secrets.yaml
#- argocd/namespace-tools.yaml
#- argocd/namespace-instana-agent.yaml
#- argocd/namespace-robot-shop.yaml
#- argocd/namespace-openshift-serverless.yaml
#- argocd/namespace-knative-eventing.yaml
#- argocd/namespace-knative-serving.yaml
#- argocd/namespace-knative-serving-ingress.yaml
#- argocd/namespace-openshift-storage.yaml
#- argocd/namespace-spp.yaml
#- argocd/namespace-spp-velero.yaml
#- argocd/namespace-baas.yaml
#- argocd/namespace-db2.yaml
#- argocd/namespace-mq.yaml
- argocd/namespace-b2bi-prod.yaml
#- argocd/namespace-b2bi-nonprod.yaml
#- argocd/serviceaccounts-ibm-common-services.yaml
#- argocd/serviceaccounts-tools.yaml
#- argocd/serviceaccounts-db2.yaml
#- argocd/serviceaccounts-mq.yaml
- argocd/serviceaccounts-b2bi-prod.yaml
#- argocd/serviceaccounts-b2bi-nonprod.yaml
- argocd/sfg-b2bi-clusterwide.yaml
#- argocd/scc-wkc-iis.yaml
#- argocd/norootsquash.yaml
- argocd/daemonset-sync-global-pullsecret.yaml
#- argocd/storage.yaml
#- argocd/infraconfig.yaml
#- argocd/machinesets.yaml
```

Now deploy these changes by committing and pushing the changes to your `multi-tenancy-gitops` repository:
```bash
#change to the `multi-tenancy-gitops` directory
cd ~/$GIT_ORG/multi-tenancy-gitops

# Verify the changes, and add the files that have been changed
git status
git add -u
 
# Finally commit and push the changes
git commit -m "for now only deploy infrastructure resources"
git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

Sync the changes in Argo at via the `infra` argo application
*** Need a picture here***


### 2. Install the Sealed Secret Service

Edit the Argo Services layer in the `multi-tenancy-gitops` **repo**   

```
vi ~/$GIT_ORG/multi-tenancy-gitops/0-bootstrap/single-cluster/2-services/kustomization.yaml
```

Install Sealed Secrets service by uncommenting the line below:

```Markdown
resources:

# Sealed Secrets
- argocd/instances/sealed-secrets.yaml

```

Now deploy the sealed-secrets service by committing and pushing the changes to your `multi-tenancy-gitops` repository:
```bash
#change to the `multi-tenancy-gitops` directory
cd ~/$GIT_ORG/multi-tenancy-gitops

# Verify the changes, and add the files that have been changed
git status
git add -u

# Finally commit and push the changes
git commit -m "only deploy the sealed secret service"

git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```
Sync the changes in Argo via the service argo application

### 3. Generate Sealed Secrets and  Volume Storage Resources required by Sterling File Gateway

Now in the `multi-tenancy-gitops-service` **repo**, change to the  B2B setup directory to generate the sealed secret and the volume storage deployment yaml files.
```bash
cd ~/$GIT_ORG/multi-tenancy-gitops-services/instances/ibm-sfg-b2bi-prod-setup
```

#### Generate Sealed Secrets resources required by Sterling File Gateway.  
The following commands will generate the yaml resource files from a template and create the deployment yaml files to deploy the sealed secrets into the cluster.

Generate a Sealed Secret for the credentials. Execute the following commands:
```bash
B2B_DB_SECRET=db2inst1 \
JMS_PASSWORD=password JMS_KEYSTORE_PASSWORD=password JMS_TRUSTSTORE_PASSWORD=password \
B2B_SYSTEM_PASSPHRASE_SECRET=password \
./sfg-b2bi-secrets.sh
```

#### Generate Persistent Volume Storage resources required by Sterling File Gateway. 
The following commands will generate the yaml resource files from a template and create the deployment yaml files to deploy the volume storage into the cluster.  Execute the command: 

```bash
./sfg-b2bi-pvc-mods.sh
```

Now deploy the generated resources changes by committing and pushing the changes to your `multi-tenancy-gitops-services` repository:
```bash
# Verify the changes by with the following command.  You should see new yaml files for the sealed secrets and volume storage yamls
git status

# Need to add the generated yaml files to git staging area
git add .

# Finally commit and push the changes
git commit -m "deploy resources needed for service"

git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

### 4. Enable DB2, MQ and prerequisites in the main multi-tenancy-gitops repository
Edit the Services layer ${GITOPS_PROFILE}/2-services/kustomization.yaml by uncommenting the following lines to install the pre-requisites for Sterling File Gateway:

```
vi ~/$GIT_ORG/multi-tenancy-gitops/0-bootstrap/single-cluster/2-services/kustomization.yaml
```
```Markdown
resources:

# B2BI
- argocd/instances/ibm-sfg-db2-prod.yaml
- argocd/instances/ibm-sfg-mq-prod.yaml
- argocd/instances/ibm-sfg-b2bi-prod-setup.yaml
#- argocd/instances/ibm-sfg-b2bi-prod.yaml
#- argocd/instances/ibm-sfg-db2-nonprod.yaml
#- argocd/instances/ibm-sfg-mq-nonprod.yaml
#- argocd/instances/ibm-sfg-b2bi-nonprod-setup.yaml
#- argocd/instances/ibm-sfg-b2bi-nonprod.yaml
```

Now deploy the resources changes by committing and pushing the changes to your `multi-tenancy-gitops` repository:
```bash
#change to the `multi-tenancy-gitops` directory
cd ~/$GIT_ORG/multi-tenancy-gitops

# Verify the changes, and add the files that have been changed
git status
git add -u

# Finally commit and push the changes
git commit -m "deploy services resources"

git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

Sync the changes in Argo  via the `service` argo application


### 5. Create the B2B Installation settings for deployment

Generate the installation settings in the `multi-tenancy-gitops-services` **repo**  by executing the following commands:

```bash
cd ~/$GIT_ORG/multi-tenancy-gitops-services/instances/ibm-sfg-b2bi-prod

./ibm-sfg-b2bi-overrides-values.sh
```

Update the  **repo** by committing and pushing the changes to your `multi-tenancy-gitops-service` repository:
```bash
# Verify the changes by with the following command.  You should see new values.yaml file generated
git status

# Need to add the generated yaml file to git staging area
git add .

# Finally commit and push the changes
git commit -m "created the deployment settings via the values.yaml"

git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

### 6. Deploy the IBM Sterling B2B Integrator 

Edit the Argo Services layer in the `multi-tenancy-gitops` **repo**  and install the IBM Sterling B2B Integrator Service 

```
vi ~/$GIT_ORG/multi-tenancy-gitops/0-bootstrap/single-cluster/2-services/kustomization.yaml
```
```Markdown
resources:

# B2BI
- argocd/instances/ibm-sfg-db2-prod.yaml
- argocd/instances/ibm-sfg-mq-prod.yaml
- argocd/instances/ibm-sfg-b2bi-prod-setup.yaml
- argocd/instances/ibm-sfg-b2bi-prod.yaml
#- argocd/instances/ibm-sfg-db2-nonprod.yaml
#- argocd/instances/ibm-sfg-mq-nonprod.yaml
#- argocd/instances/ibm-sfg-b2bi-nonprod-setup.yaml
#- argocd/instances/ibm-sfg-b2bi-nonprod.yaml
```


Now deploy the IBM Sterling B2B Integrator Service   by committing and pushing the changes to your `multi-tenancy-gitops` repository:
```bash
#change to the `multi-tenancy-gitops` directory
cd ~/$GIT_ORG/multi-tenancy-gitops

# Verify the changes, and add the files that have been changed
git status
git add -u

# Finally commit and push the changes
git commit -m "deploy the IBM Sterling B2B Integrator servcie"
git push
# Input your github username when prompted for Username
# Input the Github Token that you had created earlier when prompted for Password
```

Sync the changes in Argo  via the `service` argo application

**Note that the above sync will take approximately 1.5 hours as this part of the deployment generates the initial Sterling B2B Integrator database.**

Now verify the the Sterling File Gateway Console.  Retrieve the Sterling File Gateway console URL.
```bash
oc get route -n b2bi-prod ibm-sfg-b2bi-sfg-asi-internal-route-dashboard -o template --template='https://{{.spec.host}}'
```
and login with the default credentials:  username:`fg_sysadmin` password: `password` 



___

## Lab 2 - Validate the Use Cases for Self-Healing, Upgrade/Rollback and automatic Pod Scaling
The final part of the hands-on lab is to validate the three use cases (Self-Healing, Upgrade/RollBack, and Automatic Pod Scaling).

See the [use case instructions](./Scenarios.md) to complete final part of the workshnop.

---
