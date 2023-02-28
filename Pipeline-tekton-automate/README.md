## Introduction:

This document will guide you with step by step work instruction on how to deploy openshift redhat pipelines operator.

## Behaviour:

**Feature:** Deploys openshift redhat pipelines operator.
**When:** After Openshift cluster is deployed

## Tools Required:
1.  AWS Account
2.  RHEL machine(controller node) with Redhat Subscription (7 or higher)
3.  Git
4.  Ansible 2.9.17 or above
5.  boto
6.  boto3
7.  python >= 2.6
8. nexus - artifactory repository to store the jar or war files 
9. sonarqube - for code analysis
10. pom.xml file - to create project structure, .class files, jar or war files.

## Pre-requisites

In order to use this role the following pre-requisites are required:

1. Ansible 2.9.17 or above
2. Openshift cluster should be deployed.
3. nexus should be deployed (NEXUS_IMAGE: "sonatype/nexus3:3.38.1").

| S.No | Deployment Steps of Nexus |
|---|---|
| **1** | Inorder to deploy the nexus application, from webconsole (leftside panel) choose as a developer |
| **2** | click on "+Add" -> containerimages->select on "Image name from external registry" |
| **3** | provide image as "sonatype/nexus3:3.38.1" |
| **NOTE** | if image is not available we need to pull it and push to our internal registry in the project |
| **4** | Provide "application name", Resources-> deployment |
| **5** | leave it remain as default -> click on create |
  
| S.No  | Steps to login nexus and setup the users,role, artifact repos  | 
|---|---|
| **1** | login with default credentails (user: admin & password: cat /nexus-data/admin.password (take   terminal of nexus pod and cat that file. will see password) ). Once logged in as admin, it will ask to change the default password. so we should do it.|
| **2** | Roles -> create role -> nexus role-> provide 'RoleID',"RoleName(deploy)'Role Description'->Privileges- select ->nx-all role & add it->click on create role. |
| **3** | setup the user account as (e.g: user: deploy-user and passwd: deploy123 (anything we can setup)).Users->create local user -> provide required details as 'ID', password, first&last names, email, status as active->Roles select as deploy role & add it->click on 'create local user'. |
| **4** | logout as admin & login as deploy-user |
| **5** | create repositories as follows: select repositories -> create repository -> Maven2(hosted)->provide name of the repository e.g: java-release-> version policy as 'release'->Layout policy as 'Permissive'-> Deployment policy as 'Allow Redeploy'-> leave it all as default if remaining -> click on 'create repository' |
| **6** |  Once created the repository looks like as  https://nexus3-tekton-pipelines.apps.amoslondondev.amosonline.io/repository/maven-releases/ |
| **7** |  if you need another repository as snapshot, you should follow the below steps: select repositories -> create repository -> Maven2(hosted)->provide name of the repository e.g: java-snap-> version policy as 'snapshot'->Layout policy as 'Permissive'-> Deployment policy as 'Allow Redeploy'-> leave it all as default if remaining -> click on 'create repository' |
| **8** | Once created the repository looks like as https://nexus3-tekton-pipelines.apps.amoslondondev.amosonline.io/repository/maven-snapshots/ |         
| **NOTE** | The two repositories should be update in pom.xml file. so that the jar/war's will store into it.|
| **9** | update in pom.xml file with repository names |

4. sonarqube should be deployed (Sonar_Image: sonarqube:8.9.6-community)

| S.No | Deployment Steps of sonarqube |
|---|---|
| **1** | Inorder to deploy the nexus application, from webconsole (leftside panel) choose as a developer |
| **2** | click on "+Add" -> containerimages->select on "Image name from external registry" |
| **3** | provide image as "sonarqube:8.9.6-community" |
| **NOTE** | if image is not available we need to pull it and push to our internal registry in the project |
| **4** | Provide "application name", Resources-> deployment |
| **5** | leave it remain as default -> click on create |

| S.No  | Steps to follow  |
|---|---|
| **1** |login with default credentails. ( user: admin , passwd: admin)|
| **2** | create a project -> add project -> Manually -> project_key & display name -> click on "setup"-> provide token like "springboot java" (anything we can provide) -> click on 'Generate'-> continue -> click on Maven. |
| **3** | we can see as below example|
|***Example** | mvn sonar:sonar -Dsonar.projectKey=spring-boot -Dsonar.host.url=https://sonarqube-tekton-pipelines.apps.amoslondondev.amosonline.io  -Dsonar.login=d0f99988733210c83440dae45569096799bf0cb7 |
| **NOTE** |  as per the above o/p you can take the required parameters i.e projectkey, token(login) & add it to vars file (group_vars/all/vars) |
| **4** | update the sonar qube url and nexus artifact repo detils in pom.xml file as variable |

