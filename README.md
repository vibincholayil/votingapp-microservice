# Votingapp Microservice

This a end-to-end CI/CD of a multi-microservice voting application. I have setup entire DevOps Project in Azure DevOps using Docker images, Kubenetes deployment files and ArgoCD for deployment management.   

## Overview 
This voting application developed by the Docker team, which consists of multiple microservices written in different programming languages.  
link: https://github.com/dockersamples/example-voting-app  
-   Voting microservice written in Python (ci)
-   Results microservice written in Node.js (ci)
-   Worker microservice written in .NET
-   In-memory data store using Redis 
-   PostgreSQL database  

With the code provided by the Docker team, the Dockerfile and Kubernetes deployment file were already included in their GitHub repository. So, I started my work from there. I have understood the code structure and completed all the following steps, including handling the ERROR:  

# CI using Azure DevOps

### Create Resource group for this project Azuure
I have created a RG for easy mangement of all resources i have to create.  

### SetUp Azure DevOps, organization and project
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/project.png)  
I have created a organisation and a project inside that called 'votingapp'(Private visibility)  
imported the docker teams GitHub repo to Azure DevOps repo  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/az_repo.png)  
```
copy - http of GitHub url
import - to  Azure DevOps repo
``` 
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/branch.png)  
Made 'main' branch as default branch

### create azure registry
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/cr.png)  

### Create a VM for run Azure Agent
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/vm.png)  
Created a vm with ubuntu OS  

### Create Azure Agent
Created the Agent for connecting VM and Azure devOps, 
- created Agent Pool
- created Agent 'votingappagent' inside agentpool
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/agent_create1.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/agent_create2.png)  

Login the VM in a CLI - SSH  
```
sudo apt update
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/agent_install1.png)  
```
./config.sh
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/agent_install2.png)  
Personal token have copy from the right top corner
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/agent_install3.png)  
configuration done but NOT run yet  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/agent_install4.png)  
install Docker
```
sudo apt update
sudo apt install docker.io
```
Grant permission to user
```
whoami
sudo usermod -aG docker learning
sudo systemctl restart docker
```
Restart the VM
```
./run.sh
```
### SetUp Azure Pipelines: Create pipelines for microservice
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipeline1.png)  
Azure Repo Git (YAML)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipeline2.png)  
Docker: Build and Push to ACR  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipeline3.png)  
Configuring  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelinecode1.png)  
Set up path-based triggers in the pipeline YAML files to ensure that changes in specific directories trigger the corresponding pipeline.  
Path base trigger - stages - steps  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelinecode2.png)  
Note: result to vote  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelinecode3.png)  
Build  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelinecode4.png)  
Push  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelinecode5.png)  
Build Successful  

# CD with GitOps
GitOps is a approach to continuous delivery that uses Git as a single source of truth for declarative infrastructure and applications.  
In this project, I will use Argo CD to manage my Kubernetes deployments  

### Create a Cluster in Kubernetes
AKS setting: Node scaling minimum 1 maximum 2 for demo purpose.  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/k8s1.png)  
Connect cluster CLI configure  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/k8s2.png)  

### Install ArgoCD in the cluster  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd1.png)  
Link: https://argo-cd.readthedocs.io/en/stable/getting_started/  

![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd2.png)  
Checked ardocd pods
```
kubectl get pod -n argocd
```

### Connect Argocd to Azure repo
For connecting Argocd with my azure repo, i need to find the Argocd secret password  
confirm namespace  
```
kubectl get namespace
```
```
kubectl get secrets -n argocd
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd3.png)  
```
kubectl edit secret argocd-initial-admin-secret -n argocd  
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd4.png)  
i have copied the password and paste it in a text file.  
this password/secret is base64 encoded, so I need to decode.   
```
echo VHdvWVVLcjgxOWpXaDB4TA== | base64 -d
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd5.png)  
past the decoded scret in the same text file  

### check Node
```
Kubectl get svc -n argocd
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/node1.png)  
I need to expose my argocd service to NodePort mode , currently it is in ClusterIP  
```
kubectl edit svc argocd-server -n argocd
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/node2.png)  
Changed the argocd service type from ClusterIP mode to NodePort mode  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/node3.png)  
Changed service type  
Access ArgoCD service (SVC) in ui - need node external IP  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/node4.png)  
```
kubectl get node -o wide
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/node5.png)  
Not able to access : need to open the inbound traffic of the port  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/vmss.png)  
I need to open VMSS and Check networking and add inbound rule.  
open unsafe in unsafe mode with NodePortip:port  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/node6.png)  


