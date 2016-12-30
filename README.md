# ansible-dump1090
Use [Ansible](https://www.ansible.com/) playbooks to configure dump1090 hosts. Ansible, make it so!  
  
## Table of contents  
  
  * [ansible-dump1090](#ansible-dump1090)
    * [Why manage hosts using Ansible?](#why-manage-hosts-using-ansible)
    * [Types of playbooks](#types-of-playbooks)
    * [Install Ansible](#install-ansible)
    * [Generate SSH key](#generate-ssh-key)
    * [Clone this repo](#clone-this-repo)
    * [Change the default ansible.cgf location](#change-the-default-ansiblecgf-location)
    * [Add your dump1090 hosts to your Ansible 'hosts' file](#add-your-dump1090-hosts-to-your-ansible-hosts-file)
    * [Test Ansible](#test-ansible)
    * [Run a playbook](#run-a-playbook)
    * [Logging](#logging)
    * [More info](#more-info)
  
## Why manage hosts using Ansible?  
  
I have several raspberry pi's and orange pi's that run dump1090-mutability, piaware, dump1090-tools (collectd), etc. I reinstall them freqently. It takes a lot of steps to install all the packages and edit all the configuration files.  
  
It is hard to install new dump1090 devices always in the exact same way. It is easy to forget steps or do it differtly. None of my dump1090 devices were the same until I started using Ansible.  
  
With Ansible I can

* Keep control over the installed packages and configuration settings on a group of linux devices.
* Configure Linux and the applications using 'code' instead of doing it manually.
* Deploy packages and settings from one centralized management host.
* Deploy updates rapidly over multiple hosts.
* Guarantee that devices are always in the correct same state after every reinstall.  

Ansible just checks the 'state'. If it is already in that right state, it does nothing. But if it isn't then Ansible will configure it in the right state!
  
Ansible needs to be installed on only one 'management' host. This could be a raspberry pi or an other linux device. You can even run it on one of your dump1090 hosts! Ansible doesn't needs 'agents' or any additional software on the hosts you want to manage. It uses only SSH login with a SSH key!  

Ansible is open source software (maintained by Red Hat) and is used on a wide variety of linux distro's.  
  
## Types of playbooks  
  
Playbook                                                                                            | Description                                            | Tasks and settings                    |
----------------------------------------------------------------------------------------------------|--------------------------------------------------------|-----------------------------|
[installbasics.yml](https://github.com/tedsluis/ansible-dump1090/blob/master/installbasics.yml)     | Does the basic installation and configuration of hosts. |[packages](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/packages.yml)<br>[git settings](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/git.yml)<br>[run script](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/connectdumpsh.yml)<br>[aliasses](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/aliasses.yml)<br>[vim settings](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/vimsettings.yml)<br>[/etc/hosts file](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/etchosts.yml)<br>[crontab jobs](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/cronjobs.yml)<br>[time zone](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/basics/tasks/timezone.yml)|
[installdump1090.yml](https://github.com/tedsluis/ansible-dump1090/blob/master/installdump1090.yml)| Does the building and installation of dump1090.        | [prepare](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/dump1090-prepare/tasks/main.yml)<br>[build](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/dump1090-build/tasks/main.yml)<br>[install task](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/dump1090-install/tasks/main.yml)<br>[install handler](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/dump1090-install/handlers/main.yml)<br>[group_vars/dump1090](https://github.com/tedsluis/ansible-dump1090/blob/master/group_vars/dump1090)<br> [group_vars/default](https://github.com/tedsluis/ansible-dump1090/blob/master/group_vars/all)        |
  
## Install Ansible  
  
To install Ansible on your management system.   
````
pi@raspberry-1:~ $ sudo apt-get update
pi@raspberry-1:~ $ sudo apt-get install ansible
````
Check here the installation notes for other linux distro's: http://docs.ansible.com/ansible/intro_installation.html  

By default it store its configuration files in /etc/ansible. By we will change this in a minute (see below).  
    
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
SHA256:jkGHjiQUPMgVkyHUcmeVoxrzMBYdWX4lJHkoQf9D5ZU tedsluis@raspberry-1
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
Now use ssh to create a directory ~/.ssh as user pi on your raspberry-2 (one of your dump1090 hosts that you want to manage using Ansible). (The directory may already exist, which is fine):  
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
pi@raspberry-1:~ $ cd ~
pi@raspberry-1:~ $ mkdir git
pi@raspberry-1:~ $ cd git
pi@raspberry-1:~/git $ git clone https://github.com/tedsluis/ansible-dump1090.git
pi@raspberry-1:~/git $ cd ansible-dump1090
````
Check the results:   
````
pi@raspberry-1:~/git/ansible-dump1090 $ ls -l
total 80
-rw-r--r--. 1 tedsluis tedsluis 14035 Dec 27 21:50 ansible.cfg
drwxr-xr-x. 2 tedsluis tedsluis  4096 Dec 27 17:23 group_vars
-rw-r--r--. 1 tedsluis tedsluis   718 Dec 28 08:45 hosts
-rw-rw-r--. 1 tedsluis tedsluis   119 Dec 28 07:02 installbasics.yml
-rw-rw-r--. 1 tedsluis tedsluis   396 Dec 28 06:43 installdump1090.yml
-rw-r--r--. 1 tedsluis tedsluis 35141 May 22  2016 LICENSE
-rw-r--r--. 1 tedsluis tedsluis  5702 Dec 28 10:34 README.md
drwxr-xr-x. 7 tedsluis tedsluis  4096 Dec 28 06:38 roles
pi@raspberry-1:~/git/ansible-dump1090 $ pwd
/home/pi/git/ansible-dump1090
````
  
## Change the default ansible.cgf location  
  
The [$HOME/git/ansible-dump1090/ansible.cfg](https://github.com/tedsluis/ansible-dump1090/blob/master/ansible.cfg) file contains your personal Ansible settings. In this case the following settings are different from the default Ansible settings:  
````
inventory          = $HOME/git/ansible-dump1090/hosts
roles_path         = $HOME/git/ansible-dump1090/roles
host_key_checking  = False
remote_user        = pi 
log_path           = /tmp/ansible.log
become_method      = sudo
````
  
Before you start any playbook: Change the default location for ansible.cfg on your Ansible management host using:
````
pi@raspberry-1:~ $ export ANSIBLE_CONFIG="$HOME/git/ansible-dump1090/ansible.cfg"
````
And also add this line to your '~/.profile', to be sure it is set every time you use Ansible.  
      
## Add your dump1090 hosts to your Ansible 'inventory' file
  
The [$HOME/git/ansible-dump1090/hosts](https://github.com/tedsluis/ansible-dump1090/blob/master/hosts) inventory file contains groups of hosts. This way you can apply some tasks on one group of hosts and other tasks on an other group of hosts.  
  
Add your own dump1090 hosts to the '$HOME/git/ansible-dump1090/hosts' inventory file on your Ansible management host, for example:
````
# Raspberry Pi 1B, 2B+, 3B - Raspbian Jessie 4.4.38-v7+
[raspbian]
raspberry-1
raspberry-2
raspberry-3
raspberry-4
raspberry-5

# Orange Pi 2 plus - Armbian 3.4.112-sun8i
[armbian]
orangepi-6
orangepi-7

[dump1090]
raspberry-1
raspberry-4
raspberry-5
orangepi-7

[all]
raspberry-1
raspberry-2
raspberry-3
raspberry-4
raspberry-5
orangepi-6
orangepi-7
````
  
## Test Ansible
  
Now test Ansible from your Ansible management host and execute a command on multiple hosts: 
````
pi@raspberry-1:~ $ ansible all -a "/bin/echo hello"
raspberry-5 | SUCCESS | rc=0 >>
hello

raspberry-3 | SUCCESS | rc=0 >>
hello

raspberry-4 | SUCCESS | rc=0 >>
hello

orangepi-6 | SUCCESS | rc=0 >>
hello

orangepi-7 | SUCCESS | rc=0 >>
hello

raspberry-2 | SUCCESS | rc=0 >>
hello

raspberry-1 | SUCCESS | rc=0 >>
hello
````
Note: In this example 'all' is a group name. Therefor a section [all] should be present within your '$HOME/git/ansible-dump1090/hosts' inventory file containing your dump1090 hostnames!   
  
Now you can do things like updating a ip address of host raspberry-2 in '/etc/hosts' on all your dump1090 hosts:  
````
pi@raspberry-1:~ $ ansible all -a "sudo sed -i 's/192\.168\.11\.7 raspberry-2/192\.168\.11\.34 raspberry-2/' /etc/hosts" 
raspberry-3 | SUCCESS | rc=0 >>


raspberry-4 | SUCCESS | rc=0 >>


raspberry-5 | SUCCESS | rc=0 >>


orangepi-6 | SUCCESS | rc=0 >>


orangepi-7 | SUCCESS | rc=0 >>


raspberry-2 | SUCCESS | rc=0 >>


raspberry-1 | SUCCESS | rc=0 >>
````
  
## Run a playbook  

Using Ansible you can execute commands on multiple hosts. But using Ansible playbooks you can execute scripts (in yml format) that take care of deployment and configuration on multiple hosts.
  
In this example I run a playbook all hosts (as configured in [installbasics.yml](https://github.com/tedsluis/ansible-dump1090/blob/master/installbasics.yml)):
````
$ ansible-playbook installbasics.yml

PLAY [install basic packages and configure basic settings.] ********************

TASK [setup] *******************************************************************
ok: [raspberry-5]
ok: [raspberry-3]
ok: [raspberry-4]
ok: [orangepi-6]
ok: [orangepi-7]
ok: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Install basic packages] *****************************************
ok: [raspberry-5] => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'rsync', u'vim', u'wget'])
ok: [raspberry-3] => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'rsync', u'vim', u'wget'])
ok: [raspberry-4] => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'rsync', u'vim', u'wget'])
ok: [orangepi-6]  => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'rsync', u'vim', u'wget'])
ok: [orangepi-7]  => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'rsync', u'vim', u'wget'])
ok: [raspberry-2] => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'rsync', u'vim', u'wget'])
ok: [raspberry-1] => (item=[u'apt-utils', u'cron', u'curl', u'debhelper', u'git', u'netcat', u'net-tools', u'nmap', u'python2.7', u'rsync', u'vim', u'wget'])

TASK [basics : Creates /home/pi/git directory] *********************************
ok: [raspberry-5]
ok: [raspberry-3]
ok: [raspberry-4]
ok: [orangepi-6]
ok: [orangepi-7]
ok: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Check if /home/pi/connect.dump.sh exists] ***********************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-6]
ok: [orangepi-7]
ok: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Touch /home/pi/connect.dump.sh if not exists] *******************
skipping: [raspberry-3]
skipping: [raspberry-1]
skipping: [raspberry-2]
skipping: [raspberry-4]
skipping: [raspberry-5]
skipping: [orangepi-6]
skipping: [orangepi-7]

TASK [basics : Add /home/pi/connect.dump.sh to /etc/rc.local] ******************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-7]
ok: [orangepi-6]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Insert alias ll into .profile] **********************************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-7]
ok: [orangepi-6]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Insert alias tmp into .profile] *********************************
ok: [raspberry-3]
ok: [raspberry-4]
ok: [orangepi-6]
ok: [orangepi-7]
changed: [raspberry-2]
ok: [raspberry-1]
ok: [raspberry-5]

