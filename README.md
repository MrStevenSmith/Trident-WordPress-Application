# Trident Persistent Storage - Simple WordPress Application

## Exercise 1

In this exercise you will be asked to create the yaml files to provision various Kubernetes objects to create a WordPress application using a MySQL database and NetApp Trident to automatically provision persistent volumes from a NetApp ONTAP based storage system.
<br />

**[Exercise 1](https://github.com/MrStevenSmith/Trident-WordPress-Application/tree/master/exercise%201)**

## Exercise 2

In this exercise you will delete all objects created in Exercise 1 retaining the NetApp volumes for re-use.  You will then be asked to create the yaml files to re-import your volumes into a fresh deployment of your WordPress application.

This exercise emulates the storage being SnapMirrored from your production site to a DR site and how you would re-import your application and it's data at the DR site.
<br />

**[Exercise 2](https://github.com/MrStevenSmith/Trident-WordPress-Application/tree/master/exercise%202)**

## Application Layout

The WordPress application will utilise persistent storage from a NetApp ONTAP based system and will use Trident to automatically provision the persistent storage.

The layout for the WordPress application will be as per below: -

<br />

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/Trident-WordPress.png">
</p>

<br />

The end result should be a WordPress application which uses persistent storage to save it's configuration information.

<p align="center">
<img src="https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/images/site.png">
</p>

<br />

## License

This project is licensed under the GNU GENERAL PUBLIC LICENSE - see the [LICENSE](https://github.com/MrStevenSmith/Trident-WordPress-Application/blob/master/LICENSE) file for details
<br />
<br />
## Author

* **Steven Smith** - *Created the exercise* - [MrStevenSmith](https://github.com/MrStevenSmith)