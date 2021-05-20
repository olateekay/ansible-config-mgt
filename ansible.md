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

![alt text](image6b.jpg)

## Step 2 - Begin Ansible Development

1.) In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.


2.) Checkout the newly created feature branch to your local machine and start building your code and directory structure

3.) Create a directory and name it playbooks - it will be used to store all your playbook files.

4.) Create a directory and name it inventory - it will be used to keep your hosts organised.

5.) Within the playbooks folder, create your first playbook, and name it common.yml

6.) Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.


## Step 4 - Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

Save below inventory structure in the `inventory/dev` file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host - for this you need to copy your private (.pem) key to your server. Do not forget to change permissions to your private key chmod 400 key.pem, otherwise EC2 will not accept the key. Now you need to import your key into ssh-agent:

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>

```

Also notice, that your Load Balancer user is `ubuntu` and user for RHEL-based servers is `ec2-user`.

1.) Update your `inventory/dev.yml` file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

```

## Step 4 - Create a Common Playbook

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in `inventory/dev`.

In `common.yml` playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your `playbooks/common.yml` file with following code:

```
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    apt:
      name: wireshark
      state: latest

```

Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

Feel free to update this playbook with following tasks:

Create a directory and a file inside it
Change timezone on all servers
Run some shell script
