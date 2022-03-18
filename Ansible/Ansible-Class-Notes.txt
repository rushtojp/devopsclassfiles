https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu

# Prefer to run apt-get update on ansible_node1 and ansible_node2

########### Install ansible

sudo apt update
sudo apt install software-properties-common -y
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y

ansible --version


########### Configure ansible master
########### ansible master  (192.168.56.101)
    - install ansible

    - vi /etc/hosts    
      192.168.56.102  ansible-node1


    - /etc/ansible/hosts
        192.168.56.102 ansible_ssh_user=vagrant
    - ssh-keygen
  
# ansible master 
    - ssh-copy-id vagrant@ansible-node1  # this will copy public key to client under ~/.ssh/authorized_keys file
    - ansible all -m ping
    - ansible ansible-node1 -m ping

**************************
#add ansible-node2 as a node with vagrant user.

vi /etc/hosts    
      192.168.56.103  ansible-node2

# ansible master
/etc/ansible/hosts
        [ansible-nodes]
        192.168.56.102 ansible_ssh_user=vagrant
        192.168.56.103 ansible_ssh_user=ansible_user


# ansible-node2 machine
    - adduser ansible_user   (and give passwd -same as of now)
    - ansible_user ALL=(ALL) NOPASSWD: ALL            # add this line to /etc/sudoers

ssh-copy-id -i ansible_user@ansible-node2

ansible dev -m ping
ansible all -m ping
**************************

------------------------------------------------------------------

# gathering facts    
# https://www.tutorialspoint.com/ansible/ansible_ad_hoc_commands.htm

ansible all -m setup   

https://www.youtube.com/watch?reload=9&v=YI48bKykx7k
https://www.youtube.com/watch?v=4nKW2eF-nIw
https://www.youtube.com/watch?v=XJpN8qpxWbA



https://www.devopsschool.com/courses/ansible/ansible-advance-training.html


ansible-doc -l  # list of modules
ansible-doc -l | wc -l  #count modules
ansible-doc apt   # description of module
docs.ansible.com modules list   # at google



ansible all -m ping
ansible all -m setup


ansible all –a "mkdir ~/test" #create dir at all nodes
ansible all –a  "touch ~/test/newfile"   #create file

root@ansible-master:~# mkdir ansible_code 
root@ansible-master:~# cd ansible_code 
----------------------------------------------------
root@ansible-master:~/ansible_code# vi inventory
[dev]
ansible-node1 ansible_ssh_user=vagrant

[test]
ansible-node2 ansible_ssh_user=ansible_user
-------------------------------------------------------
############ ansible ad-hoc commands #####################################################
ansible all -m file -a "path=/root/ansible_code/tmp-file state=touch"       # create file (change - yellow color)
ansible all -m file -a "path=/root/ansible_code/tmp-file state=touch"       # create file (every time change - yellow color)
ansible all -m file -a "path=/root/ansible_code/tmp-file state=touch"       # create file (every time change - yellow color)
ansible all –a "mkdir ~/tmp" #create dir at all nodes

ansible localhost -m file -a "path=/root/ansible_code/tmp-file/mydir state=directory" 

ansible all -i inventory -m copy -a "src=/root/ansible_code/tmp-file dest=~/tmp"

ansible localhost -m file -a "path=/root/ansible_code/tmp-file state=absent"       # delete file (change - yellow color)
ansible localhost -m file -a "path=/root/ansible_code/tmp-file state=absent"       # delete file (no change - green color)

ansible localhost -m user -a "name=john password=john" -b   # -b means become root user

------------------------------------------------------


ansible all -m ping
ansible dev -m ping
ansible dev1 -m ping   # no such pattern/env
ansible test -m ping

ansible all -m setup
ansible all -m setup -a "filter=*ipv4*"

ansible all -a "df -h" -u root                  # check disk usage
ansible all -a "uptime" -u root 

ansible all -m file -a "path=~/tmp-file state=touch"       # create file (change - yellow color)
ansible all -m file -a "path=~/tmp-file state=touch"       # create file (every time change - yellow color)
ansible all -m file -a "path=~/tmp-file state=touch"       # create file (every time change - yellow color)
ansible all –a "mkdir ~/tmp" #create dir at all nodes

ansible all -m copy -a "src=~/tmp-file dest=~/tmp"

