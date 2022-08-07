# A 3-Node Cluster Setup Using JNLP

## Prerequisites 

- 3 virtual machines (VMs) up and running in the same subnet / VPC. We are using EC2 instances on the AWS Cloud, so some of the instructions are specific to EC2. 
- **Ubuntu 20.04 LTS** OS on all machines. 
- [FileZilla](https://filezilla-project.org/) installed on the local / host machine.

## Part A - Configure Jenkins Controller (Master Node)

1. Connect to the one of the VMs using any SSH client (PuTTY, MobaXTerm, Asbru, etc.). We will designate this machine as the **Jenkins Controller**.
2. [Install Jenkins for Ubuntu/Debian](https://pkg.jenkins.io/debian-stable/) on this VM (designated as Jenkins Controller).
3. To check if Jenkins is installed successfully and is in a running state, run the following command:
    ```
    $ service jenkins status
    ```
    You should see status as ```active (running)```. If not, try running ```service jenkins start``` and check the status again.
4. Open a browser and go to `public-ip-of-jenkins-controller:8080`. Jenkins by-default uses port number 8080.

5. Get the initial password by running the following command on the Ubuntu terminal:
    ```
    $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
6. Copy the initial admin password from the terminal and paste in the **Administrator password** field on the **Unlock Jenkins** page.
7. Click **Continue** and **Install suggested plugins**. 

    > Installing suggested plugins might take about 5 minutes.

8. Create the first admin user by filling in the required details. Ensure you can recall the username and password later.
9. Click **Save and Continue**. Note down the Jenkins URL and click **Save and Finish**.
10. Click **Start using Jenkins**.

Jenkins is now up and running on the **Jenkins Controller**.

## Part B - Add Jenkins Agent (Worker Node)

1. Click **Manage Jenkins** (on the left panel) of the **Welcome to Jenkins!** page.
2. Click **Configure Global Security**.
3. Select **Random** under **Agents** (TCP port for inbound agents) and click **Save**.
4. Click **Manage Nodes and Clouds** on the **Manage Jenkins** page and click **New Node**.
5. Specify a node name, e.g. ```worker-node-a```.
6. Ensure **Permanent Agent** is selected and click **Create**.
7. Specify ```/home/ubuntu/jenkins``` in the **Remote root directory** field. Ensure no spaces before or after the specified path.
8. Ensure the **Launch method** is set to ```Launch agent by connecting it to the controller```.
9. Specify ```/home/ubuntu/jenkins``` in the **Custom WorkDir path** field. Again, ensure no spaces before or after the specified path.
10. Select the **Use WebSocket** option and click **Save**.

The worker node is now added (but not yet configured).

## Part C - Configure Jenkins Agent (Worker Node)

1. Connect to another VM using any SSH client (PuTTY, MobaXTerm, Asbru, etc.). We will designate this machine as the **Jenkins Agent (Worker Node)**.
2. Install JRE using the following commands:
    ```
    $ sudo apt-get update
    $ sudo apt-get install fontconfig openjdk-11-jre
    ```
3. (if already not open) Open a browser and go to `public-ip-of-jenkins-controller:8080`.
4. Click the worker node name on the **Dashboard** > **Manage Jenkins** > **Manage node and clouds** page.
    
    > It is the same Jenkins agent (worker node) that you added in Part B.

5. Click on ```agent.jar``` under **Run from agent command line** option and save it on your local/host machine.
6. Open [FileZilla](https://filezilla-project.org/) and connect to this VM (designated as Jenkins Agent / Worker Node).
    1. Click **Edit** > **Settings** > **SFTP** > **Add key file...** and add the relevant ```.pem``` or ```.ppk``` key for this VM (specific to EC2 instance).
    2. Enter ```public-ip-of-worker-node``` in the **Host** field.
    3. Enter ```ubuntu``` in the **Username** field as we are connecting to an Ubuntu VM.
    4. Enter ```22``` in the **Port** field as we are using SSH connection.
7. Transfer the downloaded ```agent.jar``` to the ```/home/ubuntu/``` directory on the VM.
8. (if already not there) Click the worker node name on the **Dashboard** > **Manage Jenkins** > **Manage node and clouds** page.
9. Copy the command under **Run from agent command line** option. As an **example**, the commands looks like the following:
    ```
    java -jar agent.jar -jnlpUrl http://13.235.2.113:8080/computer/trial%2Dworker%2D01/jenkins-agent.jnlp -secret a5700994b3d999d697ed77ea5b7bbe18b536b5e36eb1b87b773e595debfcc1b4 -workDir "/home/ubuntu/jenkins"
    ```
    > NOTE: The above command is only an example, do not use it as is.
    
10. Open the Ubuntu terminal on this VM (designated as Jenkins Agent / Worker Node) and run the command copied in the previous step.

The worker node is now configured and connected to the **Jenkins Controller**. Repeat **Part B** and **Part C** for additional nodes.
    