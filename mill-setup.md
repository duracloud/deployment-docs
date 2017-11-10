# DuraCloud Mill Setup
The mill can be deployed (and updated) using a simple python commandline tool we created for this purpose.
It works by reading several configuration files which it then uses to create  the aws resources  on which the mill 
depends (ie queues, alarms, launch  configs, and autoscale groups).  

## Deploy the mill for the first time
1. First clone the mill deploy tool repository: 
    ```$xslt
    git clone https://github.com/duracloud/mill-deploy.git
    ```
1. Next create your mill config directory where you will keep your mill config files.
```$xslt
# create a local directory for your configs
mkdir -p mill-config/production
```
1. Then copy all the files into the above dir from the latest tagged release of mill-deploy/sample-config:

1. Edit each file, filling in the appropriate values based on the values generated in the 
[initial AWS setup](aws-setup.md).

1. Follow the instructions for deploying the mill in the mill-deploy/README.

## Deploy the duplication policy editor
