# Ansible Client as a Jump Server (Bastion Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet.


![alt text](image1.jpg)

## Task
1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple Ansible playbook to automate servers configuration

### Step 1 - Install and configure Ansible on EC2 Instance

a.) Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
b.) In your GitHub account create a new repository and name it ansible-config-mgt.
c.) Install Ansible

```
sudo apt update

sudo apt install ansible

```

d.) Check your Ansible version by running 

`ansible --version`

```
output

ubuntu@ip-172-31-44-66:~$ ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.5 (default, Jan 27 2021, 15:41:15) [GCC 9.3.0]


```

e.) Configure Jenkins build job to save your repository content every time you change it 

f.) Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

g.) Configure Webhook in GitHub and set webhook to trigger ansible build.

h.) Configure a Post-build job to save all (**) files
I.) Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

![alt text](image2.jpg)

![alt text](image3.jpg)

![alt text](image4.jpg)

Note: Trigger Jenkins project execution only for /main (master) branch.