# Trident Persistent Storage - Simple WordPress Application

## Exercise 2

In this exercise you will delete all objects created in Exercise 1 retaining the NetApp volumes for re-use.  You will then be asked to create the yaml files to re-import your volumes into a fresh deployment of your WordPress application.
<br />
<br />

### Getting Started

To carry out this exercise you will need access to a Kubernetes cluster with Trident installed.  You will also need access to a NetApp ONTAP based storage system.

You will also need to have carried out Exercise 1 and have a working WordPress application.

### Instructions

Below you will find the instructions to clone the volumes used in your DordPress application and then delete all of the objects you created in Exercise 1.  We will also collect all the information needed to re-import our volumes.

You will need to re-import your volumes using the instructions from the [Trident Documentation](https://netapp-trident.readthedocs.io/en/stable-v20.04/kubernetes/operations/tasks/volumes.html#importing-a-volume) and once done redeploy the rest of your WordPress application objects from your previously created yaml files.

## Application Layout

The layout for the WordPress application will be as per below: -

<br />

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/Trident-WordPress.png">
</p>

<br />

### Preparing environment

Firstly delete your 2 deployments.

```
[root@rhel3 ~]# kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
mysql       1/1     1            1           78m
wordpress   2/2     2            2           18m

[root@rhel3 ~]# kubectl delete deployment wordpress
deployment.extensions "wordpress" deleted

[root@rhel3 ~]# kubectl delete deployment mysql
deployment.extensions "mysql" deleted

[root@rhel3 ~]# kubectl get deployments
No resources found.

[root@rhel3 ~]# kubectl get pods
No resources found.

[root@rhel3 ~]#
```

Next, delete the 2 services.

```
[root@rhel3 ~]# kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        269d
mysql-svc       ClusterIP   10.111.185.53   <none>        3306/TCP       78m
wordpress-svc   NodePort    10.97.14.66     <none>        80:31004/TCP   76m

[root@rhel3 ~]# kubectl delete svc wordpress-svc
service "wordpress-svc" deleted

[root@rhel3 ~]# kubectl delete svc mysql-svc
service "mysql-svc" deleted

[root@rhel3 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   269d

[root@rhel3 ~]#
```

Now we can delete the secret.

```
[root@rhel3 ~]# kubectl get secret
NAME                    TYPE                                  DATA   AGE
default-token-dbq27     kubernetes.io/service-account-token   3      270d
mysql-password          Opaque                                1      23h
shared-bootstrap-data   Opaque                                1      236d

[root@rhel3 ~]#


[root@rhel3 ~]# kubectl delete secret mysql-password
secret "mysql-password" deleted

[root@rhel3 ~]# kubectl get secret
NAME                    TYPE                                  DATA   AGE
default-token-dbq27     kubernetes.io/service-account-token   3      270d
shared-bootstrap-data   Opaque                                1      236d

[root@rhel3 ~]#
```

Before we delete our Persistent Volume Claims we need to create a clone of the NetApp volumes to re-import.  These volumes could be volumes SnapMirrored to a remote system.

Firstly we need to be able to match the NetApp volume with the deployment using the volume.

```
[root@rhel3 ~]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
mysql-persistent-storage       Bound    pvc-f952eb16-388c-43ef-966b-5ee3c5fda052   1Gi        RWX            storage-class-nas   10m
wordpress-persistent-storage   Bound    pvc-775a2424-a333-48da-8e04-bd924f57207a   1Gi        RWX            storage-class-nas   10m

[root@rhel3 ~]#
```

This shows us that the volume ending da052 belongs to the mysql deployment and the volume ending 7207a belongs to the WordPress deployment.  Your volumes will be named differently.

SSH into the ONTAP cluster and list the Trident volumes.

```
cluster1::> vol show -volume trident*
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
svm1      trident_pvc_775a2424_a333_48da_8e04_bd924f57207a aggr1 online RW 1GB 972.7MB  5%
svm1      trident_pvc_f952eb16_388c_43ef_966b_5ee3c5fda052 aggr2 online RW 1GB 810.8MB  20%
2 entries were displayed.

cluster1::>
```

Create a clone of the mysql volume, which ends da052.  We will name the clone trident_pvc_mysql.

(Be sure to rename the parent-volume to match yours)

```
cluster1::> vol clone create -vserver svm1 -flexclone trident_pvc_mysql -type RW -parent-vserver svm1 -parent-volume trident_pvc_f952eb16_388c_43ef_966b_5ee3c5fda052 -junction-path /trident_pvc_mysql -space-guarantee none
[Job 571] Job succeeded: Successful

cluster1::>
```

Now create a clone of the WordPress volume, which ends 7207a.  We will name the clone trident_pvc_wordpress.

(Be sure to rename the parent-volume to match yours)

```
cluster1::> vol clone create -vserver svm1 -flexclone trident_pvc_wordpress -type RW -parent-vserver svm1 -parent-volume trident_pvc_775a2424_a333_48da_8e04_bd924f57207a -junction-path /trident_pvc_wordpress -space-guarantee none
[Job 575] Job succeeded: Successful

cluster1::>
```

We should now see our original volumes as well as our clones.

```
cluster1::> vol show -volume trident*
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
svm1      trident_pvc_775a2424_a333_48da_8e04_bd924f57207a aggr1 online RW 1GB 972.4MB  5%
svm1      trident_pvc_f952eb16_388c_43ef_966b_5ee3c5fda052 aggr2 online RW 1GB 810.6MB  20%
svm1      trident_pvc_mysql aggr2   online     RW          1GB    810.6MB   20%
svm1      trident_pvc_wordpress aggr1 online   RW          1GB    972.5MB    5%
4 entries were displayed.

cluster1::>
```

We can now split our clones.  After a few minutes the process should finish.

```
cluster1::> vol clone split start -vserver svm1 -flexclone trident_pvc_mysql

Warning: Are you sure you want to split clone volume trident_pvc_mysql in Vserver svm1 ? {y|n}: y
[Job 576] Job is queued: Split trident_pvc_mysql.

cluster1::> vol clone split start -vserver svm1 -flexclone trident_pvc_wordpress

Warning: Are you sure you want to split clone volume trident_pvc_wordpress in Vserver svm1 ? {y|n}: y
[Job 577] Job is queued: Split trident_pvc_wordpress.

cluster1::>


cluster1::> vol clone split show
This table is currently empty.

cluster1::>
```

Now we will delete the Persistent Volume Claims.  This will delete the Persistent Volume and the underlying NetApp volume.

```
[root@rhel3 ~]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
mysql-persistent-storage       Bound    pvc-f952eb16-388c-43ef-966b-5ee3c5fda052   1Gi        RWX            storage-class-nas   10m
wordpress-persistent-storage   Bound    pvc-775a2424-a333-48da-8e04-bd924f57207a   1Gi        RWX            storage-class-nas   10m

[root@rhel3 ~]# kubectl delete pvc mysql-persistent-storage
persistentvolumeclaim "mysql-persistent-storage" deleted

[root@rhel3 ~]# kubectl delete pvc wordpress-persistent-storage
persistentvolumeclaim "wordpress-persistent-storage" deleted

[root@rhel3 ~]# kubectl get pvc,pv
No resources found.

[root@rhel3 ~]#
```

We should also now see no volumes connected to Trident.

```
[root@rhel3 ~]# tridentctl get volumes -n trident
+------+------+---------------+----------+--------------+-------+---------+
| NAME | SIZE | STORAGE CLASS | PROTOCOL | BACKEND UUID | STATE | MANAGED |
+------+------+---------------+----------+--------------+-------+---------+
+------+------+---------------+----------+--------------+-------+---------+
[root@rhel3 ~]#
```

## Re-import your volumes and recreate your WordPress application

Re-import your volumes using the instructions from the [Trident Documentation](https://netapp-trident.readthedocs.io/en/stable-v20.04/kubernetes/operations/tasks/volumes.html#importing-a-volume) and once done redeploy the rest of your WordPress application objects from your previously created yaml files.

### Test your WordPress application

Once you have re-imported the existing volumes create the rest of the applications required objects.  Once done try to access your application on http://192.168.0.63:31004

You should see your new WordPress application running and you should not be asked to carry out the initial configuration again.

If you do not try to troubleshoot and fix the issue.  The answer can be found in the answer folder.
<br />

**[Home](https://github.com/MrStevenSmith/Trident-WordPress-Application)**

## License

This project is licensed under the GNU GENERAL PUBLIC LICENSE - see the [LICENSE](https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/LICENSE) file for details
<br />
<br />

## Author

* **Steven Smith** - *Created the exercise* - [MrStevenSmith](https://github.com/MrStevenSmith)