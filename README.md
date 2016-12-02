# Raleigh Consulting Demo Content

## Instructor Setup

```
oc new-build https://github.com/etsauer/rdu-consulting-demo.git --name=rdu-php-demo --strategy=docker --docker-image=registry.access.redhat.com/rhscl/php-56-rhel7 -n openshift
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

