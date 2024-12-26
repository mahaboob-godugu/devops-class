Run your Jenkins agent as a service
Create a directory called jenkins or jenkins-service in your home directory or anywhere else where you have access, for example /usr/local/jenkins-service. If the new directory does not belong to the current user home, give it the right owner and group after creation. For me, it would look like the following:

sudo mkdir -p /usr/local/jenkins-service
sudo chown jenkins /usr/local/jenkins-service

Move the agent.jar file that you downloaded earlier with the curl command to this directory.


mv agent.jar /usr/local/jenkins-service

Now (in /usr/local/jenkins-service) create a start-agent.sh file with the Jenkins java command we’ve seen earlier as the file’s content.

#!/bin/bash
cd /usr/local/jenkins-service
# Just in case we would have upgraded the controller, we need to make sure that the agent is using the latest version of the agent.jar
curl -sO http://my_ip:8080/jnlpJars/agent.jar
java -jar agent.jar -jnlpUrl http://my_ip:8080/computer/My%20New%20Ubuntu%2022%2E04%20Node%20with%20Java%20and%20Docker%20installed/jenkins-agent.jnlp -secret my_secret -workDir "/home/jenkins"
exit 0



Make the script executable by executing chmod +x start-agent.sh in the directory.

Now create a /etc/systemd/system/jenkins-agent.service file with the following content:

[Unit]
Description=Jenkins Agent

[Service]
User=jenkins
WorkingDirectory=/home/jenkins
ExecStart=/bin/bash /usr/local/jenkins-service/start-agent.sh
Restart=always

[Install]
WantedBy=multi-user.target



We still have to enable the daemon with the following command:

sudo systemctl enable jenkins-agent.service

Let’s have a look at the system logs before starting the daemon:

journalctl -f &

Now start the daemon with the following command.


sudo systemctl start jenkins-agent.service



