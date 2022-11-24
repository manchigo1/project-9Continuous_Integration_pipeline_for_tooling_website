CI-CD-on-DevOps-Website-Solution

Introduction

Jenkins is an open-source Continuous Integration server written in Java for orchestrating a chain of actions to achieve the Continuous Integration process in an automated fashion. Jenkins supports the complete development life cycle of software from building, testing, documenting the software, deploying, and other stages of the software development life cycle.

jenkins_architecture

Here I introduce Jenkins to automate code delivery to the NFS server

Jenkins Web Architecture For CI Builds

my_jenkins_architecture

Installing Jenkins Server

Spun up a web server on AWS cloud and SSH into it.

Installing JDK which is an important Java based package required for Jenkins to run.

sudo apt update
sudo apt install default-jdk-headless
java_installation

![jdk](./images/jdk.png)

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins


sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
jenkins_installation

![jenkins status](./images/jenkins%20status.png)

Since Jenkins runs on default port 8080, open this port on the Security Group inbound rule of the jenkins server on AWS jenkins_rule

Jenkins is up and running, copy and paste jenkins server public ip address appended with port 8080 on a web server to gain access to the interactive console. <jenkins_server_public_ip_address>:8080 jenkins_install_success
![jenkins success](./images/jenkins%20success.png)


The admin password can be found in the '/var/lib/jenkins/secrets/initialAdminPassword' path on the server.

suggested_installation plugin_installation login_success

Attaching WebHook to Jenkins Server


On the github repository that contains application code, create a webhook to connect to the jenkins job. To create webhook, go to the settings tab on the github repo and click on webhooks. Webhook should look like this <public_ip_of_jenkins_server>:8080/github-webhook/ webhook_creation

![webhooks](./images/webhooks.png)

Creating Job and Configuring GIT Based Push Trigger

![freestyle jenkins](./images/freestyle%20jenkins.png)

On the jenkins server, create a new freestyle job creating_job

In configuration of the Jenkins freestyle job choose Git repository, provide there the link to the GitHub repository and credentials (user/password) so Jenkins could access files in the repository. Also specify the branch containing code

git_url_input specify_branch specify_credentials

Configuring Build Triggers

Specify the particular trigger to use for triggering the job. Click "Configure" on the jenkins job and add these two configurations

1. Configure triggering the job from GitHub webhook:

![github web](./images/github%20web.png)

adding_build_triggers

2. Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

![build jenkins](./images/build%20jenkins.png)

post_build_step

At this point, our architecture has pretty much been built, lets taste it by making a change on any file on the Github repository and then push it to see the triggered job git_push

The console output shows the created job and the successful build. In this case the code on Github was built into an artifact on our Jenkins server workspace. Find the artificat by checking the status tab of the completed job . new_build persisted_archive

Our created artifact can be found on our local terminal too at this path /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/ artifact_terminal
)

Configuring Jenkins To Copy Files(Artifact) to NFS Server
![mnt jenkins](./images/mnt%20jenkins.png)

To achieve this, we install the Publish Via SSH pluging on Jenkins. The plugin allows one to send newly created packages to a remote server and install them, start and stop services that the build may depend on and many other use cases.

On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it plugin

Configure the job to copy artifacts over to NFS server. On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to the NFS server:

Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
![publish over ssh](./images/publish%20over%20ssh.png)


Hostname – can be private IP address of NFS server 
Username – ec2-user (since NFS server is based on EC2 with RHEL 8) 
Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![jenkins success](./images/jenkins%20success.png)
setting_publish server_add



