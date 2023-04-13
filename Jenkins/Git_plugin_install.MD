# Configure Git pulgin on Jenkins
Git is one of the most popular tools for version control system. you can pull code from git repositories using jenkins if you use github plugin. 


#### Prerequisites
1. Jenkins server 

#### Install Git on Jenkins server
1. Install git packages on jenkins server
   ```sh
   yum install git -y
   ```
 ```sh
 whereis git
git: /usr/bin/git /usr/share/man/man1/git.1.gz
[root@ip-172-31-10-155 ~]# cd /var/lib/jenkins/workspace/PullCodefromGithub
[root@ip-172-31-10-155 PullCodefromGithub]# ls
Dockerfile  pom.xml  README.md  regapp-deploy.yml  regapp-service.yml  server  webapp
```
#### Setup Git on jenkins console
- Install git plugin without restart  
  - `Manage Jenkins` > `Jenkins Plugins` > `available` > `github`

- Configure git path
  - `Manage Jenkins` > `Global Tool Configuration` > `git`

