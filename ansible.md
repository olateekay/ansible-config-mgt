# Ansible Client as a Jump Server (Bastion Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet.


![alt text](https://github.com/olateekay/ansible-config-mgt/blob/main/images/image1.JPG)

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

![alt text](https://github.com/olateekay/ansible-config-mgt/blob/main/images/image2.JPG)

![alt text](https://github.com/olateekay/ansible-config-mgt/blob/main/images/image3.JPG)

![alt text](https://github.com/olateekay/ansible-config-mgt/blob/main/images/image4.JPG)

Note: Trigger Jenkins project execution only for /main (master) branch.

![alt text](https://github.com/olateekay/ansible-config-mgt/blob/main/images/image6b.JPG)

## Step 2 - Begin Ansible Development

1.) In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.


2.) Checkout the newly created feature branch to your local machine and start building your code and directory structure

3.) Create a directory and name it playbooks - it will be used to store all your playbook files.

4.) Create a directory and name it inventory - it will be used to keep your hosts organised.

5.) Within the playbooks folder, create your first playbook, and name it common.yml

6.) Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

![alt text](https://github.com/olateekay/ansible-config-mgt/blob/main/images/image6.JPG)


## Step 4 - Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

Save below inventory structure in the `inventory/dev` file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host - for this you need to copy your private (.pem) key to your server. Do not forget to change permissions to your private key chmod 400 key.pem, otherwise EC2 will not accept the key. Now you need to import your key into ssh-agent:

There are several ways to achieve this, we can take this route;

a.) Copy the content of your `.pem` file with which you `ssh` into your Ansible control node
a.) In the root folder of your Ansible control node, navigate to the `.ssh` directory and create a file, paste the content of your `.pem` file into the newly created file
c.) Give the file permissions `chmod 400 <file name>`
follow this example, the content of the `.pem` file was copied into a file named keys

```
ubuntu@ip-172-31-44-66:~$ cd .ssh
ubuntu@ip-172-31-44-66:~/.ssh$ ssh-agent bash
ubuntu@ip-172-31-44-66:~/.ssh$ ssh-add keys
Identity added: keys (keys)

```


Also notice, that your Load Balancer user is `ubuntu` and user for RHEL-based servers is `ec2-user`.

1.) Update your `inventory/dev` file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ubuntu' 

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

## Step 5 - Update GIT with the latest code
Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.

In the real world, you will be working within a team of other DevOps engineers and developers. It is important to learn how to collaborate with help of GIT. In many organisations there is a development rule that do not allow to deploy any code before it has been reviewed by an extra pair of eyes - it is also called “Four eyes principle”.

Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the master branch.

Commit your code into GitHub:

use git commands to add, commit and push your branch to GitHub.

```
git status

git add <selected files>

git commit -m "commit message"

```

Create a Pull request (PR)
Wear a hat of another developer for a second, and act as a reviewer.
If the reviewer is happy with your new feature development, merge the code to the master branch.
Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.
Once your code changes appear in master branch - Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on Jenkins-Ansible server.

![alt text](https://github.com/olateekay/ansible-config-mgt/blob/main/images/image9.JPG)


Now, it is time to execute ansible-playbook command and verify if your playbook actually works:

```
ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml 

```

output

```
ubuntu@ip-172-31-44-66:~$ ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/10/archive/inventory/dev /var/lib/jenkins/jobs/ansible/builds/10/archive/playbooks/common.yml

PLAY [update web, nfs and db servers] *****************************************************************************************************
TASK [Gathering Facts] ********************************************************************************************************************ok: [172.31.13.254]
ok: [172.31.11.45]
ok: [172.31.5.122]
ok: [172.31.15.118]

TASK [ensure wireshark is at the latest version] ******************************************************************************************ok: [172.31.13.254]
ok: [172.31.11.45]
ok: [172.31.15.118]
changed: [172.31.5.122]

PLAY [update LB server] *******************************************************************************************************************
TASK [Gathering Facts] ********************************************************************************************************************

PLAY [update web, nfs and db servers] *****************************************************************************************************
TASK [Gathering Facts] ********************************************************************************************************************ok: [172.31.15.118]
ok: [172.31.11.45]
ok: [172.31.13.254]
ok: [172.31.5.122]

TASK [ensure wireshark is at the latest version] ******************************************************************************************ok: [172.31.5.122]
ok: [172.31.13.254]
ok: [172.31.11.45]
ok: [172.31.15.118]

PLAY [update LB server] ******************************************************************************************************************************************************
TASK [Gathering Facts] *******************************************************************************************************************************************************ok: [172.31.40.150]

TASK [ensure wireshark is at the latest version] ******************************************************************************************ok: [172.31.40.150]

PLAY RECAP ********************************************************************************************************************************172.31.11.45               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.31.13.254              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0
 ignored=0
172.31.15.118              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0



```

You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version

ubuntu@ip-172-31-5-122:~$ wireshark --version
Wireshark 3.2.3 (Git v3.2.3 packaged as 3.2.3-1)

Copyright 1998-2020 Gerald Combs <gerald@wireshark.org> and contributors.
License GPLv2+: GNU GPL version 2 or later <https://www.gnu.org/licenses/gpl-2.0.html>
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

Compiled (64-bit) with Qt 5.12.8, with libpcap, with POSIX capabilities (Linux),
with libnl 3, with GLib 2.64.2, with zlib 1.2.11, with SMI 0.4.8, with c-ares
1.15.0, with Lua 5.2.4, with GnuTLS 3.6.13 and PKCS #11 support, with Gcrypt
1.8.5, with MIT Kerberos, with MaxMind DB resolver, with nghttp2 1.40.0, with
brotli, with LZ4, with Zstandard, with Snappy, with libxml2 2.9.10, with
QtMultimedia, without automatic updates, with SpeexDSP (using system library),
with SBC, with SpanDSP, without bcg729.

Running on Linux 5.4.0-1047-aws, with Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz
(with SSE4.2), with 978 MB of physical memory, with locale C.UTF-8, with libpcap
version 1.9.1 (with TPACKET_V3), with GnuTLS 3.6.13, with Gcrypt 1.8.5, with
brotli 1.0.7, with zlib 1.2.11, binary plugins supported (0 loaded).

Built using gcc 9.3.0.
ubuntu@ip-172-31-5-122:~$ client_loop: send disconnect: Connection reset by peer

Olatokunbo.Ogunlade@VGG-LT-338 MINGW64 ~
```

```
Done!