**NOTE**: nexus & sonar should be deployed in same namespace where we are deploying our pipeline tasks.

## Setup Steps:

1.  Install git and ansible 2.9.17 or above on RHEL machine
2.  Clone the git repository (https://gitlab.com/amos-team/openshift4.git) to controller node
3.  Go to [tekton-pipeline](utils/tekton-pipeline) folder and perform below action steps


## Configuration:
The variables are defined in the inventories/groups_vars/all/vars file in an inventory.  
Inventories can be found in the top level inventories folder.

**Note**: add common variables to the `vars` list as required.

| Variable  | Description  | Where to Set  |
|---|---|---|
| **ocp_login_token** | copy the login token from webconsole of openshift | group_vars/all/vars|
| **ocp_api_url** | Add url of openshift cluster to login | group_vars/all/vars |
| **pipelines_channel** | provide operator channel version e.g: pipelines-1.8 | group_vars/all/vars |
| **NAMESPACE** | add namespace where you can create the pipeline tasks | group_vars/all/vars |
| **git_username** | add git username | group_vars/all/vars |
| **git_token** | add gituser token | group_vars/all/vars |
| **storage_size** | provide storage size e.g: 5Gi | group_vars/all/vars |
| **int_registry_user_name** | add internal registry username which we have used to login the openshift web  console | group_vars/all/vars |group_vars/all/vars |
|**SA**|Provide Service account name|group_vars/all/vars |
| **OCP_TOKEN** | Provide ocp token i.e copy login token from openshift webconsole|group_vars/all/vars |
| **registry_url**|add internal registry url, can get it from openshift webconsole|group_vars/all/vars |
| **Image_stream**|provide imagestream name to store the image|group_vars/all/vars |
| **image_version**|provide the image version, e.g: 0.1|group_vars/all/vars |
| **sonar_host_url**|Provide sonarqube url|group_vars/all/vars |
| **sonar_login_token**|Provide sonarqube login token after creation of the project in sonarqube will get it|group_vars/all/vars |
| **sonar_project_key**|Provide sonarqube projectkey after creation of the project in sonarqube will get it|group_vars/all/vars |
| **nexus_username**|Provide nexus username after creation of local user in nexus|group_vars/all/vars |
| **nexus_user_passwd**|Provide nexus_user_passwd after creation of local user's password in nexus|group_vars/all/vars |

## Running playbook
The following command will run the scripts using the (./inventories/hosts) inventory :

 Note: 
 - Before running below command, please check you are at this directory /utils/tekton-pipeline 

$ ansible-playbook -i inventories/ansible_hosts install_pipelinesoperator.yaml

## After successfully ran the script we need to create webhook to trigger the pipeline automatically

Get the route of the eventlistener, where you have deployed it (e.g: select the project->networking tab->route) ex: https://el-gtlb-litner-tekton-pipelines.apps.amoslondondev.amosonline.io

Go to gitlab(where the application source code is available)->repositories->choose your application code repository(e.g: pipeline)->settings->webhooks-> provide the URL (mention route)->trigger->click on push events->SSL Verification -> uncheck the "Enable SSL Verification" option-> Add webhook.

NOTE: https://docs.gitlab.com/ee/user/project/integrations/webhooks.html

Once created it. There is option to "test" it OR you can update the application code & push it to your repo so automatically the pipeline will trigger.

## Once Pipeline has ran Successfully, Application Verification steps

| S.No | Application verification steps after deployment |
|---|---|
| **1** | Get the route of your application, where you have deployed it (e.g: select the project->networking tab->route) ex: http://java-app-tekton-pipeline-peer.apps.amoslondondev.amosonline.io/ |
| **2** | copy that and paste it on the browser -> enter. will see the application page|


## If you want to rerun the pipeline again, we have to do few modifications as below. So that, always the latest image will pick it & deployed.

| S.No | where to do modifications |
| --- | ---|
| **1** | In pipeline tasks i.e image build task (task name as bu-mavn), have to update the version of the image as 0.X e.g: 0.1. There is sub task name as "push-to-internal-registry" e.g: default-route-openshift-image-registry.apps.amoslondondev.amosonline.io/tekton-pipeline-peer/javaspb:0.X|
| **2**| so that every time you build, the new version of image will create & store in respective imagestream |
| **3** | In application code, k8s folder->deployment file->under spec -> image details have to change e.g:image: default-route-openshift-image-registry.apps.amoslondondev.amosonline.io/tekton-pipeline-peer/javaspb:0.X |
| **NOTE** | Make sure the version number should same as per point #1 & #3 otherwise it will not pick the right image |

*********************Where 'X' is number***************************