ansible all -m file -a "path=/root/ansible_code/tmp-file state=absent"       # delete file (change - yellow color)
ansible all -m file -a "path=/root/ansible_code/tmp-file state=absent"       # delete file (no change - green color)

#usermod -G sudo ansible_user
ansible all -m user -a "name=john password=john" -b   # -b means become root user
----------------------------------------------
id john    # to verify if user has been created

vi /etc/ansible/ansible.cfg    # configuration file

#inventory
#forks    means no of parallel sessions
 #role_path

ansible-playbook -i inventory-name playbook1.yml --syntax-check
ansible-playbook -i inventory-name playbook1.yml --check
ansible-playbook -i inventory-name playbook1.yml --limit host2 
ansible-playbook -i inventory-name playbook1.yml --list-hosts

---------------------------------------------------
################playbook 1 ################################################################################

root@u64-18-04-ansible-master:~/ansible_code# vi 1-playbook-createfile-specific-host.yml
---

- hosts: dev
  tasks:
    - name: create a file
      file:
        path: /tmp/file-playbook1
        state: touch

-------------------------------------------------------------
root@ansible-master:~/ansible_code# cat inventory
[dev]
ansible-node1 ansible_ssh_user=vagrant

[test]
ansible-node2 ansible_ssh_user=ansible_user
-------------------------------------------------

ansible-playbook 1-playbook-createfile-specific-host.yml              # run only for specific host - dev          
###############################################################################################################
################playbook 2 ################################################################################

root@ansible-master:~/ansible_code# vi 2-playbook-createfile.yml
---

- hosts: all
  tasks:
    - name: create a file
      file:
        path: /tmp/file-playbook-1
        state: touch

-------------------------------------------------------------

ansible-playbook 2-playbook-createfile.yml              
###############################################################################################################
################playbook 3 ####################################################################################

root@ansible-master:~/ansible_code# vi 3-playbook-deletefile.yml
---

- hosts: all
  tasks:
    - name: delete a file
      file:
        path: /tmp/file-playbook-1
        state: absent

-------------------------------------------------------------

ansible-playbook 3-playbook-deletefile.yml                 
###############################################################################################################
################playbook 4 ####################################################################################
root@ansible-master:~/ansible_code# vi 4-playbook-install-webserver-apache.yml
---

-  hosts : ansible-node1
   become: true
   name: 4-playbook-install-webserver-apache
   tasks:
     - name: Install Apache
       apt:
           name: apache2
           state: latest
     - name: add website page
       script: hello.sh


---------------------------------------------------------------------------------------
ansible-playbook 4-playbook-install-webserver-apache.yml --syntax-check
ansible-playbook 4-playbook-install-webserver-apache.yml
###############################################################################################################
################playbook 5 ####################################################################################
root@ansible-master:~/ansible_code# vi 5-playbook-install-webservers-selective.yml
---

-  hosts : dev
   become: true
   name: 5-playbook-install-webservers
   tasks:
     - name: Install Apache
       apt:
           name: apache2
           state: latest
     - name: add website page
       script: hello.sh

-  hosts : test
   become: true
   name: 5-playbook-install-webservers
   tasks:
     - name: Install Nginx
       apt:
           name: nginx
           state: latest
-------------------------------------------------------------------
ansible-playbook 5-playbook-install-webservers-selective.yml
###############################################################################################################
################playbook 6 ####################################################################################
root@ansible-master:~/ansible_code# vi 6-playbook-remove-webservers-selective.yml
---

-  hosts : dev
   become: true
   name: 6-playbook-remove-webservers
   tasks:
     - name: un-install Apache
       apt:
           name: apache2
           state: absent

-  hosts : test
   become: true
   name: 5-playbook-remove-webservers
   tasks:
     - name: un-install Nginx
       apt:
           name: nginx-common
           state: absent

-------------------------------------------------------------------
ansible-playbook 6-playbook-remove-webservers-selective.yml
###############################################################################################################
################playbook 7 ####################################################################################
root@ansible-master:~/ansible_code# vi 7-playbook-with_loop-install-packages.yml
---

-  hosts : dev
   become: true
   name: playbook-install
   tasks:
     - name: Install Packages
       apt:
           name: "{{ item }}"
           state: latest
       with_items:
           - vim
           - git
           - curl

