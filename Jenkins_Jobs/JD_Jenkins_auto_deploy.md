# JD Jenkins Auto Deploy 
## CICD Pipeline using Git , Jenkins and Maven
1. Setup linux server on EC2
2. Install jenkins 
- https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Jenkins/Jenkins_Installation.MD
3. Integrate Git with Jenkins 
- Install git on linux server and git plugin in jenkin and configure git path Manage Jenkins > Global Tool Configuration > git
- https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Jenkins/Git_plugin_install.MD
- ![image](https://user-images.githubusercontent.com/69948118/231976779-429e3480-f8b7-45b2-b61e-216bf63401dd.png)
4. Integrate Maven with jenkins
- instal maven on linux server and plugin in jenkin Manage Jenkins > Global Tool Configuration > Maven
- ![image](https://user-images.githubusercontent.com/69948118/231977144-52bee94b-dc7f-421d-bc78-1c42be1959e1.png)
- ![image](https://user-images.githubusercontent.com/69948118/231977658-02e76511-8029-46df-bae7-5a868e442c77.png)
- https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Jenkins/maven_install.MD
5. Create jenkin job for fetch code from git and build using maven

## Integrating Tomcat server in CICD pipeline
1. create different EC2 linux server 
2. Install tomcat
- https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Tomcat/tomcat_installation.MD
3. Install deploy to container plugin
4. setup credentails in add credentails for tomcat server
![image](https://user-images.githubusercontent.com/69948118/231979076-57794057-2eb4-401d-a8d0-e6ac1bf07be3.png)
![image](https://user-images.githubusercontent.com/69948118/231979145-4060a64e-f010-43c5-a820-f5f5643fbfdd.png)

## Integrating Docker in CICD Pipeline
1. Install doccke on linux server https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Docker/Docker_Installation_Steps.MD
2. Create tomcat container using existing tomcat image
```Dockerfile
From tomcate:latest
RUN cp -R /user/local/tomcat/webapps.dist/* /user/local/tomcat/webapps
```
4. Integrating jenkins with Docker
- create dockeradminuser and add that user into docker group
- Enable password login in linux https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Docker/Docker_Installation_Steps.MD
- Install plugin publish over ssh to transfering artifacts to docker host
- configure system
- ![image](https://user-images.githubusercontent.com/69948118/231982196-042ea168-b753-4b79-8395-b9d713ff7923.png)

5. Jenkins job pull the code from github , build using mavena and copy artifacts on docekr host
- Please follow this readme file https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Jenkins_Jobs/Deploy_on_Container.MD
- ![image](https://user-images.githubusercontent.com/69948118/231983172-d2f80de2-6d11-4c30-bdbf-89ea841ff5a4.png)
- create one docker directory and give ownership as dockeradmin server and give require permission on the directory and Dockerfile
- copy artifacts from dockerhost to tomcat container
```Dockerfile
From tomcate:latest
RUN cp -R /user/local/tomcat/webapps.dist/* /user/local/tomcat/webapps
# copy war file on to container 
COPY ./webapp.war /usr/local/tomcat/webapps
```
- create image from dokerfile and run container using sh scripts exec commands
```sh
cd /opt/docker;
docker build -t regapp:v1 .;
docker stop registerapp;
docker rm registerapp;
docker run -d --name registerapp -p 8087:8080 regapp:v1
```


## Integrating Ansible in CICD pipelin
1. Ansible installation on ansible controller  https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Ansible/Ansible_installation.MD
  - setup EC2 , setup hostname, create admin user , add to visudo , generate ssh keys , enable passwpord based auth  install anisble
2. Integrating with managed node where docker installed
 - Create ansadmin user on docker host machine
 - add ansadmin to visudo
 - enable password based login
 - ON ANSIBLE CONTOROLLER
 - Add managed ip into hosts (private ip)
 - copy ssh keys into maanged host using private ip
 - test connetion
3. Integrating Anisble with Jenkins
 - Jenkins will do buil using maven and copy artifacts into anisble then ansible will create image and push to docker hub
 - Add ansible into configure system in jenkins
 - ![image](https://user-images.githubusercontent.com/69948118/232180803-0750ed67-8e0c-4165-bb5b-05dffff12f09.png)
- ![image](https://user-images.githubusercontent.com/69948118/232180838-40b17bb7-0ea9-4092-83f8-9a82865cad76.png)
4. Ansible playbook to create image and container
- Install docker on ansible controller
- Add ansadmin user into docker group
- create docker file in /opt/docker
```sh
From tomcate:latest
RUN cp -R /user/local/tomcat/webapps.dist/* /user/local/tomcat/webapps
# copy war file on to container 
COPY ./webapp.war /usr/local/tomcat/webapps
```
- sudo chmod 777 /var/run/docker.sock
- add hosts in /etc/ansible/hosts as inventory for ansible server and docker host
- we need to copy key into ansible machine itself even we generate key in anisble server
- -create-docker-image.yml
```yaml
---
- hosts: all
  #ansadmin doesn't need root access to create an image
  become: true 

  tasks:
  - name: building docker image
    command: "docker build -t simple-devops-image ." 
    args:
      chdir: /opt/docker
```
- anisible-plabook create-docker-image.yml --check 
- anisible-plabook create-docker-image.yml
5. Push docker image to docker hub
- login docker hub
- docker tag image-id jbirla/regapp:latest
- docker push jbirla/regapp:latest
6. Edit anible playbook for tag and push docker image
```Dockerfile
- hosts: all
  #ansadmin doesn't need root access to create an image
  become: true 

  tasks:
  - name: building docker image
    command: "docker build -t simple-devops-image ." 
    args:
      chdir: /opt/docker
  - name: Create tag for push image
    command: "docker tag -t regapp:latest jbirla/regapp:latest ." 
    
   - name: Push image to docker hub
    command: "docker push jbirla/regapp:latest." 

```
![image](https://user-images.githubusercontent.com/69948118/232181303-b236c9d1-a64a-4e25-9670-4d738bd2d32b.png)

7. Create container on dockerhos using ansible plabook 
- deploy.yaml
```Dockerfile
---
- hosts: dockerhost
  become: ture

  tasks:
  - name: creating container 
    command: docker run -d --name regapp-server -p 8080:8080 jbirla/regapp:latest
    
 ```
 - sudo chmod 777 /var/run/docker.sock
8. Continuouse deployment using jenkin job
- deploy.yaml
```Dockerfile
---
- hosts: dockerhost

  tasks:
  - name: stop existing container
    command: docker stop regapp-server
    ignore_errors: yes

  - name: remove the container
    command: docker rm regapp-server
    ignore_errors: yes

  - name: remove image
    command: docker rmi jbirla/regapp:latest
    ignore_errors: yes

  - name: create container
    command: docker run -d --name regapp-server -p 8082:8080 jbirla/regapp:latest
 ```
-  add playbook execution on jenkinn jobs
-  ![image](https://user-images.githubusercontent.com/69948118/232181479-9cdc8f86-7722-43e7-b602-b921d47aaaa2.png)

## Kubernetes on AWS
1. setup kubernetes on AWS https://github.com/jdbirla/Simple-DevOps-Project/blob/master/Kubernetes/kubernetes_setup_using_eksctl.md
2.Install AWS latest CLI
```sh
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

[root@ip-172-31-33-223 ~]# aws --version
aws-cli/1.18.147 Python/2.7.18 Linux/5.10.176-157.645.amzn2.x86_64 botocore/1.18.6
[root@ip-172-31-33-223 ~]# exit
logout
[ec2-user@ip-172-31-33-223 ~]$ aws --version
aws-cli/2.11.13 Python/3.11.3 Linux/5.10.176-157.645.amzn2.x86_64 exe/x86_64.amzn.2 prompt/off
[ec2-user@ip-172-31-33-223 ~]$

```
3. install kubctl
```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.2/2023-03-17/bin/linux/amd64/kubectl
[root@ip-172-31-33-223 ~]# ll
total 103152
drwxr-xr-x 3 root root       78 Apr 14 20:45 aws
-rw-r--r-- 1 root root 57595989 Apr 15 05:05 awscliv2.zip
-rw-r--r-- 1 root root 48029696 Apr 15 05:09 kubectl
[root@ip-172-31-33-223 ~]# clear
[root@ip-172-31-33-223 ~]# chmod +x ./kubectl
[root@ip-172-31-33-223 ~]# chmod +x ./kubectl
[root@ip-172-31-33-223 ~]# mv ./kubectl /usr/local/bin
[root@ip-172-31-33-223 ~]# kubectl --version
error: unknown flag: --version
See 'kubectl --help' for usage.
[root@ip-172-31-33-223 ~]# kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26+", GitVersion:"v1.26.2-eks-a59e1f0", GitCommit:"8b68f4b95d7121d039ceebd30870e48acc7772e4", GitTreeState:"clean", BuildDate:"2023-03-09T20:03:04Z", GoVersion:"go1.19.6", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
The connection to the server localhost:8080 was refused - did you specify the right host or port?
[root@ip-172-31-33-223 ~]#

```
4. installing eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
[root@ip-172-31-33-223 ~]# curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
[root@ip-172-31-33-223 ~]# cd /tmp/
[root@ip-172-31-33-223 tmp]# ll
total 126556
-rwxr-xr-x 1 1001  123 129593344 Apr 11 10:18 eksctl
drwx------ 3 root root        17 Apr 15 05:00 systemd-private-2dc8075677b04a94bc22e95686aa7ba3-chronyd.service-3N1FaP
[root@ip-172-31-33-223 tmp]# sudo mv /tmp/eksctl /usr/local/bin
[root@ip-172-31-33-223 tmp]# eksctl version
0.137.0
[root@ip-172-31-33-223 tmp]#


```
5. Create a role
![image](https://user-images.githubusercontent.com/69948118/232184555-61b26847-973f-487e-a9ea-ab35c22e11f5.png)
- attache that role to ec2_bootstrap server

6.Create your cluster and nodes using bootstrap server

```
[root@ip-172-31-33-223 tmp]# eksctl create cluster --name jdekscluster --region ap-south-1 --node-type t2.small
```

7. deploya application
```
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: jbirla-regapp
  labels: 
     app: regapp

spec:
  replicas: 2 
  selector:
    matchLabels:
      app: regapp

  template:
    metadata:
      labels:
        app: regapp
    spec:
      containers:
      - name: regapp
        image: jbirla/regapp
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```
```
apiVersion: v1
kind: Service
metadata:
  name: jbirla-service
  labels:
    app: regapp 
spec:
  selector:
    app: regapp 

  ports:
    - port: 8080
      targetPort: 8080

  type: LoadBalancer
```

8. Create playbook for create image and deploy into kubernetes
```
---
- hosts: kubernetes
#  become: ture
  user: root

  tasks:
    - name: deploy regapp on kubernetes
      command: kubectl apply -f regapp-deploy.yml

    - name: create service for regapp
      command: kubectl apply -f regapp-service.yml

    - name: update deployment with new pods if image updated in docker hub
      command: kubectl rollout restart deployment.apps/jbirla-regapp
[ansadmin@ip-172-31-34-166 docker]$
```
![image](https://user-images.githubusercontent.com/69948118/232223026-46d6b146-c728-4379-883c-fd08be9803d4.png)
![image](https://user-images.githubusercontent.com/69948118/232223032-058ab8ff-5a85-486f-af59-d77c7cc0f767.png)
![image](https://user-images.githubusercontent.com/69948118/232223038-b8088dd3-dece-4f19-99ed-1e80085fc8cf.png)
![image](https://user-images.githubusercontent.com/69948118/232223048-d5c5c36a-8d28-420d-832b-ce8c737c28f5.png)
![image](https://user-images.githubusercontent.com/69948118/232223056-90b91371-21b9-4426-bbc6-4a3c6a72fad9.png)
![image](https://user-images.githubusercontent.com/69948118/232223071-16d521dc-fc57-476f-989e-7713792d5ec8.png)
![image](https://user-images.githubusercontent.com/69948118/232223083-eb1744a1-1222-4f5d-a18b-2818b2eaccbd.png)
![image](https://user-images.githubusercontent.com/69948118/232223095-684abef4-cd79-4e83-8f45-04d5dc9d42f7.png)

