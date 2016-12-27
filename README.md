# ansible-dump1090
Ansible playbooks for dump1090. Make it so!

Before start a playbook: Change the default location for ansible.cfg using:
````
export ANSIBLE_CONFIG="$HOME/git/ansible-dump1090/ansible.cfg"
````
  
Add your dump1090 hosts to the hosts file, for example:
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
ted1090-2
ted1090-3
ted1090-4
ted1090-5
ted1090-6
ted1090-7
````
