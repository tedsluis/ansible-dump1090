# Reboot hosts

Playbook [reboot](https://github.com/tedsluis/ansible-dump1090/blob/master/roles/reboot/tasks/main.yml) performs a reboot of groups of hosts in three batches:  
* First batch immediately.  
* Second batch after 3 minutes.  
* Third batch after 6 minutes.  
  
Hosts gets 1 minute extra delay to cancel reboot.  
Playbook checks whether the reboot is successful.  
 
Ansible inventory file '[hosts](https://github.com/tedsluis/ansible-dump1090/blob/master/hosts)' sections|group_vars|reboot delay (minutes)|
---------------------------------|-------|--------|
reboot-order-first|[group_vars/reboot-order-first](https://github.com/tedsluis/ansible-dump1090/blob/master/group_vars/reboot-order-first)|0+1|
reboot-order-second|[group_vars/reboot-order-second](https://github.com/tedsluis/ansible-dump1090/blob/master/group_vars/reboot-order-second)|3+1|
reboot-order-third|[group_vars/reboot-order-third](https://github.com/tedsluis/ansible-dump1090/blob/master/group_vars/reboot-order-third)|6+1|
  
Example:  
````
$ ansible-playbook reboot.yml 

PLAY [reboot hosts] ************************************************************

TASK [setup] *******************************************************************
ok: [raspberry-5]
ok: [orangepi-6]
ok: [orangepi-7]
ok: [raspberry-3]
ok: [raspberry-4]
ok: [raspberry-2]
ok: [raspberry-1]

TASK [reboot : wait and reboot.] ***********************************************
changed: [orangepi-6]
changed: [orangepi-7]
changed: [raspberry-5]
changed: [raspberry-2]
changed: [raspberry-1]
changed: [raspberry-3]
changed: [raspberry-4]

TASK [reboot : Waiting for the host to come back.] *****************************
ok: [raspberry-2 -> localhost]
ok: [orangepi-7  -> localhost]
ok: [raspberry-3 -> localhost]
ok: [raspberry-4 -> localhost]
ok: [orangepi-6  -> localhost]
ok: [raspberry-1 -> localhost]
ok: [raspberry-5 -> localhost]

PLAY RECAP *********************************************************************
raspberry-1                  : ok=3    changed=1    unreachable=0    failed=0   
raspberry-2                  : ok=3    changed=1    unreachable=0    failed=0   
raspberry-3                  : ok=3    changed=1    unreachable=0    failed=0   
raspberry-4                  : ok=3    changed=1    unreachable=0    failed=0   
raspberry-5                  : ok=3    changed=1    unreachable=0    failed=0   
orangepi-6                   : ok=3    changed=1    unreachable=0    failed=0   
orangepi-7                   : ok=3    changed=1    unreachable=0    failed=0  
````
