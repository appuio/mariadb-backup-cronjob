# Simple Backup Maria DB/MySQL DB 

## Overview

The mariaDB backup uses the 'scheduledjob' functionality of OpenShift to start a pod in regular intervals. The pod dumps the database to a persistent volume and exits.

This tool uses an existing container (RedHat MariaDB container) and overrides its command. There is no need to build a container.

## How to deploy the mariaDB Backup

To set up the backup:
+ Log in using `oc login`
+ Switch to the right project using `oc project <yourproject>`

To create the scheduledjob, use
```
oc process -f mariadb-backup-template.yaml MARIADB_ADMIN_PASSWORD=passw0rd <more parameters> | oc create -f -
```

To get a list of parameters, use

```
oc process --parameters -f mariadb-backup-template.yaml

```

**The following parameters are mandatory:**

+ MARIADB_ADMIN_PASSWORD
+ MARIADB_SERVICE_HOST


You can also store the template in the OpenShift project using
```
oc create -f mariadb-backup-template.yaml
```
After you did that, you can use
```
oc process mariadb-backup-template MARIADB_ADMIN_PASSWORD=passw0rd <more parameters> | oc create -f -
```

To check if the scheduled job is present:
````
oc get scheduledjobs
````

To disable the backup, you can simply remove the scheduledjob:
```
oc delete scheduledjob mariadb-backup
```

To restore the backup you start a backup pod (e.g. in debug mode) connect to the pod and use:
````
oc rsh mariadb-backup-[xyz]-debug

mysql -uroot -p$MARIADB_ADMIN_PASSWORD -h$MARIADB_SERVICE_HOST < mariadb-backup/backupfile (unzipped)
```` 