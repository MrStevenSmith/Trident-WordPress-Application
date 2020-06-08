# Trident Persistent Storage - Simple WordPress Application

## Exercise 2 - Answer

To import our volumes we will use the command below.

```
tridentctl import volume <backendName> <volumeName> -f <path-to-pvc-file>
```

Firstly create your 2 yaml files as per the instructions on the [Trident Documentation](https://netapp-trident.readthedocs.io/en/stable-v20.04/kubernetes/operations/tasks/volumes.html#importing-a-volume) page, as saved above.

MySQL Volume yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-persistent-storage
  namespace: default
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: storage-class-nas
```

WordPress Volume yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-persistent-storage
  namespace: default
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: storage-class-nas
```

Once done we need to know the name of our backend.

```
[root@rhel3 ~]# tridentctl get backends -n trident
+---------------------+----------------+--------------------------------------+--------+---------+
|        NAME         | STORAGE DRIVER |                 UUID                 | STATE  | VOLUMES |
+---------------------+----------------+--------------------------------------+--------+---------+
| BackendForSolidFire | solidfire-san  | d9d6bef6-eef9-4ff0-b5c8-c69d048b739e | online |       0 |
| BackendForNAS       | ontap-nas      | e098abb8-8e16-4b4f-a4bc-a6c9557b39b1 | online |       2 |
+---------------------+----------------+--------------------------------------+--------+---------+

[root@rhel3 ~]#
```

We can now import our mysql Persistent Volume.

```
[root@rhel3 ~]# tridentctl import volume -n trident BackendForNAS trident_pvc_mysql -f reconnect-mysql-pvc.yml
+------------------------------------------+---------+-------------------+----------+--------------------------------------+--------+---------+
|                   NAME                   |  SIZE   |   STORAGE CLASS   | PROTOCOL |             BACKEND UUID             | STATE  | MANAGED |
+------------------------------------------+---------+-------------------+----------+--------------------------------------+--------+---------+
| pvc-e2b4ac2d-f033-487e-8f60-b83c6550ef73 | 1.0 GiB | storage-class-nas | file     | e098abb8-8e16-4b4f-a4bc-a6c9557b39b1 | online | true    |
+------------------------------------------+---------+-------------------+----------+--------------------------------------+--------+---------+

[root@rhel3 ~]#
```

Next we can import our WordPress Persistent Volume.

```
[root@rhel3 ~]# tridentctl import volume -n trident BackendForNAS trident_pvc_wordpress -f reconnect-wp-pvc.yml
+------------------------------------------+---------+-------------------+----------+--------------------------------------+--------+---------+
|                   NAME                   |  SIZE   |   STORAGE CLASS   | PROTOCOL |             BACKEND UUID             | STATE  | MANAGED |
+------------------------------------------+---------+-------------------+----------+--------------------------------------+--------+---------+
| pvc-e8c18413-a4cf-4e11-9b7c-2592b523051f | 1.0 GiB | storage-class-nas | file     | e098abb8-8e16-4b4f-a4bc-a6c9557b39b1 | online | true    |
+------------------------------------------+---------+-------------------+----------+--------------------------------------+--------+---------+

[root@rhel3 ~]#
```

We can now see our imported volumes as Persistent Volume Claims and Persistent Volumes.

```
[root@rhel3 ~]# kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS        REASON   AGE
persistentvolume/pvc-e2b4ac2d-f033-487e-8f60-b83c6550ef73   1Gi        RWX            Delete           Bound    default/mysql-persistent-storage       storage-class-nas            62m
persistentvolume/pvc-e8c18413-a4cf-4e11-9b7c-2592b523051f   1Gi        RWX            Delete           Bound    default/wordpress-persistent-storage   storage-class-nas            60m

NAME                                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
persistentvolumeclaim/mysql-persistent-storage       Bound    pvc-e2b4ac2d-f033-487e-8f60-b83c6550ef73   1Gi        RWX            storage-class-nas   62m
persistentvolumeclaim/wordpress-persistent-storage   Bound    pvc-e8c18413-a4cf-4e11-9b7c-2592b523051f   1Gi        RWX            storage-class-nas   60m

[root@rhel3 ~]#
```

From this point we can rebuild the secret, 2 deployments and 2 services.

```
[root@rhel3 ~]# kubectl create -f mysql-secret.yml
secret/mysql-password created

[root@rhel3 ~]# kubectl create -f mysql-dep.yml
deployment.apps/mysql created

[root@rhel3 ~]# kubectl create -f mysql-svc.yml
service/mysql-svc created

[root@rhel3 ~]# kubectl create -f wp-dep.yml
deployment.apps/wordpress created

[root@rhel3 ~]# kubectl create -f wp-svc.yml
service/wordpress-svc created

[root@rhel3 ~]#
```

### Test your WordPress application

Once you have re-imported the existing volumes create the rest of the applications required objects.  Once done try to access your application on http://192.168.0.63:31004

You should see your new WordPress application running and you should not be asked to carry out the initial configuration again.

If you do not try to troubleshoot and fix the issue.
<br />

## License

This project is licensed under the GNU GENERAL PUBLIC LICENSE - see the [LICENSE](https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/LICENSE) file for details
<br />
<br />

## Author

* **Steven Smith** - *Created the exercise* - [MrStevenSmith](https://github.com/MrStevenSmith)