### Connect argocd (countinue...)
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd7.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd8.png)  
Clone repo to argocd  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd9.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd10.png)  
Create Application  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd11.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd12.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd13.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd14.png)  
Create  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd15.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd16.png)  

### Connect kubernetes manifest file to gitops - using shell script
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/shellscript.png)  
Create a folder in repo called script and create shellscript 'updatek8manifest.sh'
```
#!/bin/bash

set -x

# Set the repository URL
REPO_URL="https://<personal token>@dev.azure.com/vibincholayil/votingapp/_git/votingapp"

# Clone the git repository into the /tmp directory
git clone "$REPO_URL" /tmp/temp_repo

# Navigate into the cloned repository directory
cd /tmp/temp_repo

# Make changes to the Kubernetes manifest file(s)
# For example, let's say you want to change the image tag in a deployment.yaml file
sed -i "s|image:.*|image: votingappvibincicd.azurecr.io/$2:$3|g" k8s-specifications/$1-deployment.yaml

# Add the modified files
git add .

# Commit the changes
git commit -m "Update Kubernetes manifest"

# Push the changes back to the repository
git push

# Cleanup: remove the temporary directory
rm -rf /tmp/temp_repo

```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/shellscript1.png)  
Add the shellscript in pipeline by providing script path and argument for image update automatically.  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelineupdate1.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelineupdate2.png)  
```
scripts / updatek8manifest.sh
vote $(imageRepository) $(tag)
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelineupdate3.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelineupdate4.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pipelineupdate6.png)  
Sucessful  

### Add image pull secret in the Kubernetes Manifest file
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/k8s3.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/k8s4.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/k8s5.png)  
Commit  
  
Write below comment in Kubernetes cluster
#### create ACR ImagePullSecret
DO NOT USE REAL CREDENTIALS IN FILES
```
kubectl create secret docker-registry <secret-name(anything)> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
```
Container Registry --> Access key 
finding container registy id, username, password.  

![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocd16.png)  
```
kubectl get deploy vote -o yaml - image name check
kubectl get svc
kubectl get node -o wide
```
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/pod1.png)  
Open port  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/vmss2.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/node5.png)  
Sucessful: Service
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/vote.png)  

### Update ArgoCD sync time from default 180 sec to10 Sec
```
kubectl edit cm argocd-cm -n argocd
```
```
data:
  timeout.reconciliation:10s
```

### Result K8s Deployment file
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/resultk8s.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/argocdresult.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/resultnode1.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/resultsvc.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/resultpipelinecode.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/resultoutput.png)  

# ERROR/Challenges
### Change branch
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/errorbranch1.png)  
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/errorbranch2.png)  


### imagepullbackoff
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/errorargo1.png)  
argocd has error because of image pull back off
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/errorargo2.png)  
Debug
![project](https://github.com/vibincholayil/votingapp-microservice/blob/master/images/errordebug1.png)  
Reason: When argocd trying to pull the image from a private registry (Votingappvibin) need to know the secret

# Conclusion
In this project, I used Azure DevOps and GitOps to implement a complete CI/CD workflow. I explored concepts like Continuous Integration and Continuous Delivery, set up Azure Pipelines, created Docker images, and used Argo CD for Kubernetes deployment. I faced and debugged many issues during the setup—two major errors/challenges are explained in the documentation. This project is a great demonstration of my skills in DevOps and GitOps concepts.

#### Thank you


