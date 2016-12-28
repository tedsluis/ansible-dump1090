# ansible-dump1090
Use [Ansible](https://www.ansible.com/) playbooks for dump1090 hosts. Ansible, make it so!  
  
## Table of contents  
  

## Why manage hosts using Ansible?  
  
I have several raspberry pi's and orange pi's that run dump1090, piaware, dump1090-tools (collectd), etc. I reinstall them freqently. It takes a lot of steps to install all the packages and edit all the configuration files.  
  
It is hard to install new dump1090 devices in the same way. It is easy to forget steps or do it differtly. None of my dump1090 devices were the same until I started using Ansible.  
  
With Ansible I can

* Keep control over the installed packages and configuration settings on your linux devices.
* Configure Linux and the applications using 'code' instead of doing it manually.
* Deploy packages and settings from one centralized management host.
* Deploy updates rapidly over multiple hosts.
* Guarantee that my devices are always in the correct same state after every reinstall.  

Ansible just checks the state. If it is already in that right state, it does nothing. But if it isn't then Ansible will configure it in the right state!
  
Ansible needs to be installed on only one 'management' host. This could be a raspberry pi or an other linux device. Ansible doesn't needs 'agents' or any additional software on the hosts you want to manage. It uses only SSH login with a SSH key!  

Ansible is open source software (maintained by Red Hat) and is used on a wide variety of linux distro's.  
  
## Types of playbooks  
  
Playbook                                                                                            | Description                                            | settings                    |
----------------------------------------------------------------------------------------------------|--------------------------------------------------------|-----------------------------|
(installbasics.yml)[https://github.com/tedsluis/ansible-dump1090/blob/master/installbasics.yml]     | Does the basic installation and configuration of host. | roles/basics/tasks/main.yml |
(installdump1090.yml)[https://github.com/tedsluis/ansible-dump1090/blob/master/installdump1090.yml] | Does the building and installation of dump1090.        | group_vars/dump1090         |
  
## Install Ansible  
  
To install Ansible on your management system.   
````
$ sudo apt-get update
$ sudo apt-get install ansible
````
Check here the installation notes for other linux distro's: http://docs.ansible.com/ansible/intro_installation.html  
    
## Generate SSH key  
  
To let Ansible log in from a your management host to your dump1090 hosts it need a public and private SSH key.  
The public key '~/.ssh/id_rsa.pub' must but added to the 'home/pi/.ssh/authorized_keys' on your dump1090 hosts.   
The private key must be stored on your management host in /home/username/.ssh/id.rsa.  

If you don't have a private and public SSH key yet you can generate them on your Ansible management host like this:
  
````
pi@raspberry-1:~ $ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/pi/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): <====== leave this empty.
Enter same passphrase again: 
Your identification has been saved in /home/pi/.ssh/id_rsa.
Your public key has been saved in /home/pi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:jkGHjiQUPMgVkyHUcmeVoxrzMBYdWX4lJHkoQf9D5ZU tedsluis@msi.bachstraat20
The key's randomart image is:
+---[RSA 2048]----+
|@AAA=o+..        |
|=*+B+B.o.        |
|..*oXoiEo...     |
|  .X++.          |
|    O+. S        |
|   . ..+         |
|      . .        |
|                 |
|                 |
+----[SHA256]-----+
pi@raspberry-1:~ $ ll ~/.ssh
total 16
-rw-------. 1 tedsluis tedsluis 1675 Dec 26 07:09 id_rsa
-rw-r--r--. 1 tedsluis tedsluis  407 Dec 26 07:09 id_rsa.pub
-rw-r--r--. 1 tedsluis tedsluis 4823 Dec 26 07:02 known_hosts
````
Now use ssh to create a directory ~/.ssh as user pi on your raspberry-2 (one of your dump1090 hosts you want to manage using Ansible). (The directory may already exist, which is fine):  
````
pi@raspberry-1:~ $ ssh pi@raspberry-2 mkdir -p .ssh
pi@raspberry-2's password: 
````
Finally append raspberry-1's new public key to pi@raspberry-2:.ssh/authorized_keys and enter raspberry-2's password one last time:
````
pi@raspberry-1:~ $ cat .ssh/id_rsa.pub | ssh pi@raspberry-2 'cat >> .ssh/authorized_keys'
pi@raspberry-2's password:
````
From now on you (and Ansible) can log into raspberry-2 as pi from raspberry-1 as a without password:
````
pi@raspberry-1:~ $ ssh pi@raspberry-2
connect to pi@raspberry-2 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Dec 27 10:32:12 2016 from raspberry-1
pi@raspberry-2:~ $ 
````
Note: Depending on your version of SSH you might also have to do the following changes:

* Put the public key in .ssh/authorized_keys2
* Change the permissions of .ssh to 700
* Change the permissions of .ssh/authorized_keys2 to 640
    
## Clone this repo  
  
Get the Ansible playbooks on your Ansible management host:  
````
$ cd ~
$ mkdir git
$ cd git
$ git clone https://github.com/tedsluis/ansible-dump1090.git
$ cd ansible-dump1090
````
Check the results:   
````
$ ls -l
total 80
-rw-r--r--. 1 tedsluis tedsluis 14035 Dec 27 21:50 ansible.cfg
drwxr-xr-x. 2 tedsluis tedsluis  4096 Dec 27 17:23 group_vars
-rw-r--r--. 1 tedsluis tedsluis   718 Dec 28 08:45 hosts
-rw-rw-r--. 1 tedsluis tedsluis   119 Dec 28 07:02 installbasics.yml
-rw-rw-r--. 1 tedsluis tedsluis   396 Dec 28 06:43 installdump1090.yml
-rw-r--r--. 1 tedsluis tedsluis 35141 May 22  2016 LICENSE
-rw-r--r--. 1 tedsluis tedsluis  5702 Dec 28 10:34 README.md
drwxr-xr-x. 7 tedsluis tedsluis  4096 Dec 28 06:38 roles
$ pwd
/home/pi/git/ansible-dump1090
````
  
## Change the default ansible.cgf location  
  
Before start any playbook: Change the default location for ansible.cfg on your Ansible management host using:
````
export ANSIBLE_CONFIG="$HOME/git/ansible-dump1090/ansible.cfg"
````
Add this line to your '~/.profile', to be sure it is set every time you use Ansible.  
    
## Add your dump1090 hosts to your Ansible 'hosts' file
  
Add your dump1090 hosts to the '$HOME/git/ansible-dump1090/hosts' file on your Ansible management host, for example:
````
# Raspberry Pi 1B, 2B+, 3B - Raspbian Jessie 4.4.38-v7+
[raspbian]
ted1090-1
ted1090-2
ted1090-3
ted1090-4
ted1090-5

# Orange Pi 2 plus - Armbian 3.4.112-sun8i
[armbian]
ted1090-6
ted1090-7

[dump1090]
ted1090-1
ted1090-4
ted1090-5
ted1090-7

[all]
ted1090-1
ted1090-2
ted1090-3
ted1090-4
ted1090-5
ted1090-6
ted1090-7
````
  
## Test Ansible
  
Now test Ansible from your Ansible management host: 
````
$ ansible all -a "/bin/echo hello"
ted1090-5 | SUCCESS | rc=0 >>
hello

ted1090-3 | SUCCESS | rc=0 >>
hello

ted1090-4 | SUCCESS | rc=0 >>
hello

ted1090-6 | SUCCESS | rc=0 >>
hello

ted1090-7 | SUCCESS | rc=0 >>
hello

ted1090-2 | SUCCESS | rc=0 >>
hello

ted1090-1 | SUCCESS | rc=0 >>
hello
````
Note: In this example 'all' is a group name. Therefor a section [all] should be present within your '$HOME/git/ansible-dump1090/hosts' file containing your dump1090 hostnames! 
  
## Run a playbook  
  
In this example I run a playbook only on one host (as configured in installbasics.yml):
````
$ ansible-playbook  installbasics.yml 

PLAY [install basic packages and configure basic settings.] ********************

TASK [setup] *******************************************************************
ok: [ted1090-2]

TASK [basics : Install basic packages] *****************************************
ok: [ted1090-2] => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'vim', u'wget'])

TASK [basics : Insert alias ll into .profile] **********************************
ok: [ted1090-2]

TASK [basics : Set vim syntax no (globaly)] ************************************
ok: [ted1090-2]

TASK [basics : Add /home/pi/connect.dump.sh to /etc/rc.local] ******************
ok: [ted1090-2]

TASK [basics : Update /etc/hosts from inventory] *******************************
skipping: [ted1090-2] => (item=ted1090-1) 
ok: [ted1090-2] => (item=ted1090-2)
skipping: [ted1090-2] => (item=ted1090-3) 
skipping: [ted1090-2] => (item=ted1090-4) 
skipping: [ted1090-2] => (item=ted1090-5) 
skipping: [ted1090-2] => (item=ted1090-6) 
skipping: [ted1090-2] => (item=ted1090-7) 

PLAY RECAP *********************************************************************
ted1090-2                  : ok=6    changed=0    unreachable=0    failed=0   
````
  
## More info
  
* http://docs.ansible.com/ansible
* ted.sluis@gmail.com
