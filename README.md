# Set a Jenkins server with CasC(Configuration as Code) plugin via Docker-Compose

  ## 1. Launch an EC2 instance for Docker server With internet access

   Security Group with Port '8080' open for internet
   ![Security-group!](Images/security-group.jpg)

  ## 2. Connect to the ec2 machine(Amazon Linux ec2 machine) via git bash
   ![connect-to-ec2-machine!](Images/connect-to-ec2-machine.jpg)

  ## 3. Change hostname of the ec2 machine to jenkins-server-on-docker

        . sudo su -
        . hostname jenkins-server-on-docker
        . sudo su -

   ![change-hostname!](Images/change-hostname.jpg)

  ## 4. Install docker on EC2 instance
    
        . yum install docker -y
        . docker --version

   ![install-docker!](Images/install-docker.jpg)
   
   ## 5. Start docker services

        . service docker start
        . service docker status

   ![service-docker-start!](Images/service-docker-start.jpg)

   ## 6. Install docker-compose

        Download and install:
        . sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

        Fix permissions after download:
        . sudo chmod +x /usr/local/bin/docker-compose

        Check version:
        . docker-compose version

   ![install-docker-compose!](Images/install-docker-compose.jpg)

   ## 7. How to automate Jenkins setup with Docker-compose and Jenkins configuration as code?
    This involves the following steps:

        1. Disabling the Setup Wizard
        2. Installing Jenkins Plugins
        3. Specifying the Jenkins URL
        4. Creating a User
        5. Setting Up Authorization
        6. Setting Up Build Authorization

   ### Step 1 — Disabling the Setup Wizard (branch: 1-disabling-setup-wizard)
    Using JCasC eliminates the need to show the setup wizard; therefore, in this first step, you’ll create a modified version of the official jenkins/jenkins image that has the setup wizard disabled. You will do this by creating a Dockerfile and building a custom Jenkins image from it.

    The jenkins/jenkins image allows you to enable or disable the setup wizard by passing in a system property named jenkins.install.runSetupWizard via the JAVA_OPTS environment variable. Users of the image can pass in the JAVA_OPTS environment variable at runtime using the --env flag to docker run. However, this approach would put the onus of disabling the setup wizard on the user of the image. Instead, you should disable the setup wizard at build time, so that the setup wizard is disabled by default.

    You can achieve this by creating a Dockerfile and using the ENV instruction to set the JAVA_OPTS environment variable.

    First, create a new directory inside your server to store the files

    . mkdir jcasc

    Navigate inside that directory:
    . cd jcasc

    Create the file named Dockerfile:
    . nano Dockerfile
   ![mkdir-jcasc!](Images/mkdir-jcasc.jpg)

    Copy the following content into the Dockerfile:
    
        FROM jenkins/jenkins:latest
        ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

    We’re using the FROM instruction to specify jenkins/jenkins:latest as the base image, and the ENV instruction to set the JAVA_OPTS environment variable. 

    Build a Docker image and assign it a unique tag (we’ll use jcasc here):

    docker build -t jenkins:jcasc .
   ![build-jcasc-image!](Images/build-jcasc-image.jpg)

    Check the docker images created with the comand:
    docker images
   ![docker-images!](Images/docker-images.jpg)

    Create a container with the custom image by running docker run:
    docker run --name jenkins_ctr --rm -p 8080:8080 jenkins:jcasc

    We used the --name jenkins_ctr option to give the container an easy-to-remember name; We also specified the --rm flag so the container will automatically be removed after you’ve stopped the container process. Lastly, you’ve configured your server host’s port 8080 to proxy to the container’s port 8080 using the -p flag; 8080 is the default port where the Jenkins web UI is served from.

    Jenkins will take a short period of time to initiate. When Jenkins is ready, you will see the following line in the output:
   ![output-run-docker!](Images/output-run-docker.jpg)

    Now, open up the browser to server_ip:8080. we’re immediately shown the dashboard without the setup wizard.
   ![browser!](Images/browser.jpg)

    We have just confirmed that the setup wizard has been disabled. To clean up, stop the container by pressing CTRL+C. If you’ve specified the --rm flag earlier, the stopped container would automatically be removed.

    In this step, we’ve created a custom Jenkins image that has the setup wizard disabled. However, the top right of the web interface now shows a red notification icon indicating there are issues with the setup. Click on the icon to see the details.
   ![jenkins-warning!](Images/jenkins-warning.jpg)
   ![jenkins-warning-2!](Images/jenkins-warning-2.jpg)

    The first warning informs us that we have not configured the Jenkins URL. The second tells us that we haven’t configured any authentication and authorization schemes, and that anonymous users have full permissions to perform all actions on your Jenkins instance. Previously, the setup wizard guided us through addressing these issues. Now that we’ve disabled it, you need to replicate the same functions using JCasC. we will modify our Dockerfile and JCasC configuration until no more issues remain (that is, until the red notification icon disappears).

    In the next step, we will begin that process by pre-installing a selection of Jenkins plugins, including the Configuration as Code plugin, into our custom Jenkins image.