TASK [basics : Insert alias www into .profile] *********************************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-6]
ok: [orangepi-7]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Insert alias etc into .profile] *********************************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-7]
ok: [orangepi-6]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Insert alias git into .profile] *********************************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-7]
ok: [orangepi-6]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Insert alias log into .profile] *********************************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-7]
ok: [orangepi-6]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : Set vim syntax no (globaly)] ************************************
ok: [raspberry-3]
ok: [raspberry-4]
ok: [orangepi-6]
ok: [orangepi-7]
changed: [raspberry-2]
ok: [raspberry-1]
ok: [raspberry-5]

TASK [basics : Update /etc/hosts from inventory] *******************************
ok: [raspberry-5] => (item=orangepi-7)
ok: [raspberry-3] => (item=orangepi-7)
ok: [raspberry-4] => (item=raspberry-5)
ok: [raspberry-1] => (item=raspberry-2)
ok: [raspberry-5] => (item=orangepi-6)
ok: [raspberry-3] => (item=orangepi-6)
ok: [raspberry-4] => (item=raspberry-4)
ok: [raspberry-5] => (item=raspberry-5)
ok: [raspberry-3] => (item=raspberry-5)
ok: [raspberry-2] => (item=raspberry-2)
ok: [raspberry-4] => (item=raspberry-3)
ok: [raspberry-5] => (item=raspberry-4)
ok: [raspberry-3] => (item=raspberry-4)
ok: [raspberry-3] => (item=raspberry-3)
ok: [raspberry-5] => (item=raspberry-3)
ok: [raspberry-4] => (item=raspberry-1)
ok: [raspberry-3] => (item=raspberry-1)
ok: [raspberry-5] => (item=raspberry-1)
ok: [raspberry-4] => (item=raspberry-2)
ok: [raspberry-3] => (item=raspberry-2)
ok: [raspberry-5] => (item=raspberry-2)
ok: [orangepi-6]  => (item=raspberry-2)
ok: [orangepi-7]  => (item=raspberry-2)
ok: [raspberry-4] => (item=orangepi-6)
ok: [raspberry-2] => (item=raspberry-1)
ok: [orangepi-6]  => (item=raspberry-1)
ok: [orangepi-7]  => (item=raspberry-1)
ok: [orangepi-6]  => (item=raspberry-3)
ok: [orangepi-7]  => (item=raspberry-3)
ok: [raspberry-4] => (item=orangepi-7)
ok: [orangepi-6]  => (item=raspberry-4)
ok: [orangepi-7]  => (item=raspberry-4)
ok: [orangepi-6]  => (item=raspberry-5)
ok: [orangepi-7]  => (item=raspberry-5)
ok: [orangepi-6]  => (item=orangepi-6)
ok: [orangepi-7]  => (item=orangepi-6)
ok: [orangepi-6]  => (item=orangepi-7)
ok: [orangepi-7]  => (item=orangepi-7)
ok: [raspberry-2] => (item=raspberry-3)
ok: [raspberry-1] => (item=raspberry-1)
ok: [raspberry-2] => (item=raspberry-4)
ok: [raspberry-1] => (item=raspberry-3)
ok: [raspberry-2] => (item=raspberry-5)
ok: [raspberry-2] => (item=orangepi-6)
ok: [raspberry-1] => (item=raspberry-4)
ok: [raspberry-2] => (item=orangepi-7)
ok: [raspberry-1] => (item=raspberry-5)
ok: [raspberry-1] => (item=orangepi-6)
ok: [raspberry-1] => (item=orangepi-7)

