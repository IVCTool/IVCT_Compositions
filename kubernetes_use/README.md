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
Variables:  No further variables necessary  
MountPoint: /runtimeconfig    ( to the NFS-Volume, name can be freely chosen)  
important is here: 
 [Command]   and  [Arguments] :  
 -r /root/conf/TestSuites , -r /root/conf/Badges , -r /root/conf/IVCTsut , /root/conf/IVCT.properties   /runtimeconfig

#### IVCT.properties
Now the file IVCT.properties should be located in " /runtimeconfig", in the shared directory, here an NFS-volume.
This will later be read by other applications for their start.
Values of the necessary variables that differ from the standard can be entered here.


####  12_ts-helloworld-job.yaml
JobName:  ts-helloworld-job    image: ivct/ts-helloworld:2.1.3-SNAPSHOT  
Variables:  No further variables necessary  
MountPoint: /runtimeconfig   ( to the NFS-Volume , name can be freely chosen)  
Command:    cp    -r /root/conf/TestSuites/TS_HelloWorld-2.1.3-SNAPSHOT /runtimeconfig/TestSuites

####  13_ts-hla-encoding-rules_job.yaml
Name: ts-hla-encoding-rules      image: ivct/ts-hla-encoding-rules:2.1.2-SNAPSHOT  
MountPoint:     /runtimeconfig  
[Command]  [Arguments]
cp   -r /root/conf/TestSuites/TS_HLA_EncodingRulesTester-2.1.2-SNAPSHOT  /runtimeconfig/TestSuites


#### 14_ts-hla-cs-verification_job
Name:  ts-hls-cs-verification   image: ivct/ts-hla-cs-verification:2.1.2-SNAPSHOT  
MountPoint:     /runtimeconfig  
Command]  [Arguments]
cp            -r /root/conf/TestSuites/TS_CS_Verification-2.1.2-SNAPSHOT /runtimeconfig/TestSuites


#### 15_ts-hla-service_job  
Name:  ts-hla-services  image:   ivct/ts-hla-services:2.1.2-SNAPSHOT  
MountPoint:     /runtimeconfig  
Command]  [Arguments]
cp            -r  /root/conf/TestSuites/TS_HLA_Services-2.1.2-SNAPSHOT /runtimeconfig/TestSuites


#### 16_ts-hla-object_job
Name: ts-hla-object    image:   ivct/ts-hla-object:2.1.2-SNAPSHOT  
MountPoint:     /runtimeconfig  
Command]  [Arguments]
cp            -r  /root/conf/TestSuites/TS_HLA_Object-2.1.2-SNAPSHOT /runtimeconfig/TestSuites


#### 17_ts-hla-declaration_job
Name: ts-hla-declaration   image:   ivct/ts-hla-declaration:2.1.2-SNAPSHOT  
MountPoint:     /runtimeconfig  
Command]  [Arguments]
cp            -r  /root/conf/TestSuites/TS_HLA_Declaration-2.1.2-SNAPSHOT   /runtimeconfig/TestSuites


####  18_ts-designator_job
 Name: ts-designator   image:  ivct/ts-designator:1.0.2-SNAPSHOT  
MountPoint:     /runtimeconfig  
[Command]  [Arguments]
cp                 -r /root/conf/TestSuites/TS_Designator-1.0.2-SNAPSHOT /runtimeconfig/TestSuites

### Execution of images for deployments and services

####  18_ts-designator_job
 Name: ts-designator   image:  ivct/ts-designator:1.0.2-SNAPSHOT  
MountPoint:     /runtimeconfig  
[Command]  [Arguments]
cp                 -r /root/conf/TestSuites/TS_Designator-1.0.2-SNAPSHOT /runtimeconfig/TestSuites

### Execution of images for deployments and services

####  31_activemq_deployment  
Name:   activemq     image: rmohr/activemq:5.14.5-alpine  
Variables: no variables necessary but we use for test purposes IVCT_HOME ,  IVCT_CONF  
Ports:  activemq  61616  ,  activemq-web 8161     TCP  
MountPoint:  /root/conf  ( not  necessary but for  test purposes /root/conf to the NFS-Volume ) 

#### 32_activemq_service  
Name:  activemqsrvc     app: activemq  
Ports:  activemq "61616"   61616     "8161"   8161

#### 33_logsink_deployment  
Name: logsink               image:  ivct/logsink:4.1.0  
Variables: IVCT_HOME  /root/conf                     IVCT_CONF   /root/conf/IVCT.properties  
                 ACTIVEMQ_HOST:  activemq           ACTIVEMQ_PORT:   61616  
MountPoint:  /root/conf  
 &nbsp;  &nbsp;  &nbsp; &nbsp;  &nbsp;  &nbsp;  &nbsp;  &nbsp; /logs  

( The logs are “in" the logsink container in /logs/LogSink.log  
to see them in the shared NFS volume we add a mountpoint: /logs with SubPath logs )

It may be necessary to specify the IP of the running activemq in the ACTIVEMQ_HOST variable:  
       e.g. ACTIVEMQ_HOST: &nbsp;  &nbsp;  &nbsp; 10.244.0.62

 #### 36_gui_deployment
Name:  gui    &nbsp;  &nbsp;  &nbsp;  image:   ivct/gui:4.1.0 
Variables:  
&nbsp;  &nbsp;  &nbsp; ACTIVEMQ_HOST:  &nbsp;   activemq    ( or IP of activemq deployment)  
&nbsp;  &nbsp;  &nbsp; ACTIVEMQ_PORT:   &nbsp;  "61616"  
&nbsp;  &nbsp;  &nbsp; IVCT_HOME:   &nbsp;   /root/conf  
&nbsp;  &nbsp;  &nbsp; IVCT_CONF:   &nbsp;    /root/conf/IVCT.properties  
&nbsp;  &nbsp;  &nbsp; PRTI1516E_HOME:  &nbsp;  /root/conf/prti1516e  
&nbsp;  &nbsp;  &nbsp; LRC_HOME:    &nbsp;    /root/conf/prti1516e  
&nbsp;  &nbsp;  &nbsp; LRC_CLASSPATH:  &nbsp;   /root/conf/prti1516e/lib/prti1516e.jar  

It may be necessary to specify the IP of the running activemq in the ACTIVEMQ_HOST variable:  
       e.g. ACTIVEMQ_HOST: &nbsp;  &nbsp;  &nbsp; 10.244.0.62

Ports:  "ivctgui"   8080  
MountPoint:  /root/conf 


#### 37_gui_service        Service 
name: guisrvc  
Ports       type:    NodePort  
 &nbsp;  &nbsp;  &nbsp; &nbsp;  &nbsp;    ports:  -name:"gui-port"    port: 8080         
                                                
Rancher: Via the menu item Services you can see the services started for each namespace, 
    and can read out the URL to reach this service in the browser.  
Kubectl:   “minikube service list” shows the services provided  

### Reach the IVCT-GUI  

Rancher: Via the menu item Services you can see the services started for each namespace, and can read out the URL to reach this service in the browser.  
Kubectl:   “minikube service list” shows the services provided.


