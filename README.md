# Simple Backup Maria DB/MySQL DB 

## Overview

The mariaDB backup uses the 'scheduledjob' functionality of OpenShift to start a pod in regular intervals. The pod dumps the database to a persistent volume and exits.

This tool uses an existing container (RedHat MariaDB container) and overrides its command. There is no need to build a container.

## How to deploy the mariaDB Backup

To set up the backup:
+ Log in using `oc login`
+ Switch to the right project using `oc project <yourproject>`

To get a list of parameters, use

```
oc process --parameters -f mariadb-backup-template.yaml

```

**The following parameters are mandatory:**

+ MARIADB_PASSWORD
+ MARIADB_SERVICE_HOST
+ MARIADB_BACKUP_DATABASE
+ MARIADB_BACKUP_VOLUME_CLAIM

To create the scheduledjob, use
```
oc process -f mariadb-backup-template.yml MARIADB_USER=<user> MARIADB_PASSWORD=<password> MARIADB_SERVICE_HOST=<host> MARIADB_BACKUP_DATABASE=<database> MARIADB_BACKUP_VOLUME_CLAIM=<pv-claim> MARIADB_BACKUP_SCHEDULE="<cronjob-expression>" <more parameters> | oc create -f - 
```

You can also store the template in the OpenShift project using
```
oc create -f mariadb-backup-template.yaml
```
After you did that, you can use
```
oc process mariadb-backup-template MARIADB_PASSWORD=<password> <more parameters> | oc create -f -
```

Create a pv for the backup
````
oc volume mariadb-backup --add --name "mariadb-backup" -t persistentVolumeClaim --claim-name "backup-claim" --claim-size=1Gi
````

To check if the scheduled job is present:
````
oc get scheduledjobs
````

To disable the backup, you can simply remove the scheduledjob:
````
oc delete scheduledjob mariadb-backup
````

To restore the backup you start a backup pod (e.g. in debug mode) connect to the pod and use:
````
oc rsh mariadb-backup-[xyz]-debug

mysql -u<db-user> -p<db-user-password> -h<host> < <path-to-backupfile>.sql (the backupfile has to be unzipped)
```` 