TASK [basics : schedule apt-get update & upgrade every cronjob friday 0am.] ****
ok: [raspberry-5]
ok: [raspberry-3]
ok: [orangepi-6]
ok: [orangepi-7]
ok: [raspberry-4]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : schedule reboot cronjob every friday 3am.] **********************
ok: [raspberry-3]
ok: [raspberry-5]
ok: [raspberry-4]
ok: [orangepi-6]
ok: [orangepi-7]
changed: [raspberry-2]
ok: [raspberry-1]

TASK [basics : set timezone to Europe/Amsterdam] *******************************
ok: [raspberry-3]
ok: [raspberry-4]
changed: [orangepi-6]
changed: [orangepi-7]
ok: [raspberry-5]
changed: [raspberry-2]
ok: [raspberry-1]

PLAY RECAP *********************************************************************
raspberry-1                  : ok=16   changed=0    unreachable=0    failed=0   
raspberry-2                  : ok=16   changed=12   unreachable=0    failed=0   
raspberry-3                  : ok=16   changed=0    unreachable=0    failed=0   
raspberry-4                  : ok=16   changed=0    unreachable=0    failed=0   
raspberry-5                  : ok=16   changed=0    unreachable=0    failed=0   
orangepi-6                   : ok=16   changed=1    unreachable=0    failed=0   
orangepi-7                   : ok=16   changed=1    unreachable=0    failed=0 
````
  
## Logging
  
Logging is written to '/tmp/ansible.log'.  
  
You can disable logging by putting a # in front of 'log_path=/tmp/ansible.log' in the ansible.cfg file.  

## More info
  
* http://docs.ansible.com/ansible
* ted.sluis@gmail.com
