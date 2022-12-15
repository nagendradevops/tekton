## Introduction:

This document will guide you with step by step work instruction on how to deploy redhat openshift pipelines operator.

Red Hat OpenShift Pipelines is a cloud-native continuous integration and delivery (CI/CD) solution for building pipelines using Tekton. Tekton is a flexible Kubernetes-native open-source CI/CD framework, which enables automating deployments across multiple platforms (Kubernetes, serverless, VMs, etc) by abstracting away the underlying details.

***Features***:
Standard CI/CD pipelines definition
Build images with Kubernetes tools such as S2I, Buildah, Buildpacks, Kaniko, etc
Deploy applications to multiple platforms such as Kubernetes, serverless and VMs
Easy to extend and integrate with existing tools
Scale pipelines on-demand
Portable across any Kubernetes platform
Designed for microservices and decentralized team
Integrated with OpenShift Developer Console

***Installation***:
Red Hat OpenShift Pipelines Operator gets installed into a single namespace (openshift-operators) which would then install Red Hat OpenShift Pipelines into the openshift-pipelines namespace. Red Hat OpenShift Pipelines is however cluster-wide and can run pipelines created in any namespace.

## Behaviour:

**Feature:** Deploys openshift pipelines operator.(To Automate the application build,analysis,push image to artifactory, create image and deploy the application into the openshift container platform)  
**When:** After Openshift cluster is deployed

## Tools Required:
1.  AWS Account
2.  RHEL machine(controller node) with Redhat Subscription (7 or higher)
3.  Git
4.  Ansible 2.6 or above
5.  boto
6.  boto3
7.  python >= 2.6

## Pre-requisites

In order to use this role the following pre-requisites are required:

1. Ansible 2.6 or above
2. Openshift cluster should be deployed.

## Setup Steps:

1.  Install git and ansible 2.5 or above on RHEL machine
2.  Clone the git repository (https://gitlab.com/amos-team/openshift4.git) to controller node
3.  Go to [amosds_openshift_container_storage](utils/amosds_openshift_container_storage) folder and perform below action steps

## Configuration:
The variables are defined in the inventories/groups_vars/all/vars file in an inventory.  
Inventories can be found in the top level inventories folder.

**Note**: add common variables to the `vars` list as required.

| Variable  | Description  | Where to Set  |
|---|---|---|
| **region** | region | group_vars/all/vars|
|**instance_type**| provide instance type  | group_vars/all/vars|
| **ami_id** | provide AMI id from AWS | group_vars/all/vars |
| **count** | Add extra nodes to the cephcluster| group_vars/all/vars |
|**master_node_ip**| Provide master node ip | inventories/ansible_hosts |
<!-- |**ansible_ssh_user**| Openshift cluster username(cluster admin) e.g core | group_vars/all/vars |
|**ansible_ssh_private_key_file**| Provide the path of pem file which is used to ssh master node | inventories/ansible_hosts | -->
|**ansible_ssh_user: core
ansible_become: "yes"
ansible_ssh_private_key_file: /home/username/.ssh/london-dev-key
ansible_ssh_port: 22
ansible_connection: ssh
host_key_checking: false
ansible_python_interpreter: /usr/libexec/platform-python**| Provide this details to connect the master nodes for ansible|inventories/ansible_hosts|
|**ocp_api_url**| Provide url of openshift cluster to login | group_vars/all/vars|
|**ocp_login_token**| Provide user's login token from the cluster| group_vars/all/vars|
|**odf_channel**| Provide operator version e.g: stable-4.10 | group_vars/all/vars|
|**STORAGE_CLASS**| Provide storage class name to utilize the app | group_vars/all/vars|
|**VOLUME_CAPACITY**| Provide the capacity of volume to attach to pvc | group_vars/all/vars|

## Running playbook
The following command will run the scripts using the [hosts](./inventories/hosts) inventory :

 Note: Before running below command, please check you are at this directory /utils/amosds_openshift_container_storage

$ ansible-playbook -i inventories/ansible_hosts install_ocs.yaml