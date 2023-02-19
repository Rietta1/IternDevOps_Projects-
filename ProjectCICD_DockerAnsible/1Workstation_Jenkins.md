## WEBAPPS WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION and CONTINOUS DEPLOYMENT. 
### DevOps Automation

DevOps automation refers to the practice of using automation tools and techniques to streamline and optimize the processes involved in software development, testing, and deployment. 

DevOps automation involves using tools such as Continuous Integration and Continuous Deployment (CI/CD) pipelines, configuration management tools, and infrastructure as code (IaC) to automate the entire software delivery process. This can include automating the build, testing, and deployment of applications, as well as the provisioning and management of infrastructure.

By automating these processes, DevOps teams can reduce the manual effort and errors involved in software development, accelerate the release of new features and updates, and improve the quality and stability of their applications. It also enables teams to easily scale their development and infrastructure operations to meet changing business requirements.

### Introduction to Workstation servers 

Your workstation server are used to build and test software applications, allowing DevOps teams to identify and resolve bugs and issues before deployment. It is used to manage infrastructure as code, allowing DevOps teams to automate the build, deployment and management of infrastructure components such as servers, networks, and storage.


#### Task
Build and Lunch the Webapps site through a CICI pipline. This pipeline consists of a Workstation Server, Application Server and a Production server, by which the task is configured automatically to publish source code updates from GIT to Application Server and then to the Production Server.

**GENERAL CICD REQUIRMENTS**
Create 3 Redhat servers namely:
workstation server,
application server,
production server,

**WORKSERVER REQUIRMENTS:**
- Java
- Git
- Wget
- Maven
- Jenkins



#### Step 1- Create Password for User
1. Create an AWS EC2 server based on Redhat9 Server and name it "Workstation Server"
2. Update your server

```
sudo yum update -y
```
3. Create a password for the user on the server
```
sudo su
passwd ec2-user 
(book1)
```
- Authenticate the password by editting the configuration file `sshd_config` and change the `PasswordAthentication` to yes 

