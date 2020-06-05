# Trident Persistent Storage - Simple WordPress Application

## Exercise 1 - Answer

### Create the MySQL Secret

Before we can create our Secret yaml file we need to get a base64 encoded version of our password.  To do this we use the following commands.

```
[root@rhel3 ~]# echo -n 'Pa55w0rd' | base64
UGE1NXcwcmQ=

[root@rhel3 ~]#
```

With this we can now build our Secret yaml file.

```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-password
data:
  password: UGE1NXcwcmQ=
```

We can now deploy and inspect our Secret.

```
[root@rhel3 ~]# kubectl create -f mysql-secret.yml
secret/mysql-password created

[root@rhel3 ~]# kubectl get secrets
NAME                    TYPE                                  DATA   AGE
default-token-dbq27     kubernetes.io/service-account-token   3      270d
mysql-password          Opaque                                1      28s
shared-bootstrap-data   Opaque                                1      236d

[root@rhel3 ~]# kubectl describe secret mysql-password
Name:         mysql-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes

[root@rhel3 ~]#
```

### Create the MySQL Persistent Volume Claim

Next we can build our MySQL Persistent Volume Claim yaml.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-persistent-storage
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: storage-class-nas
```

We can now deploy and inspect our MySQL Persistent Volume Claim.

```
[root@rhel3 ~]# kubectl create -f mysql-pvc.yml
persistentvolumeclaim/mysql-persistent-storage created

[root@rhel3 ~]# kubectl get pvc
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
mysql-persistent-storage   Bound    pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef   1Gi        RWX            storage-class-nas   27s

[root@rhel3 ~]# kubectl describe pvc mysql-persistent-storage
Name:          mysql-persistent-storage
Namespace:     default
StorageClass:  storage-class-nas
Status:        Bound
Volume:        pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: csi.trident.netapp.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Mounted By:    <none>
Events:
 "    "    "

[root@rhel3 ~]#
```

Trident should have automatically provisioned a volume on the NetApp and assigned this as a Persistent Volume.  You should see the NetApp volume name and the backend ID, which represents the ONTAP SVM being used.

```
[root@rhel3 ~]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                              STORAGECLASS        REASON   AGE
pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef   1Gi        RWX            Delete           Bound    default/mysql-persistent-storage   storage-class-nas            5m25s

[root@rhel3 ~]# kubectl describe pv pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef
Name:            pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: csi.trident.netapp.io
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    storage-class-nas
Status:          Bound
Claim:           default/mysql-persistent-storage
Reclaim Policy:  Delete
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            csi.trident.netapp.io
    VolumeHandle:      pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef
    ReadOnly:          false
    VolumeAttributes:      backendUUID=e098abb8-8e16-4b4f-a4bc-a6c9557b39b1
                           internalName=trident_pvc_11753d45_5c9a_4c3f_b5e4_76c2e5984fef
                           name=pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef
                           protocol=file
                           storage.kubernetes.io/csiProvisionerIdentity=1591235773069-8081-csi.trident.netapp.io
Events:                <none>

[root@rhel3 ~]#
```

### Create the MySQL Deployment

Next we can build our MySQL Deployment yaml.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        volumeMounts:
        - mountPath: "/var/lib/mysql"
          name: mysql-persistent-storage
        env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-password
                key: password
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-persistent-storage
```

We can now deploy and inspect our MySQL Deployment.

```
[root@rhel3 ~]# kubectl create -f mysql-dep.yml
deployment.apps/mysql created

[root@rhel3 ~]# kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
mysql   1/1     1            1           46s

[root@rhel3 ~]# kubectl describe deployment mysql
Name:                   mysql
Namespace:              default
CreationTimestamp:      Fri, 05 Jun 2020 13:52:42 +0000
Labels:                 app=mysql
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=mysql
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=mysql
  Containers:
   mysql:
    Image:      mysql:5.7
    Port:       <none>
    Host Port:  <none>
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-password'>  Optional: false
    Mounts:
      /var/lib/mysql from mysql-persistent-storage (rw)
  Volumes:
   mysql-persistent-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mysql-persistent-storage
    ReadOnly:   false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   mysql-86689778dd (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  63s   deployment-controller  Scaled up replica set mysql-86689778dd to 1
[root@rhel3 ~]#
```

We can see from the center section that the label has been applied, the image is mysql:5.7, the environment variable is set correctly from the Secret, the volume is mounted on the correct path and the volume is presented from the correct Persistent Volume Claim.

```
  Labels:  app=mysql
  Containers:
   mysql:
    Image:      mysql:5.7
    Port:       <none>
    Host Port:  <none>
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-password'>  Optional: false
    Mounts:
      /var/lib/mysql from mysql-persistent-storage (rw)
  Volumes:
   mysql-persistent-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mysql-persistent-storage
```

If we inspect our Persistent Volume Claim we should also see it is now mounted by our MySQL pod.