-------------------------------------------------------------------
ansible-playbook 7-playbook-with_loop-install-packages.yml
###############################################################################################################
################playbook 8 ####################################################################################
root@ansible-master:~/ansible_code# vi 8-playbook-with_loop-remove-packages.yml
---

-  hosts : dev
   become: true
   name: playbook-install
   tasks:
     - name: Remove Packages
       apt:
           name: "{{ item }}"
           state: absent
       with_items:
           - vim
           - git
           - curl

-------------------------------------------------------------------
ansible-playbook 8-playbook-with_loop-remove-packages.yml
###############################################################################################################
################playbook 9 ####################################################################################
root@ansible-master:~/ansible_code# vi 9-playbook-array-install-packages.yml
---

-  hosts : test
   become: true
   vars:
       packages: [ 'vim', 'git', 'curl' ]
   tasks:
     - name: Install Packages
       apt:
           name: "{{ item }}"
           state: latest
       with_items: "{{ packages }}"

-------------------------------------------------------------------
ansible-playbook 9-playbook-array-install-packages.yml
###############################################################################################################
################playbook 10 ####################################################################################
root@ansible-master:~/ansible_code# vi 10-playbook-array-remove-packages.yaml
---

-  hosts : test
   become: true
   vars:
       packages: [ 'vim', 'git', 'curl' ]
   tasks:
     - name: Remove Packages
       apt:
           name: "{{ item }}"
           state: absent
       with_items: "{{ packages }}"

-------------------------------------------------------------------
ansible-playbook 10-playbook-array-remove-packages.yaml
###############################################################################################################
################playbook 11 ####################################################################################
root@ansible-master:~/ansible_code# vi 11-playbook-with_loop-create-files.yml
---
- hosts: dev
  tasks:
    - name: create files
      file:
        path: /tmp/{{item}}
        state: touch
      with_items:
        - file1
        - file2
        - file3
-------------------------------------------------------------------
ansible-playbook 11-playbook-array-remove-packages.yaml
###############################################################################################################
################playbook 12 ####################################################################################
root@ansible-master:~/ansible_code# vi 12-playbook-with_loop-delete-files.yml
---
- hosts: dev
  tasks:
    - name: delete files
      file:
        path: /tmp/{{item}}
        state: absent
      with_items:
        - file1
        - file2
        - file3
-------------------------------------------------------------------
ansible-playbook 12-playbook-with_loop-delete-files.yaml
###############################################################################################################
--------------------------------------------
vault.yml

ansible-vault create vault.yml   #will ask for password to create a new enctypted password  # vault used AES256
ansible-vault view vault.yml   # to display the conents of vault file in plain text mode
ansible-vault edit vault.yml  # edit encrypted playbook
ansible-vault rekey vault.yml # change password
ansible-vault encrypt target.yml
ansible-vault decrypt target.yml

################playbook 13 ####################################################################################
root@ansible-master:~/ansible_code# vi 13-create-user.yml
---
- name: Create New Users
  hosts: all
  become: true
  gather_facts: false
  vars_files:
    - my_vault_create_user.yml
  tasks:
    - name: Create Users
      user:
        name: "{{ item }}"
        password: "{{ my_password | password_hash('sha512') }}"
        shell: /bin/bash
        #update_password: on_create    #to avoid updating password hash in /etc/shadow
      loop:
        - alice
        - vincent
-------------------------------------------------
ansible-vault create my_vault_create_user.yml    #my_vault
my_password: mysecret$1      
ansible-vault view my_vault_create_user.yml

ansible-playbook --ask-vault-pass 13-create-user.yml


--------------------------------------------
Ansible roles

ansible-galacy --version
mkdir -p playbooks/roles
cd playbooks/roles
ansible-galaxy init webserver
vi webserver/tasks/main.yml
- name: install apache2
  action: apt name=apache2 state=latest

cd ..
root@ansible-master:~/ansible_code/playbooks#
vi master.yml

 - hosts: test
   become: yes
   roles:
     - webserver

root@ansible-master:~/ansible_code/playbooks# ansible-playbook master.yml
----------------------------------------------------------------------------------












########### ansible playbook - playbook delete

root@u64-18-04-ansible-master:~/ansible-code# cat playbook-delete.yaml
---

-  hosts : slave1
   become: true
   name: playbook-delete
   tasks:
     - name: Uninstall Apache
       apt:
          name: apache2
          state: absent
     - name: Remove unwanted Apache2 packages from the system.
       apt:
          autoremove: yes
          purge: yes


