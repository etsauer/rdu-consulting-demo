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

Before you begin (or as part of the introduction to the demo), tell the room that you are all part of some fake corporation (be colorful, make up a name for yourselves). Explain that you (the instructor) are playing the part of the operations team for the company, and that each of the audience members represent an application team.

Now you can explain that you are about to establish a corporate standard image for a PHP application, based on a base image coming from Red Hat. The idea here is that we are establishing a baseline to be able to layer additional libraries, packages, etc into the image that our corporation deems as standard. You can show them the very simple [Dockerfile](Dockerfile) that we're using as a base for our image, then set up a buildConfig to produce the image with the command below.

```
oc new-build https://github.com/redhat-cop/ocp-supply-chain-demo.git --name=rdu-php-demo --strategy=docker --docker-image=registry.access.redhat.com/rhscl/php-56-rhel7 -n openshift
```

Note that the buildConfig gets created in the `openshift` project, which is globally readable by all users, which means the image itself will be available to all Developent teams to use as the base for their application.

In order to facilitate getting the app teams started using our standard PHP image, you will also deploy a quickstart app template to your cluster:

```
oc create -f php-supply-chain-app.yml -n openshift
```

## Participant Instructions

Now we get into the audience participation portion of the Lab. If you are just doing this as a pure demo, make note that you are switching hats from the Operator to App Team as you move on to the following steps.

### Set Up and Deploy an Initial Application

Instruct your "developers" to do the following:

1. Click "New Project"
2. Enter a "Project Name" that container the developer's username (i.e. `demo-user-01-project`)
3. Search for the template named `php-supply-chain-app`
4. Fill in the *Git Repository URL* field with either:
  * Fork cakephp git repo (https://github.com/openshift/cakephp-ex.git), and replace the default git repo with your fork
  * Just fill in with the default repo (https://github.com/openshift/cakephp-ex.git). If you do this you will not be able to make a code change.
5. Click *Create*

Once each user clicks *Create*, explain what just happened. Basically the user instantiated a template in his or her own project. That template contained everything needed to build and deploy a basic app, including:

* A Build Config
* A Deployment Config
* A Service
* A Route

Show the users some of the basic functionality of the UI, for those who've never been hands on with OpenShift. Specifically click on the route link (upper right hand) and  show that this pulls up a webpage served by their running application.

### (Optional) Set up a WebHook in GitHub

At this point you could optionally walk your audience through setting up a GitHub webhook to automatically kick off a new build when a code change takes place. (Not covered here)

### (Optional) Make a Code Change

Now, you can, if you so choose, invite your "App Teams" that deployed a forked git repo to make a code change in their fork. This serves just to further prove to them that their app is their app, not something centrally managed by the operator (i.e. the _nothing up my sleeves_ portion of the magic trick)

NOTE: A simple change of the cakephp example app is to make a change to the HTML of home page at `app/View/Layouts/default.ctp`. You could change the heading, the background color, or something easily observable.

### Observe Your Running Application Pod Before it changes

Instruct your users to Open up a Terminal for their running PHP pod. (Click on the `N Pods` link in the Blue circle of your php app, then click on the name of the pod, and go to the *Terminal* tab)

Within the pod terminal run `ls /opt/app-root/etc` to observe the set of files that exist in that directory (we're about to add one via an Image Change, so it's important they note that it's not there at the beginning).

```
sh-4.2$ ls /opt/app-root/etc/
conf.d  httpdconf.sed  php.d  php.ini.template  scl_enable  security.txt
```

### Make an Image Change

At this point, it's time to put your Operator hat back on, and direct audience attention back to you. You are about to make a change to your corporation's standard PHP image and push it out to all running applications.

Feel free to come up with a colorful story about getting an angry call from your security team that the current apps are all out of compliance, and you need to patch immediately.

Clone your fork of the demo repo:
```
git clone https://github.com/redhat-cop/ocp-supply-chain-demo.git
cd ocp-supply-chain-demo
```

Add the following line to `Dockerfile`:
```
ADD security.txt /opt/app-root/etc/security.txt
```

Commit and push
```
git add Dockerfile
git commit -m"Security patch added"
git push
```

Now re-run the image build with the following:
```
oc start-build php-supply-chain-app -n openshift
```

Your Audience should almost immediately see a build kick off in each of their projects. Tell them that as soon as you built the updated image, it triggered a rebase of their application code on top of this new image, and that now each of their applications have been patched.

Now just to confirm:

6. Inspect the pod using the Terminal feature. Specifically check the `/opt/app-root/etc/` directory.
7. Sit back and watch while the Operator builds a new image
8. Inspect the pod again and see what's different
