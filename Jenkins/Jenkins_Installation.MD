# Install Jenkins on AWS EC2
Jenkins is a self-contained Java-based program, ready to run out-of-the-box, with packages for Windows, Mac OS X and other Unix-like operating systems. As an extensible automation server, Jenkins can be used as a simple CI server or turned into the continuous delivery hub for any project.

### Follow this artical [On YouTube Channel](https://youtu.be/ERR7cqW28FY)

### Prerequisites
1. EC2 Instance 
   - With Internet Access
   - Security Group with Port `8080` open for internet
1. Java 11 should be installed  

## Steps:
1. Connect throug mobaxtram
```
https://aws.amazon.com/amazon-linux-2/
[ec2-user@ip-172-31-0-179 ~]$ sudo su -
[root@ip-172-31-0-179 ~]# clear
```

2. https://pkg.jenkins.io/redhat-stable/

```
[root@ip-172-31-0-179 ~]# amazon-linux-extras install epel
[root@ip-172-31-0-179 ~]# amazon-linux-extras install java-openjdk11
[root@ip-172-31-0-179 ~]#   yum install jenkins
[root@ip-172-31-0-179 ~]# service jenkins status
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@ip-172-31-0-179 ~]# service jenkins start
Starting jenkins (via systemctl):                          [  OK  ]
[root@ip-172-31-0-179 ~]# service jenkins status
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-04-13 06:53:24 UTC; 8s ago
 Main PID: 4716 (java)
   CGroup: /system.slice/jenkins.service
           └─4716 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=%C/jenkins/war --httpPort=8080
Apr 13 06:52:51 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: d728307574b0475392e4993e47ca3bef
Apr 13 06:52:51 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
Apr 13 06:52:51 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: *************************************************************
Apr 13 06:52:51 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: *************************************************************
Apr 13 06:52:51 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: *************************************************************
Apr 13 06:53:24 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: 2023-04-13 06:53:24.316+0000 [id=29]        INFO        jenkins.InitReactorRunner...zation
Apr 13 06:53:24 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: 2023-04-13 06:53:24.352+0000 [id=23]        INFO        hudson.lifecycle.Lifecycl...unning
Apr 13 06:53:24 ip-172-31-0-179.ap-south-1.compute.internal systemd[1]: Started Jenkins Continuous Integration Server.
Apr 13 06:53:25 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: 2023-04-13 06:53:25.321+0000 [id=45]        INFO        h.m.DownloadService$Downl...taller
Apr 13 06:53:25 ip-172-31-0-179.ap-south-1.compute.internal jenkins[4716]: 2023-04-13 06:53:25.322+0000 [id=45]        INFO        hudson.util.Retrier#start...mpt #1
Hint: Some lines were ellipsized, use -l to show in full.
[root@ip-172-31-0-179 ~]#
```



## Install Jenkins
 You can install jenkins using the rpm or by setting up the repo. We will set up the repo so that we can update it easily in the future.
1. Get the latest version of jenkins from https://pkg.jenkins.io/redhat-stable/ and install
   ```sh
   sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
   sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
   amazon-linux-extras install epel 
   amazon-linux-extras install java-openjdk11  
   
   #on RedHat/CentOs 
   #yum install epel-release # repository that provides 'daemonize'
   #yum install java-11-openjdk-devel
   #yum install jenkins
   ```

   ### Start Jenkins
   ```sh
   # Start jenkins service
   service jenkins start

   # Setup Jenkins to start at boot,
   chkconfig jenkins on
   ```

   ### Accessing Jenkins
   By default jenkins runs at port `8080`, You can access jenkins at
   ```sh
   http://YOUR-SERVER-PUBLIC-IP:8080
   ```
  #### Configure Jenkins
- The default Username is `admin`
- Grab the default password 
- Password Location:`/var/lib/jenkins/secrets/initialAdminPassword`
- `Skip` Plugin Installation; _We can do it later_
- Change admin password
   - `Admin` > `Configure` > `Password`
- Configure `java` path
  - `Manage Jenkins` > `Global Tool Configuration` > `JDK`  
- Create another admin user id

### Test Jenkins Jobs
1. Create “new item”
1. Enter an item name – `My-First-Project`
   - Chose `Freestyle` project
1. Under the Build section
	Execute shell: echo "Welcome to Jenkins Demo"
1. Save your job 
1. Build job
1. Check "console output"
