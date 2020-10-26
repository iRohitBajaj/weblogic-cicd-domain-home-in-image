# Demo of CI/CD in WebLogic on K8S project  

cd {cloned-repo-location}  
kubectl create namespace sample-domain2-ns  

### Prerequisites  
* k8 weblogic operator is running  
* k8 domain exists for weblogic deployment  
* mysql db pod exists  
* ingress controller is configured  

## Add jenkins helm chart repo to helm client cache  
Ref: https://github.com/jenkinsci/helm-charts  

helm repo add jenkins https://charts.jenkins.io  

## Create persistence volume to persist jenkins data during restarts  
kubectl apply -f ./jenkins/jenkins-pv.yaml -n sample-domain2-ns

## Copy values.yaml from https://github.com/jenkinsci/helm-charts and update it to point to persistence volume created in previous step  
helm install jenkins -f ./jenkins/helm-values.yaml jenkins/jenkins --namespace sample-domain2-ns  

## Retrieve jenkins password that got generated during pod creation  
printf $(kubectl get secret --namespace sample-domain2-ns jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo  
admin/r0SNlYJ80k  

export POD_NAME=$(kubectl get pods --namespace sample-domain2-ns -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=jenkins" -o jsonpath="{.items[0].metadata.name}")  
kubectl --namespace sample-domain2-ns port-forward $POD_NAME 8080:8080  

## Encrypt admin server and datasource password using weblogic jar and replace it on domain properties file  
java -cp $ORACLE_HOME/wlserver/server/lib/weblogic.jar:$CLASSPATH -Dweblogic.RootDirectory=/Users/rbajaj/Oracle/Middleware/Oracle_Home/user_projects/domains/medrec weblogic.security.Encrypt $PASSWORD  

## Curate your extracted weblogic domain yaml per your needs  
## Curate your domain.yaml per your needs  

## Set up jenkins node  
* Open jenkins console, login as admin, add jdk8 as java installation and point it to download java from oracle site [would require you to configure oracle account in global configuration]. http://localhost:8080/descriptorByName/hudson.tools.JDKInstaller/enterCredential  
* Open jenkins console, login as admin, add kubernetes cluster connection details under cloud configuration, pointing to API server and create a global security configuration using secret as type and upload .kube/config file.  
* Install Docker plugin via Manage plugins.   
* Add docker credentials with id 'dockerhub-login' under manage credentials   


## Add docker agent  
Use image jenkins/jnlp-slave for agent template    
Container Settings -> Volumes - /var/run/docker.sock:/var/run/docker.sock  
For Connect Method select - Attach Docker container  
For user - Jenkins  

## Create a job to execute CI pipeline  
Under pipeline from SCM section add your git repo to pull Jenkinsfile from -  
https://github.com/iRohitBajaj/weblogic-cicd-domain-home-in-image.git  

## Execute job  
* For first time deployment use DEPLOY_TYPE param as "Create"  
* For updates to weblogic domai use DEPLOY_TYPE param as "Update"  








