# Set a Jenkins server with CasC(Configuration as Code) plugin via Docker-Compose

  ### Step 2 — Installing Jenkins Plugins (git branch: 2-install-jenkins-plugins)

    To use JCasC, we need to install the Configuration as Code plugin. Currently, no plugins are installed. we can check this by navigating to http://server_ip:8080/pluginManager/installed.
   ![list-plugins-installed!](Images/list-plugins-installed.jpg)

    In this step, we’re going to modify our Dockerfile to pre-install a selection of plugins, including the Configuration as Code plugin.

    To automate the plugin installation process, we can make use of an installation script that comes with the jenkins/jenkins Docker image. You can find it inside the container at /usr/local/bin/install-plugins.sh. To use it, we would need to:

    . Create a text file containing a list of plugins to install.
    . Copy it into the Docker image.
    . Run the install-plugins.sh script to install the plugins.

    1. Create a new file named plugins.txt:
    nano plugins.txt

    Then, add in the following newline-separated list of plugin names and versions (using the format <id>:<version>):

    build-timeout:latest
    cloudbees-folder:latest
    configuration-as-code:latest
    credentials-binding:latest
    email-ext:latest
    git:latest
    github-branch-source:latest
    gradle:latest
    ldap:latest
    mailer:latest
    pipeline-github-lib:latest
    pipeline-stage-view:latest
    ssh-slaves:latest
    timestamper:latest

    The list contains the Configuration as Code plugin, as well as all the plugins suggested by the setup wizard. For example, we have the Git plugin, which allows Jenkins to work with Git repositories; we also have the Pipeline plugin, which is actually a suite of plugins that allows we to define Jenkins jobs as code.

    2. Next, open up the Dockerfile file:
    nano Dockerfile

    In it, add a COPY instruction to copy the plugins.txt file into the /usr/share/jenkins/ref/ directory inside the image; this is where Jenkins normally looks for plugins. Then, include an additional RUN instruction to run the install-plugins.sh script:

    COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
    RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

    3. Then, build a new image using the revised Dockerfile:
    docker build -t jenkins:jcasc .
   ![build-jasc-image-with-plugins!](Images/build-jasc-image-with-plugins.jpg)
   ![build-jasc-image-with-plugins-2!](Images/build-jasc-image-with-plugins-2.jpg)

    This step involves downloading and installing many plugins into the image, and may take some time to run depending on your internet connection. 
   
    4. Once the plugins have finished installing, run the new Jenkins image:
    docker run --name jenkins --rm -p 8080:8080 jenkins:jcasc
   ![build-jasc-image-with-plugins-2!](Images/build-jasc-image-with-plugins-2.jpg)

    After the Jenkins is fully up and running message appears on stdout, navigate to server_ip:8080/pluginManager/installed to see a list of installed plugins. We will see a solid checkbox next to all the plugins you’ve specified inside plugins.txt, as well as a faded checkbox next to plugins, which are dependencies of those plugins.

   ![browser-plugins-installed!](Images/browser-plugins-installed.jpg)

    We can check that the Configuration As Code plugin is installed.

    In this step, we’ve installed all the suggested Jenkins plugins and the Configuration as Code plugin. we’re now ready to use JCasC to tackle the issues listed in the notification box. In the next step, you will fix the first issue, which warns you that the Jenkins root URL is empty.