
#### Task
**Modify the Project 8 architecture by adding a Jenkins server and configuring a task to automatically publish source code updates from Git to an NFS server.**



#### Step 1 - Configure Jenkins to retrieve source codes from GitHub using Webhooks
Here you configure a simple Jenkins job/project. This job will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings:

```
Go to the 
itern repository
Click on settings
Click on webhooks on the left panel
On the webhooks page under Payload URL enter: http:// Jenkins server IP address/github-webhook/
Under content type select: application/json
Then add webhook
```
![j1](https://user-images.githubusercontent.com/101978292/219968490-b165c5f3-88ab-414d-a4b2-2c1aafb0fee7.jpg)



2. Go to Jenkins web console, click **New Item** and create a **Freestyle project** and click OK

3. Connect your GitHub repository, copy the repository URL from the repository

4. In configuration of your Jenkins freestyle project under Source Code Management select **Git repository**, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![j2](https://user-images.githubusercontent.com/101978292/219968525-c8652b8b-b53a-4aaa-9404-798c3f149777.jpg)


![10](https://user-images.githubusercontent.com/101978292/217399209-47c65df8-2963-47a9-a0cc-89e54727ad36.jpg)


5. Save the configuration and let us try to run the build. For now we can only do it manually.


6. Click **Configure** your project1d and add save this configuration:

``` 
Under **Build triggers** select: Github trigger for GITScm polling

```

<<<<<<< HEAD
7. Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the main branch.

8. You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.
=======
![j4](https://user-images.githubusercontent.com/101978292/219968540-817e0c1c-5ec1-4420-b352-7c1a2cd9f1f4.jpg)

![j5](https://user-images.githubusercontent.com/101978292/219968545-e6ec561d-79a0-4a1a-95ac-23e67b20b8a5.jpg)

9. Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the main branch.
10. You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.
>>>>>>> 3cca7806ad6a610541527442813bcb5fdd35b439




9. By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`


#### Step 2 – Configure Jenkins to copy files to Applicatin and Production server via SSH

**Publish over SSh**

1. Configure the job/project to copy artifacts over to APP server.

2. On main dashboard select **Manage Jenkins** and choose **Configure System** menu item.

3. Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:


```

Name- appserver
Hostname – can be private IP address of your NFS server
Username – ec2-user 
Remote directory: . 
Passphrase (the password of the server)

```

![j6](https://user-images.githubusercontent.com/101978292/219968611-8bfa3a80-de8e-4228-9fec-6e8421e45ed5.jpg)

<<<<<<< HEAD
4. Test the configuration and make sure the connection returns **Success** Remember, that TCP port 22 on NFS server must be open to receive SSH connections.
=======

12. Test the configuration and make sure the connection returns **Success** Remember, that TCP port 22 on App and Prod server must be open to receive SSH connections.
>>>>>>> 3cca7806ad6a610541527442813bcb5fdd35b439

![18](https://user-images.githubusercontent.com/101978292/217401048-80806dce-8415-408f-866d-71166ad1a105.jpg)




5. Save this configuration and go ahead, change something in **README.MD** file the GitHub Tooling repository.

6. Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

```
SSH: Transferred 25 file(s)
Finished: SUCCESS

```


![d4](https://user-images.githubusercontent.com/101978292/219968729-ca8f3d3d-71c2-48d3-b704-f7be0a5fd38f.jpg)



