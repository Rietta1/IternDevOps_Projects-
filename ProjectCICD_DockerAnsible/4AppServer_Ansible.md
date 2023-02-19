## WEBAPPS WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION and CONTINOUS DEPLOYMENT. 
### DevOps Automation: App Server with Ansible

Ansible is a popular automation tool used in DevOps to manage the deployment and configuration of software applications. When used together with an application server, Ansible can provide a powerful platform for automating the deployment and management of web-based applications. it is used to manage the configuration of applications running on an application server. This can include managing settings such as database connections, security settings, and application-specific configuration files.


### INTRODUCTION TO APPLICATION SERVER(Ansible) AND PRODUCTION SERVER

#### Task
Build,Conternarize and Lunch the Webapps site through a CICI pipline. This pipeline consists of a Workstation Server pulling the website build from git to the Application were it is containarized by docker to Production with Ansible

SERVER REQUIRMENTS:
- Java
- Tomcat
- Docker
- Python
- Ansible



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

![1](https://user-images.githubusercontent.com/101978292/219963839-6e55ae4f-097a-4389-af69-27640b14bcf6.jpg)


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
![1a](https://user-images.githubusercontent.com/101978292/219963891-6e49da36-1681-4564-a52d-072989ae6375.jpg)


9. To remove the password request. 
Generate keygen and copy to the production server, this enables easy access between production an application server
*ensure you are signed in access ec2-user to create the ssh keygen and not as root*
Follow the promote and keep pressing enter, do not add a paraphrase because we do not want to use password

```
ssh-keygen
```
![keygen](https://user-images.githubusercontent.com/101978292/219964075-d9787cde-b860-42dc-9944-f129ad871faf.jpg)


10. Now copy the id into the server we went to have ssh access, which is to the production server

```
ssh-copy-id 172.31.59.87
```

*it will ask of the password to the server `book1`, no of users `1`*

11. Now ssh into the server this will esterblished a perminet connection between our application server and production server *now it wont request for a password again*

```
ssh 172.31.59.87
```

![2](https://user-images.githubusercontent.com/101978292/219964013-eed23f65-5dac-4422-80dd-db7a6d297083.jpg)


 12. Run the playbook in application server to production server
 *make sure the docker installed on prod server is also running, also make sure to service is using java on the prod server as well, also make sure tomcat is turned-on on the prodserver*

```
ansible-playbook -i hosts devops.yml --limit 172.31.59.87

```

![3](https://user-images.githubusercontent.com/101978292/219964023-e1379bec-c8b6-466c-9e03-a35ec927aeea.jpg)


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
![1](https://user-images.githubusercontent.com/101978292/219964462-1bc998e5-f530-4a85-9588-6892564138c8.jpg)

![2](https://user-images.githubusercontent.com/101978292/219964473-01481ca5-3572-4764-9989-75849be89e4a.jpg)

![3](https://user-images.githubusercontent.com/101978292/219964512-d6221fb3-35c3-4fa4-9946-eadf4f02383f.jpg)


then go to Project1B Configuration *POST BUILD ACTION*

```
pubish over ssh to app server to Tomcat
project1d

[source files:] 
webapp/target/webapp.war

[Remove prefix:] 
webapp/target

[Remote directory]
.

[Exec command:]
  ansible-playbook -i hosts devops.yml --limit 172.31.59.87

```
![j6](https://user-images.githubusercontent.com/101978292/219964250-6277c937-8829-4153-bbe4-27c8cc1c69c6.jpg)


![5](https://user-images.githubusercontent.com/101978292/219964052-254fb063-e10a-4df9-bef2-64b44d7eddfa.jpg)


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


![5](https://user-images.githubusercontent.com/101978292/219958962-c106f8a8-5125-4889-9d82-8175d2583b55.jpg)


3. Connect your GitHub repository, copy the repository URL from the repository

4. In configuration of your Jenkins freestyle project under Source Code Management select **Git repository**, provide there the link to your Itern GitHub repository and credentials (user/password) so Jenkins could access files in the repository.




![20](https://user-images.githubusercontent.com/101978292/219959185-46b02123-4fc8-42ae-b5a0-831944cfee91.jpg)


5. Click **Configure** your job/project and add and save these two configurations:
 


6. Save the configuration and let us try to run the build. 
7. Click **Build Now** button, if you have configured everything correctly, the build will be successfull and you will see it under **#3**
8. Open the build and check in **Console Output** if it has run successfully.



![6](https://user-images.githubusercontent.com/101978292/219964147-a0dc4c5a-99c0-4e23-81e7-77dcf3199f5e.jpg)



9. By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/webapps.war_github/builds/<build_number>/archive/`

10. go to the server and check for the  build 
 
`docker exec -it IternCon /bin/bash` 

11. go to your <ipaddress:8080/webapps> and you will see your code running


![4](https://user-images.githubusercontent.com/101978292/219964158-cfb6d7a4-30e6-463c-b667-a9c9ed51b46a.jpg)

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