```
[root@rhel3 ~]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
mysql-persistent-storage       Bound    pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef   1Gi        RWX            storage-class-nas   42m
wordpress-persistent-storage   Bound    pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7   1Gi        RWX            storage-class-nas   13m

[root@rhel3 ~]# kubectl describe pvc mysql-persistent-storage
Name:          mysql-persistent-storage
Namespace:     default
StorageClass:  storage-class-nas
Status:        Bound
Volume:        pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: csi.trident.netapp.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Mounted By:    mysql-86689778dd-mxsmh
Events:

[root@rhel3 ~]#
```

### Create the MySQL Service

Next we can build our MySQL Service yaml.

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql-svc
  name: mysql-svc
spec:
  ports:
  - name: mysql-svc
    port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    app: mysql
  type: ClusterIP
```

We can now deploy and inspect our MySQL Service.

```
[root@rhel3 ~]# kubectl create -f mysql-svc.yml
service/mysql-svc created

[root@rhel3 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP    270d
mysql-svc    ClusterIP   10.100.134.215   <none>        3306/TCP   5s

[root@rhel3 ~]# kubectl describe svc mysql-svc
Name:              mysql-svc
Namespace:         default
Labels:            app=mysql-svc
Annotations:       <none>
Selector:          app=mysql
Type:              ClusterIP
IP:                10.100.134.215
Port:              mysql-svc  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.36.0.1:3306
Session Affinity:  None
Events:            <none>

[root@rhel3 ~]#
```

We can see our service has bound to 1 endpoint.  This should be the IP address of the pod running MySQL.

```
[root@rhel3 ~]# kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE    NOMINATED NODE   READINESS GATES
mysql-86689778dd-mxsmh   1/1     Running   0          14m   10.36.0.1   rhel1   <none>           <none>

[root@rhel3 ~]#
```

### Create the WordPress Persistent Volume Claim

Next we can build our WordPress Persistent Volume Claim yaml.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-persistent-storage
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: storage-class-nas
```

We can now deploy and inspect our WordPress Persistent Volume Claim.

```
[root@rhel3 ~]# kubectl create -f wp-pvc.yml
persistentvolumeclaim/wordpress-persistent-storage created

[root@rhel3 ~]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
mysql-persistent-storage       Bound    pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef   1Gi        RWX            storage-class-nas   29m
wordpress-persistent-storage   Bound    pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7   1Gi        RWX            storage-class-nas   14s

[root@rhel3 ~]# kubectl describe pvc wordpress-persistent-storage
Name:          wordpress-persistent-storage
Namespace:     default
StorageClass:  storage-class-nas
Status:        Bound
Volume:        pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: csi.trident.netapp.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Mounted By:    <none>
Events:
 "    "    "

[root@rhel3 ~]#
```

Trident should have automatically provisioned a volume on the NetApp and assigned this as a Persistent Volume.  You should see the NetApp volume name and the backend ID, which represents the ONTAP SVM being used.

```
[root@rhel3 ~]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS        REASON   AGE
pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef   1Gi        RWX            Delete           Bound    default/mysql-persistent-storage       storage-class-nas            31m
pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7   1Gi        RWX            Delete           Bound    default/wordpress-persistent-storage   storage-class-nas            2m45s

[root@rhel3 ~]# kubectl describe pv pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7
Name:            pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: csi.trident.netapp.io
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    storage-class-nas
Status:          Bound
Claim:           default/wordpress-persistent-storage
Reclaim Policy:  Delete
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            csi.trident.netapp.io
    VolumeHandle:      pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7
    ReadOnly:          false
    VolumeAttributes:      backendUUID=e098abb8-8e16-4b4f-a4bc-a6c9557b39b1
                           internalName=trident_pvc_c82ef336_16ae_48bc_80bf_2a58bc46c5c7
                           name=pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7
                           protocol=file
                           storage.kubernetes.io/csiProvisionerIdentity=1591235773069-8081-csi.trident.netapp.io
Events:                <none>

[root@rhel3 ~]#
```

### Create the WordPress Deployment

Next we can build our WordPress Deployment yaml.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress
        volumeMounts:
        - mountPath: "/var/www/html"
          name: wordpress-persistent-storage
        env:
          - name: WORDPRESS_DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-password
                key: password
          - name: WORDPRESS_DB_HOST
            value: mysql-svc
      volumes:
        - name: wordpress-persistent-storage
          persistentVolumeClaim:
            claimName: wordpress-persistent-storage
```

We can now deploy and inspect our WordPress Deployment.

```
[root@rhel3 ~]# kubectl create -f wp-dep.yml
deployment.apps/wordpress created

[root@rhel3 ~]# kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
mysql       1/1     1            1           25m
wordpress   0/2     2            0           13s

[root@rhel3 ~]# kubectl describe deployment wordpress
Name:                   wordpress
Namespace:              default
CreationTimestamp:      Fri, 05 Jun 2020 14:17:57 +0000
Labels:                 app=wordpress
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=wordpress
Replicas:               2 desired | 2 updated | 2 total | 1 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=wordpress
  Containers:
   wordpress:
    Image:      wordpress
    Port:       <none>
    Host Port:  <none>
    Environment:
      WORDPRESS_DB_PASSWORD:  <set to the key 'password' in secret 'mysql-password'>  Optional: false
      WORDPRESS_DB_HOST:      mysql-svc
    Mounts:
      /var/www/html from wordpress-persistent-storage (rw)
  Volumes:
   wordpress-persistent-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  wordpress-persistent-storage
    ReadOnly:   false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   wordpress-66cc68ccb5 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set wordpress-66cc68ccb5 to 2

[root@rhel3 ~]#
```

We can see from the center section that the label has been applied, the image is wordpress, the environment variable is set correctly from the Secret and the db host variable is set correctly, the volume is mounted on the correct path and the volume is presented from the correct Persistent Volume Claim.

```
  Labels:  app=wordpress
  Containers:
   wordpress:
    Image:      wordpress
    Port:       <none>
    Host Port:  <none>
    Environment:
      WORDPRESS_DB_PASSWORD:  <set to the key 'password' in secret 'mysql-password'>  Optional: false
      WORDPRESS_DB_HOST:      mysql-svc
    Mounts:
      /var/www/html from wordpress-persistent-storage (rw)
  Volumes:
   wordpress-persistent-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  wordpress-persistent-storage
```

If we inspect our Persistent Volume Claim we should also see it is now mounted by our WordPress pods.

```
[root@rhel3 ~]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
mysql-persistent-storage       Bound    pvc-11753d45-5c9a-4c3f-b5e4-76c2e5984fef   1Gi        RWX            storage-class-nas   39m
wordpress-persistent-storage   Bound    pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7   1Gi        RWX            storage-class-nas   10m

[root@rhel3 ~]# kubectl describe pvc wordpress-persistent-storage
Name:          wordpress-persistent-storage
Namespace:     default
StorageClass:  storage-class-nas
Status:        Bound
Volume:        pvc-c82ef336-16ae-48bc-80bf-2a58bc46c5c7
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: csi.trident.netapp.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Mounted By:    wordpress-66cc68ccb5-7rxxn
               wordpress-66cc68ccb5-ks4np
Events:
 "    "    "

[root@rhel3 ~]#
```

### Create the WordPress Service

Next we can build our WordPress Service yaml.

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: wordpress-svc
  name: wordpress-svc
spec:
  ports:
  - name: wordpress-svc
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 31004
  selector:
    app: wordpress
  type: NodePort

```

We can now deploy and inspect our WordPress Service.

```
[root@rhel3 ~]# kubectl create -f wp-svc.yml
service/wordpress-svc created

[root@rhel3 ~]# kubectl get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP        270d
mysql-svc       ClusterIP   10.100.134.215   <none>        3306/TCP       28m
wordpress-svc   NodePort    10.109.140.15    <none>        80:31004/TCP   6s

[root@rhel3 ~]# kubectl describe svc wordpress-svc
Name:                     wordpress-svc
Namespace:                default
Labels:                   app=wordpress
Annotations:              <none>
Selector:                 app=wordpress
Type:                     NodePort
IP:                       10.109.140.15
Port:                     wordpress  80/TCP
TargetPort:               80/TCP
NodePort:                 wordpress  31004/TCP
Endpoints:                10.36.0.2:80,10.44.0.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

[root@rhel3 ~]#
```

Now we can see the service listening on port 31004 and is a Node Port.

We can also see our service has bound to 2 endpoints.  These should be the IP addresses of the pods running WordPress.

```
[root@rhel3 ~]# kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP          NODE    NOMINATED NODE   READINESS GATES
mysql-86689778dd-mxsmh       1/1     Running   0          41m   10.36.0.1   rhel1   <none>           <none>
wordpress-66cc68ccb5-7rxxn   1/1     Running   0          16m   10.44.0.2   rhel2   <none>           <none>
wordpress-66cc68ccb5-ks4np   1/1     Running   2          16m   10.36.0.2   rhel1   <none>           <none>

[root@rhel3 ~]#
```

### Test your WordPress application

Once you have created the required objects try to access your application on http://192.168.0.63:31004

You should see your new WordPress application running and be able to configure it.  Do this as per the instructions on the [main Exercise 1 page.](https://github.com/MrStevenSmith/Trident-WordPress-Application/tree/master/exercise%201)

<br />

## License

This project is licensed under the GNU GENERAL PUBLIC LICENSE - see the [LICENSE](https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/LICENSE) file for details
<br />
<br />

## Author

* **Steven Smith** - *Created the exercise* - [MrStevenSmith](https://github.com/MrStevenSmith)