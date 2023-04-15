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


