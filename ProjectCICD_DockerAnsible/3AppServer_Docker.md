## WEBAPPS WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION and CONTINOUS DEPLOYMENT. 
### DevOps Automation : Application Server with Docker

Docker is a popular containerization technology that has become increasingly important in DevOps. When used together with an application server, Docker can provide a powerful platform for managing and deploying web-based applications in a DevOps environment. This Application servers is used to create containerized applications that can be deployed and managed using Docker. This approach can simplify the deployment process and make it easier to manage multiple versions of an application.

Docker containers can be used to isolate application dependencies, ensuring that each application has the required libraries and resources. This can simplify the deployment process and help avoid version conflicts.

Docker containers can be easily scaled up or down, depending on demand. This can help ensure that applications can handle varying levels of traffic and provide a good user experience.


### INTRODUCTION TO APPLICATION SERVER(DOCKER)

#### Task
Build,Conternarize and Lunch the Webapps site through a CICI pipline. This pipeline consists of a Workstation Server pulling the website from git to the Application Tomcat Server which which is then containerized by docker then lunched

SERVER REQUIRMENTS:
- Java
- Tomcat
- Docker


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



#### Step -  Manual configuration

*Ideally we use automated process which is publish over ssh, this is just for understanding the process purposes, check the next step for the automated process*
- ensure no service is using your port 8080

```
docker ps
```


- If it showed our port 8080 is already in use

- how do we detect what service is using our port 8080, *its usually java, in this case it shows the tomcat installed in the server which is still a java process*

```
sudo netstat -tnlp | grep :8080
```

- kill the service using the process id 

```
sudo kill -9 6424

```


- check the images on your server

```
sudo docker images

```
- remove any container if there is any

```
docker ps
sudo docker stop 97f970b9f6di
sudo docker rm 97f970b9f6di
sudo netstat -tnlp | grep :8080

```


- pull the latest image into docker

```
sudo docker pull tomcat:latest

```
OR

- create custom image

```
sudo docker build -t iternimage .

```

- put the image into a container
but we need the image id or image name
*so what are the images i have in my docker* `sudo docker p`

- create a container and put this docker image inside the container
*were IternCon is the name we are giving to our container, -p means print*

```
sudo docker run -d --name IternCon -p 8080:8080 97f970b9f6di

OR

sudo docker run -d --name IternCon -p 8080:8080 internimage

```

- Check your list of containers

```
sudo docker container ls -la

```
NOTE : with one single command we can pull the latest image and put in into our container


- to do it remove the conatiner
*either you use the container id or the container name*

```
sudo docker stop IternCon
sudo docker rm IternCon
docker ps
sudo docker rmi 97f970b9f6di

```

- To combine the command of pulling the latest image from docker hub into docker then create a container and move the image into the container

```
docker run - d --name IternCon -p 8080:8080 tomcat:latest

docker ps

```
- Now manually copy the webapp.war file into our container manually

- first go into the container

```
docker exec -it containerID /bin/bash
```

- now u are in the container `ls -l`

- Since `webapps` in docker comes empty, you have to copy the contents of `webapps.dist` to `webapps` to be able to access the site on the browser.
RUN
- go into webapps.dist `cd webapps.dist`
- now copy everything in webapps.dist to webapps, since webapps in docker is usually empty

```
cp -r ./* /user/local/tomcat/webapps

```
- exit the container `exit`

- Now manually copy the webapp.war file into our container manually

```
sudo cd /opt/apache-tomcat-10.0.27/webapps

docker cp  webapp.war IternCon:/usr/local/tomcat/webapps

```

- check

```
docker exec -it IternCon /bin/bash

```


#### Step 6- pubish over ssh to Docker 


1. Go to configure systems and add the server information for publish over ssh

```
 SSH Servers name: app server
Hostname: 172.31.48.168
Username: ec2-user
remote directory: .
Passphrase / Password: book1

```
then go to and clone Project1B and name it Project1c Configuration *POST BUILD ACTION*

```
pubish over ssh to app server to Docker
project1b

[source files:] 
webapp/target/webapp.war

[Remove prefix:] 
webapp/target

[Remote directory]
.

[Exec command:]
  sudo docker build -t mattewimages
sudo docker run -d --name immacon -p 8080:8080 mattewimage

```

![d3](https://user-images.githubusercontent.com/101978292/219962166-13c64c5c-7029-4485-b7cf-48e99f5be8ec.jpg)


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

2. Go to Jenkins web console, click **New Item** and clone Project1 or create a **Freestyle project** name it Project1c and click OK


![5](https://user-images.githubusercontent.com/101978292/219958962-c106f8a8-5125-4889-9d82-8175d2583b55.jpg)


3. Connect your GitHub repository, copy the repository URL from the repository

4. In configuration of your Jenkins freestyle project under Source Code Management select **Git repository**, provide there the link to your Itern GitHub repository and credentials (user/password) so Jenkins could access files in the repository.


![19](https://user-images.githubusercontent.com/101978292/219959178-26e7ce50-be31-4174-9edb-98c199fb93ae.jpg)


![20](https://user-images.githubusercontent.com/101978292/219959185-46b02123-4fc8-42ae-b5a0-831944cfee91.jpg)


5. Click **Configure** your job/project and add and save these two configurations:
 


6. Save the configuration and let us try to run the build. 
7. Click **Build Now** button, if you have configured everything correctly, the build will be successfull and you will see it under **#3**
8. Open the build and check in **Console Output** if it has run successfully.




![d4](https://user-images.githubusercontent.com/101978292/219963075-5b25e5b2-fcf2-4682-8b84-c62279e8f143.jpg)


9. By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/webapps.war_github/builds/<build_number>/archive/`

10. go to the server and check for the  build 
 ```
 docker exec -it immacon /bin/bash
```


