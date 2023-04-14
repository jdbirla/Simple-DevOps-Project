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
