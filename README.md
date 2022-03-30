# WeightTracker Deployment Using Ansible

---
Structure:

```.
├── inventories
│   ├── stage
│   │   ├── group_vars
│   │   │   └── stageservers.yml
│   │   └── hosts
│   └── test
│       ├── group_vars
│       │   └── testservers.yml
│       └── hosts
├── main_playbook.yml
└── roles
    └── common
        └── tasks
            └── main.yml
			
			
			
commands:
ansible-playbook -v -i inventories/staging main_playbook.yml
ansible-playbook -v -i inventories/production main_playbook.yml

```
# Instructions

 - Install ansible on ubuntu 18.04
 - Create ssh key and copy to remote machine
 - Configure remote machine to enable ansible to run it.

---
## prerequisites (ON MASTER/CONTROLLER)
```
1.SSH to the loadbalancer fornted ip -> get from run output.
2.ssh from loadbalancer  to the controller machine.
3.Deploy all prerequisites using the attached script (weightTrackerDepoly.sh) The script will deploy ansible + python3
-> copy the file to the controller machine and chnge permissions using: weightTrackerDepoly.sh cmod +x 
- run the script and type "yes" for all.

```

## Generate ssh key and copy to remote nodes
# Do it from the Master node to each node (Slave)
```
$ ssh-keygen
press enter for filename
Overwrite - Y
press enter twice with passphrase empty

Validate that key was created:
rootAdmin@Controller-Server:~$ sudo ls -la /root/.ssh/
total 20
drwx------ 2 root root 4096 Mar 30 14:17 .
drwx------ 5 root root 4096 Mar 30 13:47 ..
-rw------- 1 root root    0 Mar 30 13:15 authorized_keys
-rw------- 1 root root  399 Mar 30 14:17 id_ed25519
-rw-r--r-- 1 root root   89 Mar 30 14:17 id_ed25519.pub
-rw-r--r-- 1 root root  222 Mar 30 14:16 known_hosts

rootAdmin@Controller-Server:~$ cat  /root/.ssh/id_ed25519.pub
rootAdmin@Controller-Server:~$ sudo cat  /root/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMG4PCYGOnH8arOQUHD/EgwYJOiUYGzeArNmNvFN07Lc ansible

```
```
copy the public key to all connected nodes (you can see it from the terraform
$ ssh-copy-id -i (username)@(node machine ip) - For example - ssh-copy-id -i sela@34.76.142.118
When promped whether to continue connecting press yes
```

## Configure remote machines (Slaves) to enable ansible to run commands on it.
```
$ ssh (username)@(node machine ip)
$ sudo apt upgrade -y
$ sudo apt install python
```

## Change the file value for pubkeyAuthentication to "yes"
```
$ sudo vim /etc/ssh/sshd_config
press i
```

## Search for the pubkeyAuthentication attribute and change its value from no to yes - If it is already set to "yes" you can exit the editor with pressing esc and then press :q
```
#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

PubkeyAuthentication no

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none'
```

  
## Save changes
```
Press esc
press :wq
```

## Restart the ssh service and exit to control machine - if the value is correct already - skip.
```
$ sudo service ssh restart
$ sudo systemctl reload sshd

```

## Exit node machine
```
$ exit

```

# Repeat the copy key and flag check for ALL nodes!


## Edit the hosts file (On Master)

### Update hosts file for production in editor
```
$ sudo vi ~/weightTracker-ansible/inventories/production/hosts
type i (for insert)
Update the ip according to the vmss instances ips.

```
### Save entry and exit to terminal
```
press esc
press :wq
```

### Update hosts file for staging in editor
```
$ sudo vi ~/weightTracker-ansible/inventories/staging/hosts
type i (for insert)
Update the ip according to the vmss instances ips.

```
### Save entry and exit to terminal
```
press esc
press :wq
```

### Update the productionservers.yml file and stagingservers.yml file 
```
$ sudo vi ~/weightTracker-ansible/inventories/production/group_vars/productionservers.yml
press i
update the LB_front_ip -> you can take it from the TERRAFORM output.
press esc
press :wq
```

## Ensure valid connection between control machine and managed node machine
```
$ ansible -m ping all
``` 

The result should be as : 

(node machine ip) | SUCCESS => {
    "changed": false,
    "ping": "pong"
}



