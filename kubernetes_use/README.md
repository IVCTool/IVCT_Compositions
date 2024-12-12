# How to get and use  IVCT-Images in kubernetes

To run the Interoperability, Verification and Certification Tool (IVCT)   in a container environment, you need to load different images and start several containers created from them.


### IVCT setup in a Docker application
Controlled by Docker Compose Files .env and docker-compose-*.yml
docker-Compose  fetch the necessary images, compiles the containers  and start them.

The docker-compose-*.yml contains various sections for processing the different components of the IVCT environment.


### IVCT setup in a Kubernetes surrounding
In the Kubernetes environment, *.yaml files can be used to fulfill these tasks.

Here too, several tasks can be combined in a *.yaml file.  
To be able to better control and test the various settings of the containers / jobs, we have used individual scripts for each task.

Success also depends on the order in which the individual components are started.


### Prerequisite:  Data common directory

First of all, it must be ensured that there is a data directory to which various deployments have read and write access.
One possibility is to use a directory provided by an NFS server.

The scripts 01* and 02* establish a connection to an NFS share, 
and make it available for other scripts 

01_nfs-pv.yaml  
02_nfs-pvc.yaml

### Execution of images with only one task
To provide information and necessary files for IVCT tests,
containers from some images  are started only once.
They copy files to the shared data area.

In the Kubernetes application, these executions are called jobs

#### 11_runtime-config_job.yaml  
ContainerName:  runtime-config-job  --  image:  ivct/runtime-config:4.1.0

Should be executed relatively early in order to create the necessary directory structures in the shared directory and transfer various data to it.
e.g. the Directories  ( with content)  Badges IVCTsut  TestSuites  + Datei IVCT.properties

MountPoint: /runtimeconfig   (name can be freely chosen,   to the NFS-Volume)

important is here: 
 [Command]   and  [Arguments] :  
 -r /root/conf/TestSuites , -r /root/conf/Badges , -r /root/conf/IVCTsut , /root/conf/IVCT.properties   /runtimeconfig



