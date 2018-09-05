# Simple Backup Maria DB/MySQL DB

## Overview

The maria DB backup uses the 'scheduledjob' functionality of OpenShift to start a pod in regular intervals. The pod dumps the database to a persistent volume and exits.

This tool uses an existing container (RedHat Maria DB container) and overrides its command. There is no need to build a container.

## How to deploy the maria DB backup pod

### Prequisits

* Log in using `oc login`
* Switch to the right project using `oc project <yourproject>`

### Create a pv for the backup

I would recommend to use the GUI for this part.

### Take a look at the parameters of the template

```bash
oc process --parameters -f mariadb-backup-template.yaml

```

**The following parameters are mandatory:**

* DATABASE_USER
* DATABASE_PASSWORD
* DATABASE_HOST
* DATABASE_PORT
* DATABASE_NAME
* DATABASE_BACKUP_VOLUME_CLAIM

### Create the scheduled Job

```bash
oc process -f mariadb-backup-template.yml DATABASE_USER=<dbuser> DATABASE_PASSWORD=<dbpassword> DATABASE_HOST=<dbhost> DATABASE_PORT=<dbport> DATABASE_NAME=<dbname> DATABASE_BACKUP_VOLUME_CLAIM=<pvc-claim-name> ICINGA_USERNAME=<icinga-user> ICINGA_PASSWORD=<icinga-password> ICINGA_SERVICE_URL=<icinga-service-url> | oc create -f -
```

You can also store the template in the project using and `oc process` afterwards

```bash
oc create -f mariadb-backup-template.yml
oc process mariadb-backup-template DATABASE_USER=<dbuser> DATABASE_PASSWORD=<dbpassword> ... | oc create -f -
```

To check if the scheduled job is present:

````bash
oc get scheduledjobs
````

### Housekeeping

To disable the backup, you can simply remove the scheduledjob:

````bash
oc delete scheduledjob mariadb-backup
````

To restore the backup you start a backup pod (e.g. in debug mode) connect to the pod and use:

````bash
oc rsh mariadb-backup-[xyz]-debug
mysql -u<db-user> -p<db-user-password> -h<host> < <path-to-backupfile> (the backupfile has to be unpacked)
````

### Monitoring

In this template an passive incinga service is monitoring the backup. Should the bash script (responsible for the backup) throw an error at any point during the executing the notification will not be sent to icinga. The passive service checks periodically if a notification was received. If not the service will update its status. The following parameters are used for the monitoring:

* ICINGA_USERNAME
* ICINGA_PASSWORD
* ICINGA_SERVICE_URL

If you don't use icinga/use another monitoring service you can remove/update the corresponding parts of the template.
