## WEBAPPS WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION and CONTINOUS DEPLOYMENT. 
### DevOps Automation

DevOps automation refers to the use of tools, processes, and systems to automate various aspects of software development and IT operations. The goal of DevOps automation is to improve efficiency, speed up the software delivery process, and ensure consistent and reliable operations. Some examples of DevOps automation include continuous integration and delivery (CI/CD), infrastructure as code (IaC), configuration management, and monitoring and logging.


### INTRODUCTION TO APPLICATION SERVER(Ansible) AND PRODUCTION SERVER

#### Task
**Build,Conternarize and Lunch the Webapps site through a CICI pipline. This pipeline consists of a Workstation Server pulling the website build from git to the Application were it is containarized by docker to Production with Ansible** 

SERVER REQUIRMENTS:
- JAVA
- TOMCAT
- DOCKER
- PYTHON
- ANSIBLE


![Screenshot 2023-02-08 012722](https://user-images.githubusercontent.com/101978292/217397916-0c21b85c-73fa-449a-b23a-a810280d2221.jpg)


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
- unzip the file just downloaded to your server using `tar -zxvf`
```
 tar -zxvf apache-tomcat-10.0.27.tar.gz
 

```

- Copy the unzipped file to `/opt`

```
sudo cp -r apache-tomcat-10.0.27 /opt

```
3. Go to the folder `sudo cd /opt/apache-tomcat-10.0.27/bin`
were tomcat is installed and start it with: `./startup.sh`

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
#### Step 4- Docker Installation

1. Install Docker
```
sudo dnf check-update

sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo dnf install docker-ce docker-ce-cli containerd.io

sudo systemctl start docker
sudo systemctl status docker

```

2. Docker is a group, so go to groups
```
cat /etc/group
```
3. Add the ec2-user to the group `docker`

```
sudo usermod -aG docker ec2-user

groups ec2-user

systemd-journal docker
sudo systemctl restart docker
sudo systemctl enable docker
sudo systemctl status docker

```
4. To use a docker, we need 3 things
- create a container[that is the docker that we installed]
- pull the latest tomcat image from the docker hub into docker
-  we move the image into that our container


5. Create Dockerfile 

```
touch Dockerfile

```
Add this to the Dockerfile `sudo vi Dockerfile`

```
FROM tomcat:latest 

#pull the latest tomcat image

COPY ./webapp.war /usr/local/tomcat/webapps

#copy webapp.war from homepage to the container
#Copy is a container command

RUN cp -r /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps

#copy all the files in webapps.dist file in the container to webapps in the container bcos the webapps of the docker container always comes empty

```
*right now you can run the commands for creating an image  `sudo docker build -t iternimage .`  and container: `sudo docker run -d --name IternCon -p 8080:8080 Internimage` manually on the server but we are going to put it in jenkins for it to be automated. Also this is the code to combine the command of pulling the latest image from docker hub into docker then create a container and move the image into the container `docker run - d --name IternCon -p 8080:8080 tomcat:latest`*


#### Step 5-  Ansible Installation
Note: There are 4 files ansibl will need on this server for it to work
- Dockerfile (file containing the commands)
- webapp.war (your build code)
- hosts (containing the ipaddr of all deployment servers)
- playbook (yaml file)

1. Update the server 

```
sudo yum update -y
```
2. Install pip package for check if there is python with this `python3 -m pip -V` if not found run the commands :

```


curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

python3 get-pip.py --user

```

3. Install python
*ansible needs python to work*

```
python3 -m pip install --user ansible


```
4. Create host file

```
sudo vi hosts
```
5. add the ipaddresses of the servers you want to deploy too

```
localhost

172.31.59.87

```

6. Create yaml file 

```
sudo vi devops.yml
```

7. put this script in the devops.yaml file

```
---
- hosts: all
  tasks:
  - name: copy Dockerfile into Remote machine
    copy:
     src: Dockerfile
     dest: .
  - name: copy .war file into Remote machine
    copy:
     src: webapp.war
     dest: .
  - name: stop the running container
    command: docker stop devopscont
    ignore_errors: True
  - name: remove the running container
    command: docker rm devopscont
    ignore_errors: True
  - name: remove the running image
    command: docker rmi devopsimage
    ignore_errors: True
  - name: create devopsimage from Dockerfile
    command: docker build -t devopsimage .
  - name: create and run container
    command: docker run -d -p 8080:8080 --name devopscont  devopsimage


```

8. Install  netstat to enable you check what process is using your server

```
sudo yum install net-tools 
```

*You can use yaml validator to valid your yaml script : https://jsonformatter.org/yaml-validator*

#### Step 6-  Connect Ansible in AppServer to ProdServer 

8. Ssh into the production server from the app server
*it will request for the prodserver password*

```

ssh 172.31.59.87

```

9. To remove the password request. 
Generate keygen and copy to the production server, this enables easy access between production an application server
*ensure you are signed in access ec2-user to create the ssh keygen and not as root*
Follow the promote and keep pressing enter, do not add a paraphrase because we do not want to use password

```
ssh-keygen
```
10. Now copy the id into the server we went to have ssh access, which is to the production server

```
ssh-copy-id 172.31.59.87
```
*it will ask of the password to the server `book1`, no of users `1`*

11. Now ssh into the server this will esterblished a perminet connection between our application server and production server *now it wont request for a password again*

```
ssh 172.31.59.87
```
 12. Run the playbook in application server to production server
 *make sure the docker installed on prod server is also running, also make sure to service is using java on the prod server as well, also make sure tomcat is turned-on on the prodserver*

```
ansible-playbook -i hosts devops.yml --limit 172.31.59.87

```

#### Step 6- pubish over ssh with Ansible to Production

*note: when building your image and container, everything must be in lower case*

1. Go to global configuration and add the server information for publish over ssh

```
 SSH Servers name: app server
Hostname: 172.31.48.168
Username: ec2-user
remote directory: .
Passphrase / Password: book1
```
then go to Project1B Configuration *POST BUILD ACTION*

```
pubish over ssh to app server to Tomcat
project1b

[source files:] 
webapp/target/webapp.war

[Remove prefix:] 
webapp/target

[Remote directory]
.

[Exec command:]
  ansible-playbook -i hosts devops.yml --limit 172.31.59.87

```

#### Step 7-  Configure Jenkins with the declared variables details. 

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

2. Go to Jenkins web console, click **New Item** and clone Project1 or create a **Freestyle project** name it Project1d and click OK
3. Connect your GitHub repository, copy the repository URL from the repository
4. In configuration of your Jenkins freestyle project under Source Code Management select **Git repository**, provide there the link to your Itern GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![9](https://user-images.githubusercontent.com/101978292/217399139-b719a731-55f6-4340-8ec3-f4ba79957d2b.jpg)

![10](https://user-images.githubusercontent.com/101978292/217399209-47c65df8-2963-47a9-a0cc-89e54727ad36.jpg)

5. Click **Configure** your job/project and add and save these two configurations:

``` 

Under **Post Build Actions** select Archieve the artifacts and enter `**` in the text box.
```

6. Save the configuration and let us try to run the build. For now we can only do it manually.
7. Click **Build Now** button, if you have configured everything correctly, the build will be successfull and you will see it under **#4**
8. Open the build and check in **Console Output** if it has run successfully.


![12](https://user-images.githubusercontent.com/101978292/217399432-7691320a-902d-4302-96a0-616596a9acd5.jpg)




9. By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/webapps.war_github/builds/<build_number>/archive/`

10. go to the server and check for the  build 
 
`docker exec -it IternCon /bin/bash` 

#### Miscellaneous
- If it showed our port 8080 is already in use

- how do we detect what service is using our port 8080, *its usually java, in this case it shows the tomcat installed in the server which is still a java process*

```
sudo netstat -tnlp | grep :8080
```

- kill the service using the process id 

```
sudo kill -9 6424

```
- to remove docker images and container
docker ps
sudo docker stop interncon
sudo docker rm interncon
sudo docker rmi internimage
sudo netstat -tnlp | grep :8080