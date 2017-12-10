# CI/CD Demo with OpenShift Origin

### Reference guide:

* [CI/CD Demo on OpenShift]

### Preconditions:

* RHEL 7 or CentOS 7 on bare metal or VM
* Docker CE installed
* OpenShift Origin installed

## Step 0: Pre-pull the images to make sure the deployments go faster

```sh
$ docker pull openshiftdemos/nexus:2.13.0-01

$ docker pull openshiftdemos/gogs:0.11.29

$ docker pull openshiftdemos/sonarqube:6.7

$ docker pull registry.access.redhat.com/openshift3/jenkins-2-rhel7

$ docker pull registry.access.redhat.com/openshift3/jenkins-slave-maven-rhel7

$ docker pull registry.access.redhat.com/jboss-eap-7/eap70-openshift
```

## Step 1: Create projects for CI/CD components, Dev and Stage environments

Start and access OpenShift:

```sh
$ oc cluster up

$ oc login -u developer
```

Open the OpenShift Web Console in browser: https://127.0.0.1:8443

Create new projects:

```sh
$ oc new-project dev --display-name="Tasks - Dev"

$ oc new-project stage --display-name="Tasks - Stage"

$ oc new-project cicd --display-name="CI/CD"
```

Add edit role to the Jenkins service account to access Dev and Stage environments:

```sh
$ oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev

$ oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n stage
```

Deploy Jenkins and install selected plugins:

```sh
$ oc new-app jenkins-persistent --param=MEMORY_LIMIT=1Gi -e INSTALL_PLUGINS=analysis-core:1.92,findbugs:4.71,pmd:3.49,checkstyle:3.49,dependency-check-jenkins-plugin:2.1.1,htmlpublisher:1.14,jacoco:2.2.1,analysis-collector:1.52 -n cicd
```

> * Wait for Jenkins deployment to finish (OpenShift Console - CI/CD Project).
> * Check for Jenkins readiness (https://jenkins-cicd.127.0.0.1.nip.io/).

## Step 2: Checkout openshift-cd-demo project locally

```sh
$ git clone https://github.com/OpenShiftDemos/openshift-cd-demo
```

Switch to branch ``ocp-3.6``:
```sh
$ git checkout ocp-3.6
```

## Step 3: Deploy auxiliary tools (Gogs, Nexus, Sonarqube) and the pipeline

Deploy with SonarQube:

```sh
$ oc new-app -n cicd -f cicd-template-with-sonar.yaml
```

Deploy without SonarQube:

```sh
$ oc new-app -n cicd -f cicd-template.yaml
```

> * Wait for all Pods to deploy (OpenShift Console - CI/CD Project)
> * Check for SonarQube readiness (http://sonarqube-cicd.127.0.0.1.nip.io/)
> * Check for Gogs readiness (http://gogs-cicd.127.0.0.1.nip.io/)
> * Check for Nexus readiness (http://nexus-cicd.127.0.0.1.nip.io/)
> * Nexus credentials: admin/admin123 | deployment/deployment123

## Step 4: Checkout openshift-tasks project on Gogs

* Enter Gogs (http://gogs-cicd.127.0.0.1.nip.io/)
* Login with credentials: gogs/password
* Create -> New Migration -> 
  * Clone Address: https://github.com/OpenShiftDemos/openshift-tasks.git
  * Repository Name: openshift-tasks
  * Migrate Repository

## Step 5: Import the JBoss imagestreams in OpenShift

```sh
$ oc login -u system:admin

$ oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
```

## Step 6: Launch tasks-pipeline

Either:

* From OpenShift Web Console:
  * Builds -> Pipelines -> Start Pipeline
* From Jenkins:
  * cicd/tasks-pipeline -> Schedule a Build
* From Terminal:
  * ``$ oc start-build tasks-pipeline``

Manually Promote to STAGE.

[CI/CD Demo on OpenShift]: <https://github.com/OpenShiftDemos/openshift-cd-demo>