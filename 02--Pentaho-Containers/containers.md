## <font color='red'>Pentaho 9.3 - Containers</font>  

The following pre-requisites configure the Pentaho Server and Data Integration 9.3.

Prerequisites for DockMaker:
* Docker
* A Docker Hub user account - pull required database images
* CURL command on host
* Java 8 or 11
* Pentaho EE license - PENTAHO_INSTALLED_LICENSE_PATH=/data/licenses/.installedLicenses.xml

---

<em>Pentaho EE License</em>

<font color='teal'>This section is for reference only. The license has been generated and  is located at: /data/license</font>

To install or update license files, from the command line, follow the steps below:  
``* Download the .lic file(s) you want to install.``  
``* Navigate to the /license-installer/ directory``   
``* Copy your .lic files to the /license-installer/ directory.``  
``* Run the license installation script.``  

Linux:  
Run install_license.sh with the install switch and the location and name of your .lic file as a parameter. You can specify multiple .lic files separated by spaces. Be sure to use backslashes (\) to escape any spaces in the path or file name.
```
 ./install_license.sh install Pentaho\ BI\ Platform\ Enterprise\ Edition.lic 
 ```
Windows:  
Run install_license.bat with the install switch and the location and name of your license file as a parameter. Be sure to use quotation marks (") to escape any spaces in the path or file name.
```
install_license.bat install "C:\dock-maker-9.3.0.0-428\license-installer"\Pentaho BI Platform Enterprise Edition.lic"
```
``* Repeat the previous step for all needed licenses.``
 
Set the PENTAHO_INSTALLED_LICENSE_PATH variable so that when you start Pentaho, the licenses can install.  

<font color='red'>If you do not set the variables, Pentaho will not start correctly.</font>

``open a terminal window and log in as root:``
```
sudo -i
```
``edit /etc/environment file:``
```
cd /etc/environment
nano /etc/environment
```
``add the following:``
```
export PENTAHO_INSTALLED_LICENSE_PATH=/data/licenses/.installedLicenses.xml
```
Note: You will need to log out and back in to set the variable. 

``verify that the PENTAHO_INSTALLED_LICENSE_PATH variable is set:``
```
env | grep PENTAHO_INSTALLED_LICENSE_PATH
```

---

<em>Install DockMaker</em>  

DockMaker is a command line tool used to create containers for Pentaho products:
* Pentaho Server BA/DI
* Pentaho Data Integration
* Carte server

<font color='teal'>DockMaker has been downloaded and copied to ~/dock-maker-9.3.0.0</font>

Install the package by completing these steps.

``run the install script:``
```
cd dock-maker-9.3.0.0
./install.sh
```
Note: a console window will appear.
``accept license:``

``edit the installation path to:``
```
/home/pentaho/dock-maker-9.3.0.0
```
Note: The Installation Progress window appears. Progress bars indicate the status of the installation. When the installation progress is complete, click Quit to exit the Unpack Wizard.

---

<em>DockMaker</em>

DockMaker is a command line tool for building and deploying Pentaho Containers. 
DockMaker directories: 

* <font color='teal'>artifactCache:</font> This folder serves as the default storage location for any artifacts that are downloaded or required to setup the image.  The location of this folder can be changed in the DockMaker.properties file.
* <font color='teal'>containers:</font> This has various files and templates that will be tapped when running the command line tool.
* <font color='teal'>generatedFiles:</font> This folder is created when the command line tool is executed.  It contains all the file necessary to create a docker image and use docker compose to bring up the containers.
* <font color='teal'>lib:</font> Dependent libraries.

---

<em>Pentaho EE 9.3.0.0-428 BA / DI Containers</em>

``build Pentaho Server EE 9.3.0.0:428:``
```
cd dock-maker-9.3.0.0-428
./DockMaker.sh -V 9.3.0.0/428/ee -A paz,pdd,pir -U --EULA_ACCEPT=true
```
Note: if you wish to automate the build and deploy add the flag: -X

  > log into the server: http://localhost:8081/pentaho/Login 
  
Note: Default port 8081.  This can be changed with the -p parameter.

``to stop containers:``
```
docker compose -f generatedFiles/docker-compose.yml stop
```
``restart containers:``
```
docker compose -f generatedFiles/docker-compose.yml start
```

---

<em>Pentaho EE 9.3.0.0-428 PDI / Carte Containers</em>

The DockMaker command tool supports three different volumes used by the containers:

Override Files Volume
The Override Volume contains all the changes that must be made to the basic artifact files to create the configuration desired.  This volume binds the generatedFiles\fileOverride folder on the host with the /docker-entrypoint-init folder on the container.  When the container is started, the files present in this folder will overwrite the files/folders in the /opt/pentaho/data-integration or /opt/pentaho/pentaho-server folder, obviously depending on the Type of container.  
If you need any additional files, drivers, etc to be placed in the application’s folder just declare the proper path, in the fileOverride folder.
Be aware, however, any changes made here will be lost if the command tool is ever executed again.  That is why the command line normally gives you the docker commands without executing them.  In many cases the user will need to make more changes to the templates provided.  If you want to make sure you do not lose the files that were generated, (or manually changed), rename the generatedFiles folder to something else, or copy the folder entirely to another place.  It will still work though some paths may have to be adjusted in the commands listed in this document.

Metastore Volume
This volume is provided by the user of tool by including the -M or –metastore parameter.  It is intended to allow the user to create a bi-directional bind on a metastore folder.  It binds the folder defined in the -M parameter with the /home/pentaho folder on the container.  You probably don’t want to link it directly to your own metastore because that breaks the container contract of not changing host environment, but you can make a copy of your metastore and bind to that.  To do this, perform the following steps:
1.	Create an arbitrary folder on your host file system to serve as the shared folder.  We’ll use d:\metastore which demonstrates Windows drive usage as well.
2.	Copy the “.kettle” and “.pentaho” folders from your home folder to the “d:/metastore” folder, or whatever names you used.  This is enough to share the metastore.
3.	Copy any other files you need to process your use case.  The d:/metastore folder will be bound to the /home/pentaho folder on the container.
4.	When you generate the files with DockMaker command, it should contain “-M d:/metastore”.

Database Volume
The database volume is only created when a Pentaho Server container is run.  This volume contains all the database tables associated with the server repository, quartz scheduler, and log tables.  When the database container is started for the first time, it will run DDL provided in generatedFiles that will define and possibly populate the tables needed.  The container’s form is essentially dictated by the database provider and is included here for completeness.



``build Pentaho Data Integration 9.3.0.0:``
```
cd dock-maker-9.3.0.0-428
./DockMaker.sh -V 9.3.0.0/428/ee -T pdi -p 8082 -U --EULA_ACCEPT=true
```
Note: When executed with -X, the container will be built and a test.ktr transformation will be run on the container.  

``build a Carte server 9.3.0.0:``
```
cd dock-maker-9.3.0.0-428
./DockMaker.sh -V 9.3.0.0/428/ee -T carte -p 8083 -U --EULA_ACCEPT=true
```
user: cluster  
password: cluster

---