```
 vi /etc/ssh/sshd_config

```
![1](https://user-images.githubusercontent.com/101978292/219952626-245c97b9-bb23-4a3c-9b9f-4ec17629efb7.jpg)


- Restart the sshd file
```
systemctl restart sshd
systemctl enable sshd
systemctl status sshd

```

![2](https://user-images.githubusercontent.com/101978292/219952645-19f90b92-516a-413f-b569-184aa3873dfe.jpg)


- switch back to ec2-user
```
su ec2-user
```
#### Step 2- Install JDK (Java development kit) since 

1. Jenkins is a Java-based application:

```
sudo yum install java-1.8.0-openjdk-devel -y
```

#### Step 3- Install Maven
1. Install Wget, since redhat9 doesnt seem to have it installed `sudo yum install wget`

2.  Download Maven compressed file
Go to this website   ,copy the link to tar.gz and paste it on your server with sudo wget infront of it, 
```
 sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.0/binaries/apache-maven-3.9.0-bin.tar.gz

ls

```
- unzip the file just downloaded to your server using `tar -zxvf`
```
tar -zxvf  apache-maven-3.9.0-bin.tar.gz

```
- install maven
```
sudo yum install maven -y

```
-Copy the unzipped file to `/opt`

```
sudo cp -r apache-maven-3.9.0 /opt

```
3. Declare varables of java and Maven, for jenkins to easily access when we run the pipeline and build. Open `.bash_profile`

```
sudo vi .bash_profile

```
Paste this delarations underthe line 

```
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
MAVEN_HOME=/opt/apache-maven-3.9.0/
M2=/opt/apache-maven-3.9.0/


PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME:$MAVEN_HOME:$M2


export PATH

```
![3](https://user-images.githubusercontent.com/101978292/219952746-968d8790-2684-4ea9-b2df-dd33aae87605.jpg)


#### Step 4- Install Git
1. install git
```
sudo yum install git -y
```

2. Add your git profile to your server
```
git config --global user.name "Rietta1"
git config --global user.email "yourgitemail@gmail.com"

```

3. Create a directory for your git and initalize it

```
git init Project1

cd Project1

```

![4](https://user-images.githubusercontent.com/101978292/219952852-dfce900a-a9c2-4d3f-acfe-2d9f81b70e43.jpg)


4. Add the repository you are to pull code changes from and build

```
git remote add origin https://github.com/Rietta1/itern.git
git clone https://github.com/Rietta1/itern.git

```

5. Then list `ll` and go into the repo installed `cd itern`

#### Step 4- Install Jenkins 
1. The installations usually change, you can use this link to cross check: https://www.jenkins.io/doc/book/installing/linux/

```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

sudo yum upgrade

sudo yum install jenkins
sudo systemctl daemon-reload

```


2. Make sure Jenkins is up and running: 


```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

```



![5](https://user-images.githubusercontent.com/101978292/219952920-877a8f87-6746-49f7-b6ef-5d0863ae3eb6.jpg)


3. By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
4. Next, setup Jenkins. From your browser access `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080` You will be prompted to provide a default admin password


![6](https://user-images.githubusercontent.com/101978292/219952990-8b10abf6-57cb-440a-bebc-87d9657c6a74.jpg)


5. Retrieve the password from your Jenkins server: 
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```

![7](https://user-images.githubusercontent.com/101978292/219953009-588640f4-480f-4889-a684-d4b86e06b72b.jpg)


6. Copy the password from the server and paste on Jenkins setup to unlock Jenkins.
7. Next, you will be prompted to install plugins – **choose install suggested plugins**


![8](https://user-images.githubusercontent.com/101978292/219953213-9bd2c42a-f2c7-4522-b1e8-2c421545f2ed.jpg)

![10](https://user-images.githubusercontent.com/101978292/219953219-fe19725a-f649-4911-811f-cb01e5d982e8.jpg)

8. Once plugins installation is done – create an admin user and you will get your Jenkins server address. **The installation is completed!**


![11](https://user-images.githubusercontent.com/101978292/219953233-cb7671f6-efd9-4242-a8c2-b24e74d8585e.jpg)

![12](https://user-images.githubusercontent.com/101978292/219953258-667f48bf-d2d1-49de-94d4-dbc819fe2f9b.jpg)

9. Go to Plugin manager and install 

![15](https://user-images.githubusercontent.com/101978292/219954716-3826f4c7-6940-45f1-9bfd-5bc23297c489.jpg)


#### Step 5-  Configure Jenkins with the declared variables details. 

1. Go to Setup Global Tool Configuration and on the installations part, add

```
add java  
name :JAVA_HOME

JAVA_HOME: /usr/lib/jvm/java-1.8.0-openjdk

add git
name: git
Path to Git executable: /usr/bin/git

add maven
name:MAVEN_HOME
MAVEN_HOME:/opt/apache-maven-3.9.0/

```

![16](https://user-images.githubusercontent.com/101978292/219953537-1f696dcc-4ab6-4ee8-8052-1eff55a141ad.jpg)

![18](https://user-images.githubusercontent.com/101978292/219953548-f7c9f49a-28e3-4216-84fd-93c0ed542c62.jpg)

2. Go to Jenkins web console, click **New Item** and create a **Freestyle project** name it Project1 and click OK
3. Connect your GitHub repository, copy the repository URL from the repository
4. In configuration of your Jenkins freestyle project under Source Code Management select **Git repository**, provide there the link to your Itern GitHub repository and credentials (user/password) so Jenkins could access files in the repository.


![19](https://user-images.githubusercontent.com/101978292/219953601-dbf50281-270c-42f1-8b39-fd99aae161ff.jpg)

![20](https://user-images.githubusercontent.com/101978292/219955195-acf2a27d-9fba-40b1-8d9a-501e827a7a94.jpg)


5. Save the configuration and let us try to run the build. For now we can only do it manually.
6. Click **Build Now** button, if you have configured everything correctly, the build will be successfull and you will see it under **#1**
7. Open the build and check in **Console Output** if it has run successfully.


![21](https://user-images.githubusercontent.com/101978292/219953713-e1cffd0a-55a4-4980-9672-d1905c8d95ac.jpg)


8. Click **Configure** your job/project and add and save these two configurations:

``` 

Under **Post Build Actions** select Archieve the artifacts and enter `**` in the text box.
```

12. By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/webapps.war_github/builds/<build_number>/archive/`

13. 
go to the server and check for the  build 
 
`cd /var/lib/jenkins` and find the webapp.war file in the workspace directory then cd into target, 

![22](https://user-images.githubusercontent.com/101978292/219955536-b8926e92-38f8-41d8-98b3-ef3d6469700d.jpg)

![23](https://user-images.githubusercontent.com/101978292/219954474-c18a97d2-87e9-463f-a08b-f7377804db4a.jpg)

