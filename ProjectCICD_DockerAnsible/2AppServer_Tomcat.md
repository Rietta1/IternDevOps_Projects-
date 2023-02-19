## WEBAPPS WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION and CONTINOUS DEPLOYMENT. 
### DevOps Automation : Application Server


In DevOps, application servers are a key component of the software delivery process, providing a platform for deploying and running applications. An application server is a type of server that is specifically designed to host and manage web-based applications. Application servers provide a platform for deploying web-based applications, allowing DevOps teams to easily manage and update applications throughout the development and deployment lifecycle. It can be integrated with other DevOps tools, such as CI/CD pipelines, to streamline the deployment process and ensure that applications are released quickly and with minimal errors.


### INTRODUCTION TO APPLICATION SERVER(TOMCAT)

#### Task
Build and Lunch the Webapps site through a CICI pipline. This pipeline consists of a Workstation Server pulling the website from git to the Application Tomcat Server which has tomcat installed on it


SERVER REQUIRMENTS:
- Java
- Tomcat



#### Step 1- Setup Server and create a user password
1. Create an AWS EC2 server based on Redhat9 Server and name it "App Server"
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
![1](https://user-images.githubusercontent.com/101978292/219957126-c347d0e4-e1c6-4a23-bf60-64e084d48981.jpg)


- Authenticate the password by editting the configuration file `sshd_config` and change the `PasswordAthentication` to yes 

```
 vi /etc/ssh/sshd_config

```
- Restart the sshd file
```
systemctl restart sshd
systemctl enable sshd
systemctl status sshd

```
- switch back to ec2-user
```
su ec2-user
```
#### Step 2- Install JDK (Java development kit) since 

1. Jenkins is a Java-based application:

```
sudo yum install java-1.8.0-openjdk-devel -y
```

#### Step 3- Install Tomcat
1. Install Wget, since redhat9 doesnt seem to have it installed `sudo yum install wget`

2.  Download Tomcat compressed file
Go to this website   ,copy the link to tar.gz and paste it on your server with sudo wget infront of it, 
```
 sudo wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.0.27/bin/apache-tomcat-10.0.27.tar.gz

ls

```
![2](https://user-images.githubusercontent.com/101978292/219957389-4bd943ef-6186-4b80-98fd-b068efe061d1.jpg)


- unzip the file just downloaded to your server using `tar -zxvf`
```
 tar -zxvf apache-tomcat-10.0.27.tar.gz
 
```
![2](https://user-images.githubusercontent.com/101978292/219960759-4c93f081-39cc-4514-a89f-d8e637418020.jpg)



- Copy the unzipped file to `/opt`

```
sudo cp -r apache-tomcat-10.0.27 /opt

```
3. Go to the folder `cd /opt/cd apache-tomcat-10.0.27/bin`
were tomcat is installed and start it with: `./startup.sh`

![4](https://user-images.githubusercontent.com/101978292/219957305-5a784cd4-1fc0-461f-b22f-f8fa412e22a1.jpg)


4. Declare varables of java and Tomcat, for jenkins to easily access when we run the pipeline and build. Open `.bash_profile`

```
sudo vi.bash_profile

```
- Paste this delarations underthe line 

```
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
CATALINA_HOME=/opt/apache-tomcat-10.0.27/
CATALINA_BASE=/opt/apache-tomcat-10.0.27/


PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME:$CATALINA_HOME:$CATALINA_BASE


export PATH

```

![3](https://user-images.githubusercontent.com/101978292/219957240-8b5f7413-40a2-480c-bd92-a00cd2d7b23e.jpg)

5. Go the the <ipaddr:8080> and see if the tomcat is running


![5](https://user-images.githubusercontent.com/101978292/219957553-1ba30b6f-ea6b-484d-a100-a9010bb0e302.jpg)


#### Step 4- pubish over ssh to tomcat 


1. Go to jenkins global configuration and add the server information for publish over ssh

```
 SSH Servers name: app server
Hostname: 172.31.48.168
Username: ec2-user
remote directory: .
Passphrase / Password: book1
```
then go to Project1b Configuration *POST BUILD ACTION*

```

![1](https://user-images.githubusercontent.com/101978292/219957595-32e20d33-b284-4801-9ac3-bad666e77b70.jpg)


![2](https://user-images.githubusercontent.com/101978292/219957707-bf49b188-a09b-4489-b545-338e19dd1074.jpg)



pubish over ssh to app server to Tomcat
project1b

[source files:] 
webapp/target/webapp.war

[Remove prefix:] 
webapp/target

[Remote directory]
.

[Exec command:]
 sudo cp ./webapp.war /opt/apache-tomcat-10.0.27/webapps
```

![6](https://user-images.githubusercontent.com/101978292/219957789-f1e84ed5-0d16-47ef-8858-a95d7908e4bb.jpg)

![7](https://user-images.githubusercontent.com/101978292/219957754-cbadcde7-e0f9-452d-a693-3084db78df94.jpg)


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

2. Go to Jenkins web console, click **New Item** and clone Project1 or create a **Freestyle project** name it Project1b and click OK

![4](https://user-images.githubusercontent.com/101978292/219958929-032af221-c23e-42a5-a27d-f4b83dbabf67.jpg)

![5](https://user-images.githubusercontent.com/101978292/219958962-c106f8a8-5125-4889-9d82-8175d2583b55.jpg)


3. Connect your GitHub repository, copy the repository URL from the repository

![19](https://user-images.githubusercontent.com/101978292/219959603-83f715d3-8c04-4cab-949a-27c775c414a8.jpg)

![20](https://user-images.githubusercontent.com/101978292/219959617-b46eba16-3314-4007-b757-f8a6131ca6b1.jpg)


4. In configuration of your Jenkins freestyle project under Source Code Management select **Git repository**, provide there the link to your Itern GitHub repository and credentials (user/password) so Jenkins could access files in the repository.


![19](https://user-images.githubusercontent.com/101978292/219959178-26e7ce50-be31-4174-9edb-98c199fb93ae.jpg)


![20](https://user-images.githubusercontent.com/101978292/219959185-46b02123-4fc8-42ae-b5a0-831944cfee91.jpg)


5. Click **Configure** your job/project and add and save these two configurations:

``` 


6. Save the configuration and let us try to run the build. 
7. Click **Build Now** button, if you have configured everything correctly, the build will be successfull and you will see it under **#2**
8. Open the build and check in **Console Output** if it has run successfully.


![12](https://user-images.githubusercontent.com/101978292/217399432-7691320a-902d-4302-96a0-616596a9acd5.jpg)




9. *By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/webapps.war_github/builds/<build_number>/archive/`*

10. go to the server and check for the  build 
 
`cd /opt/apache-tomcat-10.0.27/webapps` and find the webapp.war file in the workspace directory then cd into target, 

![8](https://user-images.githubusercontent.com/101978292/219958369-7de4dbd3-ad83-489f-bd76-098378953e9b.jpg)

![9](https://user-images.githubusercontent.com/101978292/219958887-e188333e-db9f-448c-a3f5-4d3ec9f2f2c9.jpg)
