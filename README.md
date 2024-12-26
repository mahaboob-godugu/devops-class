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
We can see some interesting logs in the journalctl output:

Aug 03 19:37:27 ubuntu-machine systemd[1]: Started Jenkins Agent.
Aug 03 19:37:27 ubuntu-machine sudo[8821]: pam_unix(sudo:session): session closed for user root
Aug 03 19:37:28 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:28 PM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
Aug 03 19:37:28 ubuntu-machine bash[8826]: INFO: Using /home/jenkins/remoting as a remoting work directory
Aug 03 19:37:28 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:28 PM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
Aug 03 19:37:28 ubuntu-machine bash[8826]: INFO: Both error and output logs will be printed to /home/jenkins/remoting
Aug 03 19:37:28 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:28 PM hudson.remoting.jnlp.Main createEngine
Aug 03 19:37:28 ubuntu-machine bash[8826]: INFO: Setting up agent: My New Ubuntu 22.04 Node with Java and Docker installed
Aug 03 19:37:28 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:28 PM hudson.remoting.Engine startEngine
Aug 03 19:37:28 ubuntu-machine bash[8826]: INFO: Using Remoting version: 3046.v38db_38a_b_7a_86
Aug 03 19:37:28 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:28 PM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
Aug 03 19:37:28 ubuntu-machine bash[8826]: INFO: Using /home/jenkins/remoting as a remoting work directory
Aug 03 19:37:29 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:29 PM hudson.remoting.jnlp.Main$CuiListener status
Aug 03 19:37:29 ubuntu-machine bash[8826]: INFO: Locating server among [http://controller_ip:58080/]
Aug 03 19:37:29 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:29 PM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
Aug 03 19:37:29 ubuntu-machine bash[8826]: INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
Aug 03 19:37:29 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:29 PM hudson.remoting.jnlp.Main$CuiListener status
Aug 03 19:37:29 ubuntu-machine bash[8826]: INFO: Agent discovery successful
Aug 03 19:37:29 ubuntu-machine bash[8826]:   Agent address: controller_ip
Aug 03 19:37:29 ubuntu-machine bash[8826]:   Agent port:    50000
Aug 03 19:37:29 ubuntu-machine bash[8826]:   Identity:      31:c4:f9:31:46:c3:eb:72:64:a3:c7:d6:c7:ea:32:2f
Aug 03 19:37:29 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:29 PM hudson.remoting.jnlp.Main$CuiListener status
Aug 03 19:37:29 ubuntu-machine bash[8826]: INFO: Handshaking
Aug 03 19:37:29 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:29 PM hudson.remoting.jnlp.Main$CuiListener status
Aug 03 19:37:29 ubuntu-machine bash[8826]: INFO: Connecting to controller_ip:50000
Aug 03 19:37:29 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:29 PM hudson.remoting.jnlp.Main$CuiListener status
Aug 03 19:37:29 ubuntu-machine bash[8826]: INFO: Trying protocol: JNLP4-connect
Aug 03 19:37:29 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:29 PM org.jenkinsci.remoting.protocol.impl.BIONetworkLayer$Reader run
Aug 03 19:37:29 ubuntu-machine bash[8826]: INFO: Waiting for ProtocolStack to start.
Aug 03 19:37:30 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:30 PM hudson.remoting.jnlp.Main$CuiListener status
Aug 03 19:37:30 ubuntu-machine bash[8826]: INFO: Remote identity confirmed: 31:c4:f9:31:46:c3:eb:72:64:a3:c7:d6:c7:ea:32:2f
Aug 03 19:37:30 ubuntu-machine bash[8826]: Aug 03, 2022 7:37:30 PM hudson.remoting.jnlp.Main$CuiListener status
Aug 03 19:37:30 ubuntu-machine bash[8826]: INFO: Connected
We can now check the status with the command below, and the output 
