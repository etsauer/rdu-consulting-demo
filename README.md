# Demo (or Lab) of a Container Image Supply Chain in OpenShift

The intent of this demo is to represent how an Operations or Middleware Ops team can centrally manage an organization standard set of container images, and automatically roll out patches to running applications that are consuming those images. For the purpose of speed and simplicity, this demo uses the `cake-php-mysql-ephemeral` that ships with OpenShift Container Platform.

This content can be run purely as a demo (Presenter does everything while audience watches) or as a hands on Lab where the audience gets to log into the demo cluster and play the part of Application Development Teams.

## Bill of Materials

Here are the things you will need in order to run through this demo:

* An OpenShift cluster accessible to your audience, capable of handling the proper number of users for the given audience.
 * Each audince member will need enough space to run at least 3 pods
* A user created in OpenShft for each audience member
 * Simplest way to do this is to enable htpasswd auth in OpenShift and do something like the following on your master(s):
 ```
 for num in {1..30}; do htpasswd -b /etc/origin/master/htpasswd demo-user${num} demo-pass${num}
 ```
* A fork of the demo repo `https://github.com/redhat-cop/ocp-supply-chain-demo` as you will need to make a Dockerfile change


## Instructor Setup

Before you begin (or as part of the introduction to the demo),

```
oc new-build https://github.com/redhat-cop/ocp-supply-chain-demo.git --name=rdu-php-demo --strategy=docker --docker-image=registry.access.redhat.com/rhscl/php-56-rhel7 -n openshift
```

Add the following line to Dockerfile:
```
ADD security.txt /opt/app-root/etc/security.txt
```

Run the following:
```
oc start-build rdu-php-demo -n openshift
```

## Participant Instructions

1. Click "New Project"
2. Enter the following project name "rdu-user{X}-demo"
3. Search for the template named `rdu-php-template`
4. (optional) Fork cakephp git repo, and replace the default git repo with yours
5. Click "Create"
6. Inspect the pod using the Terminal feature. Specifically check the `/opt/app-root/etc/` directory.
7. Sit back and watch while the Operator builds a new image
8. Inspect the pod again and see what's different
