# Trident Persistent Storage - Simple WordPress Application

## Exercise 1

In this exercise you will be asked to create the yaml files to provision various Kubernetes objects to create a WordPress application using a MySQL database and NetApp Trident to automatically provision persistent volumes from a NetApp ONTAP based storage system.
<br />
<br />

### Getting Started

To carry out this exercise you will need access to a Kubernetes cluster with Trident installed.  You will also need access to a NetApp ONTAP based storage system.

For NetApp employees and partners who have access to [NetApp LabOnDemand](https://labondemand.netapp.com/) this exercise can be carried out using the [Using Trident with Kubernetes and ONTAP v3.1](https://labondemand.netapp.com/lab/sl10556) lab.

### Instructions

Below you will find an image of a WordPress application.  You will also find a set of parameters which you need to include in the yaml to build your objects.

Using the sample templates on [Kubernetes.io](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pod-templates) or manually building your yaml files, you will need to define and create each object using the parameters specified.

If you are unable to complete any of the objects a yaml file can be found in the answers folder.

## Application Layout

The layout for the WordPress application will be as per below: -

<br />

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/Trident-WordPress.png">
</p>

<br />

### Kubernetes object parameters

1. Create a secret with the following parameters<br />
```
Secret Name = mysql-password
Secret = password: Pa55w0rd
```

2. Create a MySQL Persistent Volume Claim with the following parameters
```
Claim Name = mysql-persistent-storage
Storage Request = 1Gi
Access Mode = ReadWriteMany
Storage Class Name = storage-class-nas
```

3. Create a MySQL Deployment with the following parameters
```
Name = mysql
Label = app: mysql
Replicas = 1
Image = mysql:5.7
Volume Mount Name = mysql-persistent-storage
Volume Mount Path = /var/lib/mysql
Environment Variable Name = MYSQL_ROOT_PASSWORD = secret: mysql-password: password
Volume Name: mysql-persistent-storage
Volume Persistent Volume Claim Name: mysql-persistent-storage
```

4. Create a MySQL Service with the following parameters
```
Name = mysql-svc
Label = app: mysql-svc
Type = ClusterIP
Port Name: mysql-svc
Port = 3306
Selector = app: mysql
```

5. Create a WordPress Persistent Volume Claim with the following parameters
```
Claim Name = wordpress-persistent-storage
Storage Request = 1Gi
Access modes = ReadWriteMany
Storage Class Name = storage-class-nas
```

6. Create a WordPress Deployment with the following parameters
```
Name = wordpress
Labels = app: wordpress
Replicas = 2
Image = wordpress
Volume Mount Name = wordpress-persistent-storage
Volume Mount Path = /var/www/html
Environment Variable = MYSQL_ROOT_PASSWORD = secret: mysql-password: password
Environment Variable = WORDPRESS_DB_HOST = value: mysql-svc
Volume Name: wordpress-persistent-storage
Volume Persistent Volume Claim Name: wordpress-persistent-storage
```

7. Create a WordPress Service with the following parameters
```
Name = wordpress-svc
Label = app: wordpress-svc
Type = NodePort
Port Name = wordpress-svc
Port = 80
Node Port = 31004
Selector = app: wordpress
```

### Test your WordPress application

Once you have created the required objects try to access your application on http://192.168.0.63:31004

You should see your new WordPress application running and be able to configure it.  Firstly, select the correct language.

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/language.png">
</p>

On the Welcome Page enter the following details

```
Site Title: Test-Site
Username: User
Password: Pa55word
Confirm Password: Check
Email: User@Mail.com
```

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/welcome.png">
</p>

Once done click 'Log on' and enter your User credetials.

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/login.png">
</p>

You will now see your blank WordPress site.

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/test-site.png">
</p>
<br />

**[Home](https://github.com/MrStevenSmith/Trident-WordPress-Application)**

<br />

**[Exercise 2](https://github.com/MrStevenSmith/Trident-WordPress-Application/tree/master/exercise%202)**

## License

This project is licensed under the GNU GENERAL PUBLIC LICENSE - see the [LICENSE](https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/LICENSE) file for details
<br />
<br />
## Author

* **Steven Smith** - *Created the exercise* - [MrStevenSmith](https://github.com/MrStevenSmith)