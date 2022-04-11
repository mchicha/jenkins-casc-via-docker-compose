# Set a Jenkins server with CasC(Configuration as Code) plugin via Docker-Compose

   ### Step 3 — Specifying the Jenkins URL (git branch: 3-specify-jenkins-url)

    The Jenkins URL is a URL for the Jenkins instance that is routable from the devices that need to access it. For example, if we’re deploying Jenkins as a node inside a private network, the Jenkins URL may be a private IP address, or a DNS name that is resolvable using a private DNS server. 

    We can set the Jenkins URL on the web interface by navigating to server_ip:8080/configure and entering the value in the Jenkins URL field under the Jenkins Location heading. Here’s how to achieve the same using the Configuration as Code plugin:

    1. Define the desired configuration of our Jenkins instance inside a declarative configuration file (which we’ll call casc.yaml).
    2. Copy the configuration file into the Docker image (just as you did for your plugins.txt file).
    3. Set the CASC_JENKINS_CONFIG environment variable to the path of the configuration file to instruct the Configuration as Code plugin to read it.

    1. create a new file named casc.yaml:
    nano casc.yaml

    Then, add in the following lines:
    unclassified:
        location:
            url: http://172.31.7.79:8080/ (172.31.7.79 is the private ip)

    unclassified.location.url is the path for setting the Jenkins URL. It is just one of a myriad of properties that can be set with JCasC. Valid properties are determined by the plugins that are installed.
    To see what properties are available, navigate to server_ip:8080/configuration-as-code/reference.

    2. open the Dockerfile file:
    nano Dockerfile

    Add a COPY instruction to the end of our Dockerfile that copies the casc.yaml file into the image at /var/jenkins_home/casc.yaml. We’ve chosen /var/jenkins_home/ because that’s the default directory where Jenkins stores all of its data:

    COPY casc.yaml /var/jenkins_home/casc.yaml

    3. Then. add a further ENV instruction that sets the CASC_JENKINS_CONFIG environment variable:

    ENV CASC_JENKINS_CONFIG /var/jenkins_home/casc.yaml

   ![Dockerfile-updated!](Images/Dockerfile-updated.jpg)

    Next, build the image:

    docker build -t jenkins:jcasc .

    And run the updated Jenkins image:

    docker run --name jenkins --rm -p 8080:8080 jenkins:jcasc

    As soon as the Jenkins is fully up and running log line appears, navigate to server_ip:8080 to view the dashboard. This time, we may have noticed the warning about the Jenkins URL has disappeared.

   ![browser-warnings!](Images/browser-warnings.jpg)


    Now, navigate to server_ip:8080/configure and scroll down to the Jenkins URL field. Check that the Jenkins URL has been set to the same value specified in the casc.yaml file.
   ![jenkins-location!](Images/jenkins-location.jpg) 

    In this step, we used the Configuration as Code plugin to set the Jenkins URL. In the next step, we will tackle the second issue from the notifications list ("the Jenkins is currently unsecured" message).
   
    