########### ansible playbook 2 - playbook install

---

-  hosts : slave1
   become: true
   name: playbook-install
   tasks:
     - name: Install Apache
       apt:
           name: apache2
           state: latest
     - name: add website page
       script: hello.sh

-  hosts : slave2
   become: yes
   name: playbook-install
   tasks:
     - name: Install Nginx
       apt:
           name: nginx
           state: latest

=============
root@u64-18-04-ansible-master:~/ansible-code# cat hello.sh
#!/bin/sh

echo hello world > /var/www/html/1.html     # 192.168.56.102/1.html

=============
# changing hello.sh contents at master, you can change the nodes without logging into the node

root@u64-18-04-ansible-master:~/ansible-code# cat hello.sh
#!/bin/sh

echo hello world updated > /var/www/html/1.html     # 192.168.56.102/1.html




########### ansible playbook 2 - playbook delete

---

-  hosts : slave1
   become: yes
   name: playbook-delete
   tasks:
     - name: remove Apache
       apt: name=apache2 state=absent

-  hosts : slave2
   become: yes
   name: playbook-delete
   tasks:
     - name: remove nginx
       apt: name=nginx-common state=absent

=======================================================================


------------------------------------------------------------------------

root@u64-18-04-ansible-master:~/ansible-code# cat playbook-createfile-register.yml

---

- hosts: localhost
  tasks:
    - name: create a file
      file:
        path: /tmp/file1
        state: touch
      register: output
    - debug: 'msg="this is the value of register variable {{ output }}"'
    - name: edit file
      lineinfile:
        path: /tmp/file1
        line: '{{ ansible_hostname }} - {{ output.uid }}'

cat /tmp/file1
hostname - id



https://www.digitalocean.com/community/tutorials/configuration-management-101-writing-ansible-playbooks

root@u64-18-04-ansible-master:~/ansible-code# cat playbook-createfile-with.yml
---
- hosts: localhost
  tasks:
    - name: create files
      file:
        path: /tmp/{{item}}
        state: touch
      with_items:
        - file1
        - file2
        - file3



root@u64-18-04-ansible-master:~/ansible-code# cat playbook-loop.yaml
---

-  hosts : slave1
   become: true
   name: playbook-install
   tasks:
     - name: Install Packages
       apt:
           name: "{{ item }}"
           state: latest
       with_items:
           - vim
           - git
           - curl
root@u64-18-04-ansible-master:~/ansible-code# cat playbook-loop-remove-packages.yaml
---

-  hosts : slave1
   become: true
   name: playbook-install
   tasks:
     - name: Remove Packages
       apt:
           name: "{{ item }}"
           state: absent
       with_items:
           - vim
           - git
           - curl
root@u64-18-04-ansible-master:~/ansible-code# cat playbook-array.yaml
---

-  hosts : slave1
   become: true
   vars:
       packages: [ 'vim', 'git', 'curl' ]
   tasks:
     - name: Install Packages
       apt:
           name: "{{ item }}"
           state: latest
       with_items: "{{ packages }}"
root@u64-18-04-ansible-master:~/ansible-code# cat playbook-array-remove-packages.yaml
---

-  hosts : slave1
   become: true
   vars:
       packages: [ 'vim', 'git', 'curl' ]
   tasks:
     - name: Remove Packages
       apt:
           name: "{{ item }}"
           state: absent
       with_items: "{{ packages }}"











--------------------
links

https://linuxhint.com/begineers_guide_tutorial_ansible/

https://medium.com/@ahmadfarag/ansible-in-action-f2f17706931

https://techexpert.tips/ansible/ansible-playbook-examples-ubuntu-linux/

https://www.tecmint.com/create-ansible-plays-and-playbooks/

https://www.digitalocean.com/community/tutorials/configuration-management-101-writing-ansible-playbooks

https://blog.ssdnodes.com/blog/step-by-step-ansible-guide/

https://www.tutorialspoint.com/ansible/ansible_yaml_basics.htm


===================================================================================
root@u64-18-04-ansible-master:~/ansible-code# cat playbook-variable.yaml
---
-  hosts : all
   become: true
   vars:
       package: vim
   tasks:
     - name: Install Package
       apt:
           name: "{{ package }}"
           state: latest
