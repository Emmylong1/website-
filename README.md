# Ansible Playbook for Apache Web Server Deployment on Multiple EC2 Instances


Why Ansible?



Ansible can help you demonstrate value, link teams, and improve efficiency regardless of your job or automation ambitions. Ansible automates monotonous activities to free up DevOps teams for strategic work.



Agenda
In this blog, we will learn how to deploy a website on multiple AWS EC2 Instances using Ansible Playbooks without installing the service on each server separately using automation, which is the ultimate need for DevOps and the key principle of "Automate everything."


Pre-requisites:
Basic knowledge of Ansible and playbooks
Linux Project Architecture: This blog follows our project's reference architecture, shown below.


The Process:
1. Create AWS VPC Security Groups for our Ansible Control Master Server and nodes.Log into the AWS interface, pick your region, return to VPC, and select security groups on the left navigation bar:


Security group creation:


Name, description, and default VPC are required for your security group:


Allow ssh from anywhere under Inbound rules and click Create security group for this demonstration.


Let's build another security group for all three Ansible nodes where we'll install the web server.


Allow HTTP from everywhere and SSH from the master security group only under Inbound Rules and click Create security group:



2. Create the Ansible master EC2 instance after creating both security groups.
Launch instance on the EC2 dashboard:



EC2 instance name:




Select free-tier Amazon Linux 2023 AMI under AMI:




Choose t2.micro, a free tier instance with 1 vCPU and 1 GiB memory.





We can build or utilize a key pair to ssh into the EC2 instance. This example creates a new key pair.





Name your key pair and click Create key pair. Save this key pair since you can only download it once.



Edit Network Settings. Choose the default VPC, a subnet, or nothing. Enable auto-assign public IP and select the current security group for the ansible-master server we setup in step 1.




Keep the default settings and click Launch instance.


After the success notification, click on Connect to Instance:


To ssh into the server, copy and paste these commands from the Connect to instance window into your terminal:


ssh -i "devops.pem" ec2-user@ec2-52-90-115-27.compute-1.amazonaws.com




Final Output:





3. Create a key-pair on the Ansible master server to connect the master and nodes with the command:


ssh-keygen --rsa-2048





4. After creating the SSH key pairs, we must import the public key into the EC2 dashboard to allow the Ansible nodes to communicate with the master server.
Copy the public key using the output steps:





Select Key Pairs on the left navigation bar under Network & Security in the EC2 management interface after copying the public key. Import Key Pair under Actions.






Paste the terminal public key and name the new key. Click Import key combination.



5. Launch Ansible Node Servers: This example requires three EC2 instances.
Launch instances on the EC2 dashboard.
Name and type 3 under instances.




Choose t2.micro and Amazon Linux 2023 AMI.
We must select ansible-public-key from our demo under the key pair.


Enable Auto-assign public IP under Network settings for the default VPC.Select the security group we created before, ansible-nodes-sg.


Launch instances without changing any settings.


We may view and tag instances after launching:



6. Test Ansible master-node server connectivity: Log into the master Ansible server and retrieve the first node server's private IP from the console to test the ssh connection between the master and worker/slave nodes:



As shown in the screenshot, type the following command in the Ansible master server terminal:




The private IP should match the AWS console copy.
Check connectivity between the other two servers and the master server.


Note: Since we haven't installed Ansible on the master server, SSH allows us to connect to all other nodes.


7. Ansible on Master Server:
The following command will update our Ansible master server:
yum update


Amazon Linux 2023 AMI installation methods differ from Amazon Linux 2 AMI steps.
Install Ansible using these commands:
curl  Bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user python3 -m pip install-user ansible




Check the Ansible version using the following command to verify installation:




8. Create an Ansible Inventory file: Inventory listings automate tasks on managed nodes, or hosts, in your system. Most Ansible users use inventory files rather than command-line host names. To automate several hosts at once, your inventory groups controlled nodes.
The inventory file lets you group and subgroup servers.
The following command creates an inventory file on the Ansible Master Server:
sudo inventory
Create a webservers group and enter the node servers' IP addresses:







9. Create an Ansible Playbook: Automation tasks are complicated IT actions performed without human intervention. Ansible inventory hosts execute playbooks.
Create the playbook on the Ansible Master Server in the home directory, where our inventory file is:
website.yml sudo
Open the playbook file and paste the code. This blog ends with the GitHub repo URL.



[ec2-user@ip-172-31-92-75 ~] $cat website.yml


deploy bootstrap-website hosts: all
become: root


tasks:
update ec2 instance
yum:
name: "*"
update_cache: yes


install apache server
yum:
name: httpd
state: latest


convert directory to HTML
C: /var/www/html


github web file download
get_url: https://github.com/mudasirhaji/website/archive/refs/tags/zipfile.zip dest: /var/www/html/


unzip ansible.builtin.unarchive:
src: /var/www/html/website-zipfile.zip dest: /var/www


name: copy webfiles from website-zipfile to html copy: src: /var/www/html/website-zipfile/ dest: /var/www/html remote_src: yes


name: delete website-zipfile:
website-zipfile: /var/www/html
state: absence


remove website-zipfile.zip:
website-zipfile.zip: /var/www/html
state: absence


name: start Apache if necessary.
ansible.builtin.service:
enable: yes
name: httpd
state: begun




Ansible may be used to test connectivity between the master and all nodes before running the playbook to deploy our website on the Apache server:
ansible all --key-file /.ssh/id_rsa --inventory --ping --ec2-user
This is the command's successful output:




Ansible connects to nodes as seen in green above.
Before executing our Ansible Playbook, we need to generate an Ansible configuration file that lists our key pair, inventory file, and default username of ec2-user.
Sudo vi ansible.cfg to generate the config file.
Enter the following after opening the file:
[defaults]
remote_user=ec2-user inventory=inventory private_key_file=/.ssh/id_rsa


Save and close.


10. Install Webservers using the playbook:
Ansible Master Server: ansible-playbook website.yml



We may also verify by browsing to any Node Server public IP. Copy the public IP of Ansible-node-1 from the EC2 console:




Paste the public IP into your browser to see your website like I do:



Our website is also deployed successfully on additional node servers.


Conclusion
In this blog, we wrote an Ansible playbook on an EC2 instance and deployed a website on an Apache web server on numerous EC2 instances.


See my blog for details:
My blog:


