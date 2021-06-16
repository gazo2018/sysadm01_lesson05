# Lesson 4. Home Work. Practice
## Package managment
### Environment Setup:
 1. Open Cloud shell: https://shell.cloud.google.com/
 2. Switch to *sysadm01* project: `gcloud config set project sysadm01`
 3. Set Default zone: `gcloud config set compute/zone us-central1-a`

### Debian Practice:
 4. Create debian instance: 
`gcloud compute instances create lesson4-debian --image-project debian-cloud --image-family debian-10`
 5. Get remote shell to debian instance
	- Get list of running instances: `gcloud compute instances list`
	- Find *EXTERNAL_IP* of instance *lesson4-debian* 
	- Run `ssh <EXTERNAL_IP>` to login
 6. Install Apache web Server:
	- Run  `sudo apt install -y apache2` to install apache package and skip confirmation
 7. Install nginx from deb package
	- To download nginx package from the internet run 
	`curl http://nginx.org/packages/debian/pool/nginx/n/nginx/nginx_1.20.0-1~buster_amd64.deb` 
	- Install the package using dpkg tool: `sudo dpkg -i nginx_1.20.0-1~buster_amd64.deb`
 8. Remove Apache and Nginx package 
	- Run `sudo apt remove -y apache2` to remove apache2 package and skip confirmation
 	- Run `sudo apt remove -y nginx` to remove nginx package and skip confirmation
 9. Update all outdated packages
	- Run `sudo apt update` to update local package database
	- Run `sudo apt -y upgrade` to upgrade all packages and skip confirmation
10. User Management
	- Run `sudo useradd sysadm` to create `sysadm` user in the system
	- Run `sudo usermod -a -G adm sysadm` to add *sysadm* user to *adm* group
	- Run `sudo usermod -m -d /var/lib/sysadm sysadm` to change home directory to */var/lib/sysadm* and move content to the new home folder
	- Run `sudo gpasswd -d sysadm adm` to remove user from adm group 
    - Run `sudo userdel sysadm` to remove user from the system
11. Close remote shell and terminate instnce
	-   Run `exit` or press `ctrl + d` to close remote shell
	-   Run `gcloud compute instances delete lesson4-debian` to remove Debian instance

### CentOS Practice:
12. Create CentOS instance
		`gcloud compute instances create lesson4-centos --image-project centos-cloud --image-family centos-7`
13. Get remote shell to CentOS instance
	- Get list of running instances: `gcloud compute instances list`
	- Find *EXTERNAL_IP* of instance *lesson4-centos* 
	- Run `ssh <EXTERNAL_IP>` to login
14. Install Apache Web Server
	-   Run `sudo yum install -y httpd` to install apache web server and skip confirmation
15. Install nginx from rpm package by URL
	-   Install the package using rpm tool: 
`sudo rpm -i http://nginx.org/packages/centos/7/x86_64/RPMS/nginx-1.20.1-1.el7.ngx.x86_64.rpm`
16. Remove Apache and Nginx package 
	- Run `sudo yum remove -y httpd` to remove httpd package and skip confirmation
 	- Run `sudo yum remove -y nginx` to remove nginx package and skip confirmation
17. Update all outdated packages
	-   Run `sudo yum -y upgrade` to upgrade all packages and skip confirmation
18. User Management
	- Run `sudo useradd sysadm` to create `sysadm` user in the system
	- Run `sudo usermod -a -G adm sysadm` to add *sysadm* user to *adm* group
	- Run `sudo usermod -m -d /var/lib/sysadm sysadm` to change home directory to */var/lib/sysadm* and move content to the new home folder
	- Run `sudo gpasswd -d sysadm adm` to remove user from adm group 
    - Run `sudo userdel sysadm` to remove user from the system
19. Close remote shell and terminate instnce
	-   Run `exit` or press `ctrl + d` to close remote shell
	-   Run `gcloud compute instances delete lesson4-centos` to remove CentOS instance
### Ansible Practice:
20. Install Ansible on cloud shell instance
	-  Run `sudo pip3 install --upgrade pip` to upgrade PIP verison
	-  Run `sudo pip3 install ansible`  to Install ansible
	-  Run `ansible --version` to make sure installation was successful
21.  Create ansible play
		-  Run `mkdir ansible` to create ansible folder
		-  Run `cd ansible` to change directory to ansible folder
		-   Use *nano* or *vi* `nano ./lesson4.yml` to start editing yaml file and add following content:
```
---
- name: Lesson 4 Playbook
  hosts: web_servers
  gather_facts: true
  become: true
  vars:
    apache_pkg: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' if ansible_os_family == 'RedHat' }}"
  tasks:
  - name: Create user
    user:
      name: sysadm
      shell: /bin/bash
      groups: adm
      append: yes
      create_home: yes
      home: /var/lib/sysadm

  - name: Install apache web server
    package:
      name: "{{ apache_pkg }}"
      state: latest
  - name: Start Apache web server
    service:
      name: "{{ apache_pkg }}"
      state: started
      enabled: true
```
22. Create inventory folder
	-  Run `mkdir inventory` to create the folder
    -  Use *nano* or *vi* `nano ./inventory/my.gcp.yml` to start editing yaml file and add following content:
```
plugin: gcp_compute
zones:
  - us-central1-a
projects:
  - sysadm01
filters: []
auth_kind: application
keyed_groups:
  - key: zone
groups:
  web_servers: "'lesson4-' in name"
```
23. Create Ansible configuration file
	- Use *nano* or *vi* `nano ./ansible.cfg` to start editing cfg file and add following content:
```
[inventory]
enable_plugins = gcp_compute
```  
24. Create FireWall rule for HTTP server
	-   To create FW rule with *http-server* tag and allow *incoming* connections to port *80* run: 
`gcloud compute firewall-rules create webserveraccess --target-tags http-server --allow TCP:80`
25. Create debian and centos instances
	-   Debian instance: 
`gcloud compute instances create lesson4-debian --image-project debian-cloud --image-family debian-10 --tags http-server`
	-   CentOS instance: 
`gcloud compute instances create lesson4-centos --image-project centos-cloud --image-family centos-7 --tags http-server`
26. Validate ansible GCP plugin:
	-   Run `ansible-inventory -i ./inventory/my.gcp.yml --list`
    -   At the end of long JSON output You should see (Ip addresses will be different):
```
"web_servers": {
  "hosts": [
    "34.122.56.118",
    "35.238.218.255"
  ]
}
```
27. Apply Ansible playbook
	-   Run `ansible-playbook -i ./inventory/my.gcp.yml ./lesson4.yml -vv`
28. Validate ansible provision
	- Run `ansible -i ./inventory/my.gcp.yml web_servers -m shell -a 'netstat -tulnp|grep 80'` 
	- Analyze output, make sure both hosts have port 80 open
	- Run `ansible -i ./inventory/my.gcp.yml web_servers -m shell -a 'sudo cat /etc/passwd|grep sysadm'` 
	- Analyze output, make sure both hosts have *sysadm* user and home folder is set to `/var/lib/sysadm`
29. Remove instances
	-   Delete Debian instance: `gcloud compute instances delete lesson4-debian`
	-   Delete CentOS instance: `gcloud compute instances delete lesson4-centos`
30. Make sure you don't have any running instances:
	- Run `gcloud compute instances list`
