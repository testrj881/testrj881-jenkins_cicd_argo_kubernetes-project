# Jenkins-Zero-To-Hero

steps for CICD pipeline

🟢 #Execute the application locally and access it using your browser
#Checkout the repo and move to the directory

git clone https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/sprint-boot-app
cd java-maven-sonar-argocd-helm-k8s/sprint-boot-app

🟢 #Execute the Maven targets to generate the artifacts locally

mvn clean package

#The above maven target stores the artifacts to the target directory.
🟢 #You can either execute the artifact on your local machine (or) run it as a Docker container.
** Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way. **

## Aprroach1. #Execute locally (Java 11 needed) and access the application on http://localhost:8080

java -jar target/spring-boot-web.jar

## Aprroach2. #The Docker way, Build the Docker Image

docker build -t ultimate-cicd-pipeline:v1 .
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1



---------------

Are you looking forward to learn Jenkins right from Zero(installation) to Hero(Build end to end pipelines)? then you are at the right place. 

🟢## Installation on EC2 Instance

YouTube Video ->
https://www.youtube.com/watch?v=zZfhAXfBvVA&list=RDCMUCnnQ3ybuyFdzvgv2Ky5jnAA&index=1

🟢Install Jenkins, configure Docker as agent, set up cicd, deploy applications to k8s and much more.

## AWS EC2 Instance

- Go to AWS Console
- Instances(running)
- Launch instances



🟢 Install Java

```
sudo apt update
sudo apt install openjdk-17-jre
```

Verify Java is Installed

```
java -version
```

🟢 Now, you can proceed with installing Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

🟢 **Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).



🟢 ### Login to Jenkins using the below URL:

http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins UI, 
      - username: admin
      - Run the command to copy the Jenkins Admin Password - sudo cat /var/lib/jenkins/secrets/initialAdminPassword
      - Enter the Administrator password: 4260312c14c249b981fd0152030d337c
      

🟢 ### Click on Install suggested plugins

Wait for the Jenkins to Install suggested plugins
Create "Admin" User or Skip other steps for now. 
Login in Jenkins UI server
## Install the Docker and SonarQube Scanner Pipeline plugin in Jenkins:
   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for "Docker Pipeline".
   - Select the plugin and click the Install button.
   - In the Available tab, search for "sonarqube Scanner" Pipeline".
   - Select the plugin and click the Install button.
   - Restart Jenkins after the plugin is installed.

## Docker Slave Configuration. Run the below command to Install Docker on EC2 machine 

```
sudo apt update
sudo apt install docker.io
```
 
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su -  # this switches to root user (not recommended) 
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

### create a pipeline

go to manage Jenkins >> credentials >> system >> global credential >> add credential >> (add SonarQube authentication here )
in Add credential >> Select kind : secret text >>  add describe and fill details and save

------------
🟢🟢🟢🟢🟢🟢


****SonarQube (installed on EC2)

adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
#starting the SonarQube UI on
./sonar.sh start

Hurray !! Now you can access the SonarQube Server on http://<ip-address>:9000
open SonarQube in  new browser : >> https://<PublicIP>:9000
user: admin and password: admin
reset the password: admin1

Go to Administrator >> Seurity >> Generate token
New token "jenkins_userToken" has been created. Make sure you copy it now, you won't be able to see it again!
token: squ_f82238e984ab5c8f5fa910f2d2b19916e5c778b3
copy the token

New token "jenkins_globalAnalysisToken" has been created. Make sure you copy it now, you won't be able to see it again!
token: sqa_1a9dc22113d89c1c22a1f7c8ae6a4e32124fd85e
copy the token

Go to Jenkins:
After you login to Jenkins, -
Run the command to copy the Jenkins Admin Password -
sudo cat /var/lib/jenkins/secrets/initialAdminPassword -
username: admin
Enter the Administrator password: 4260312c14c249b981fd0152030d337c

go to >> manage Jenkins >> credentials >> system >> global credential >> add credential >> (add SonarQube authentication here )
in Add credential >> Select kind : secret file >>  add describe and fill details and save


Docker: (installed on EC2)

exit from SonarQube user login
as root or sudo user

1.	Run the below command to Install Docker

sudo apt update
sudo apt install docker.io

