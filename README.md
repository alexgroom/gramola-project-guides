Modern Mobile App Dev Workshop [![Build Status](https://travis-ci.org/openshift-labs/cloud-native-guides.svg?branch=ocp-3.10)](https://travis-ci.org/openshift-labs/cloud-native-guides)
===
This one day hands-on cloud-native workshop provides developers with and introduction to building cloud-native mobile applications using OpenShift, Spring Boot, NodeJS and Flutter.

The aim of this workshop is to walk you through the process of creating a pretty basic mobile app based on Flutter that leverages some microservices to show a list of musical events and a timeline-like comments related to the events.

Agenda
===
* Introduction to Cloud-Native apps 
* Introduction to and Flutter
* Building a CRUD service with Spring Boot to manage events
* Building a service to store/serve images with NodeJS
* Building our mobile application with Flutter
* Adding a new feature: a timeline-like service with NodeJS
* Adding a new module to our mobile app for the new feature
* Continuous Delivery 
* Debugging Services
* Telepresence?

Using RHPDS Workshop
===

Order an Workshops | OpenShift Workshop environment from RHPDS service Catalog

Name thw environment with a city, user SFDC code 000000 if no specific customer. Choose a region as appropriate.

The environment will take about 30 minutes to be created and look like this example:

  ```
  Bastion: bastion.NAME-XXXX.openshiftworkshop.com
  Console: https://master.NAME-XXX.openshiftworkshop.com
  ```


Follow the email instructions to access the workshop admin via SSH. Download the workshop PEM as required

  ```
  $ ssh -i ~/.ssh/ocp-workshop.pem ec2-user@bastion.NAME-XXXX.openshiftworkshop.com
  ```

For console access, connect to the bastion host and add Cluster admin to a given user eg user3, then use this acount to access the web console.

  ```
  $ sudo -i
  $ oadm policy add-cluster-role-to-user cluster-admin user3
  $ oc new-project lab-infra
  ```

Now login into the openshift console and (user3/openshift) and create the lab-infra project contents and all the associated applications and guides from the service catalog entering all the configuration information. Choose the "Cloud-Native Workshop Installer" from the Catalog.

The typical responses are:

  ```
  master url: https://master.NAME-XXX.openshiftworkshop.com
  user password: openshift
  Gogs services etc: adminuser/adminpwd
  ```


Manually deploy the lab instructions as described below, either in the Openshift cluster or locally via docker.

Or simply edit the existing "guides-che" application and change the following environment variables to point to the project content. the advantage here is that the existing URLs are already populated and pointing at the services just installed.

  ```
  CONTENT_URL_PREFIX = https://raw.githubusercontent.com/alexgroom/gramola-project-guides/master
  WORKSHOPS_URLS = https://raw.githubusercontent.com/alexgroom/gramola-project-guides/master/_cloud-native-mobile-workshop-che.yml
  ```

Install Workshop Infrastructure
===

An [APB](https://hub.docker.com/r/openshiftapb/cloudnative-workshop-apb) is provided for 
deploying the Cloud-Native Workshop infra (lab instructions, Nexus, Gogs, Eclipse Che, etc) in a project 
on an OpenShift cluster via the service catalog. In order to add this APB to the OpenShift service catalog, log in 
as cluster admin and perform the following in the `openshift-ansible-service-broker` project :

1. Edit the `broker-config` configmap and add this snippet right after `registry:`:

  ```
    - name: dh
      type: dockerhub
      org: openshiftapb
      tag: ocp-3.10
      white_list: [.*-apb$]
  ```

2. Redeploy the `asb` deployment

You can [read more in the docs](https://docs.openshift.com/container-platform/3.10/install_config/oab_broker_configuration.html#oab-config-registry-dockerhub) 
on how to configure the service catalog.

Note that if you are using the _OpenShift Workshop_ in RHPDS, this APB is already available in your service catalog.


Run the APB directly in a pod on OpenShift to install the workshop infra:

```
oc login
oc new-project lab-infra
oc run apb --restart=Never --image="cvincens/mobile-cloudnative-workshop-apb:latest" \
    -- provision -vvv -e namespace=$(oc project -q) -e openshift_token=$(oc whoami -t)
```

Install Workshop Directly
===
If you have Ansible installed locally ie. on your Mac or Linux laptop, you can also run the Ansilbe playbooks directly on your machine:

> NOTE:
Python3 is a dependancy along with jmespath and the openshift API. The default on most systems including RHEL and Mac OS is Python => Python2. Changing this default is undesirable since it is alsmost a system dependancy so install Python3 in a new location and then update its runtime.

```
 sudo pip3 install openshift
 sudo ansible-galaxy install -r requirements-travis.yml
 sudo pip3 install jmespath
 sudo pip install jmespath
```

```
oc login
oc new-project lab-infra

ansible-playbook -vvv playbooks/provision.yml \
       -e namespace=$(oc project -q) \
       -e openshift_token=$(oc whoami -t) \
       -e openshift_master_url=$(oc whoami --show-server) \
       -e 'ansible_python_interpreter=/usr/local/bin/python3' 
``` 

On a system where both Python2 and Python3 are present, Ansible will need to be forced to use Python3 by specifiying its location since it is a dependancy of the openshift API.


Local Lab Instructions
===
```
$ docker run -it -p 8080:8080 \
    -v $(pwd):/app-data\
    -e CONTENT_URL_PREFIX="file:///app-data" \
    -e WORKSHOPS_URLS="file:///app-data/_cloud-native-mobile-workshop-che.yml" \
    osevg/workshopper:latest
```