2. Grant Jenkins user and Ubuntu user permission to docker deamon.
Note: Docker user must be same as Jenkins
sudo su -
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu #here ubuntu is user. you can use $USER instead of ubuntu
systemctl restart docker
systemctl restart jenkins
systemctl status docker
systemctl status jenkins
#verify the group and you must see docker in the list
groups jenkins


Once you are done with the above steps, it is better to restart Jenkins.
you can run below command in browser
http://<ec2-instance-public-ip>:8080/restart
The docker agent configuration is now successful.




MINIKUBE-kubernetes (not installed on EC2)

open a project directory in local machine

1.	Run below commands

minikube version
kubectl version --client
minikube start --memory=4098 --driver=docker

output --------
Verifying Kubernetes components ...
. Using image gcr. io/k8s-minikube/storage-provisioner:v5
Enabled addons: storage-provisioner, default-storageclass
Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default



AGROCD (not installed on EC2)

browse search operatorhub in google. search argocd and install. step given as below
Note: can you other version of OLM tool which are compactible with your installed Kubernetes

1.	Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.

$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.32.0/install.sh | bash -s v0.32.0

2. Install the operator by running the following command:What happens when I execute this command?

$ kubectl create -f https://operatorhub.io/install/argocd-operator.yaml

This Operator will be installed in the "operators" namespace and will be usable from all namespaces in the cluster.

3. After install, watch your operator come up using next command.

$ kubectl get csv -n operators


-----------Alternate approach------
reference: https://devopscube.com/setup-argo-cd-using-helm/#argocd-setup-using-manifest-files 

kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

To login to the UI, you need a username: admin  and password using below command:
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode ; echo


kubectl port-forward svc/argocd-server -n argocd 8080:443

#It binds your local port 8080 to the ArgoCD server service’s port 443, and WAITS indefinitely —
#this is expected behavior. It doesn’t “complete” — it just stays running and starts forwarding traffic.
#No success message is printed unless there's an error.
#Once port-forwarding starts, it just sits there silently.
#The ArgoCD UI is now accessible at:
#Open your browser and go to:
https://localhost:8080
Ignore the security warning (because the certificate is self-signed) and proceed.

Login credentials:
Username: admin
Password: The initial password is stored in a Kubernetes secret.
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


kubectl get svc -n argocd
kubect edit svc argocd-server -n argocd
#change service type from clusterip to nodeport

minikube service argocd-server
minikube service argocd-server -n argocd
minikube service list
kubectl get pods
kubectl get pods -n argocd
kubectl get secret
kubectl get secret -n argocd
kubectl edit secrets argocd-initial-admin-secret -n argocd
#copy admin password to decode in base64 and Edit cancelled, no changes made.


♦♦♦ Addition change jenkins pipeline:
-- create new clone and update the git repo in jenkins 
-- validate docker iimage is downloable
-- update giturl in "checkout" task 
-- update token and credential id in sonarqube task 
-- update github credentials and add git token in stage('Update Deployment File')

-- push the change in the testrj881 repo. 



♦♦♦Troubleshoot: 


✅ disk full issue in Jenkins 
sudo rm -rf /var/lib/jenkins/workspace/*
docker system prune -af
sudo apt-get clean
sudo journalctl --vacuum-time=2d
df -h

Optional:
# Clean old Docker images (use with caution)
docker system prune -af

# Clean apt cache
sudo apt clean

# Remove unused snap versions
sudo snap list --all | awk '/disabled/{print $1, $3}' | while read snapname revision; do sudo snap remove "$snapname" --revision="$revision"; done





✅ Increase EC2 volume and mount in ec2 instance
1️⃣ Check current disk and partition
lsblk
Look for output like:
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   20G  0 disk
└─xvda1 202:1    0    8G  0 part /
If you see xvda1 is still only 8G, continue.

2️⃣ Install cloud-guest-utils (if not already installed)
sudo apt update
sudo apt install cloud-guest-utils -y
This gives you the growpart command.

3️⃣ Grow the partition
sudo growpart /dev/xvda 1
This will resize /dev/xvda1 to use the full 20 GiB.

4️⃣ Resize the filesystem
Now grow the actual file system:
sudo resize2fs /dev/xvda1
This may take a few seconds.

5️⃣ Confirm the new size
df -h /
Expected output:
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        20G  6.0G   14G  30% /
 

 ✅ sonarqube running on port 80 instead of 9000


















