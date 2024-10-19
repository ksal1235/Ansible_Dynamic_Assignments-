# Ansible Dynamic Assignments (Include) and Community Roles- 101

Last 2 projects have already equipped with some knowledge and skills on Ansible, so we can perform configurations using playbooks, roles and imports. Now we will continue configuring UAT servers learning and practicing new Ansible concepts and modules.

#### In this project we will introduce dynamic assignments by using include module.

Now you may be wondering, what is the difference between static and dynamic assignments?

Well, from previous project, We can already tell that static assignments use import Ansible module. The module that enables dynamic assignments is include.

Hence,

```
import = Static
include = Dynamic
```

When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

#### Take note that in most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.


## Introducing Dynamic Assignment Into Our structure:

1) In your https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.
![image](https://github.com/user-attachments/assets/de34523f-570b-4192-9f4b-3e281a260597)

Create a new folder, name it dynamic-assignments. 
![image](https://github.com/user-attachments/assets/c99b8b10-c3ed-4d78-a65c-397d10995e82)

Then inside this folder, create a new file and name it env-vars.yml. 
![image](https://github.com/user-attachments/assets/defcbe03-b57e-44d4-8949-9534fda5521f)

We GitHub shall have following structure by now.
![image](https://github.com/user-attachments/assets/801bd07f-b720-4e8f-b8bb-f7e0f3aab50a)

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

- Create another directory and call it env-vars.
- Inside it, create a new YAML file for each environment, that is dev.yml prod.yml staging.yml uat.yml.

![image](https://github.com/user-attachments/assets/494b032b-9b72-43d9-bfad-f3369f575bdb)

Now paste the instruction below into the env-vars.yml file.

```
---
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always

```
![image](https://github.com/user-attachments/assets/9eb3c74a-1830-4773-bf62-9efb66e610f0)

Notice 3 things to notice here:

1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:

- include_role
- include_tasks
- include_vars

In the same version, variants of import were also introduces, such as:
- import_role
- import_tasks

2. We made use of a special variables {{ playbook_dir }} and {{ inventory_file }}. {{ playbook_dir }} will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. {{ inventory_file }} on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

#### Update site.yml with dynamic assignments.

```
---
- hosts: all
  name: Include dynamic variables
  tasks:
    import_playbook: ../static-assignments/common.yml
    include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

- hosts: webservers
  name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml

```
![image](https://github.com/user-attachments/assets/aa6f231d-918d-43ad-9874-12238b14b12d)

#### Community Roles:
Now it is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

#### Download Mysql Ansible Role

We will be using a MySQL role developed by geerlingguy.

```
git init
git pull https://github.com/ksal1235/ansible-config-mgt.git
git remote add origin https://github.com/ksal1235/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```
![image](https://github.com/user-attachments/assets/fc7dbf29-83b3-4dde-be71-bb1393e67201)

Inside roles directory create new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql

```
ansible-galaxy install geerlingguy.mysql
```
![image](https://github.com/user-attachments/assets/d68fe81e-45dc-40f9-bf24-3d9d827745ae)

Now Rename the downloaded folder to mysql
```
         mv geerlingguy.mysql/ mysql
```
- Inside the mysql/vars/main.yml , configure your db credentials. P.S: these credentials will be used to connect to our website later on.

Need to go inside the mysql/vars/main.yml.

```
mysql_root_password: ""
mysql_databases:
  - name: tooling
    encoding: utf8
    collation: utf8_general_ci
mysql_users:
  - name: webaccess
    host: "172.31.0.0/20"
    password: Admin123
    priv: "tooling.*:ALL"
```

![image](https://github.com/user-attachments/assets/60c34efa-c484-4137-9036-ffd629718eee)
Save It

#### Create a new playbook inside static-assignments folder and call it db-servers.yml , update it with the created roles. use the code below.

```
- hosts: db_servers
  become: yes
  vars_files:
    - vars/main.yml
  roles:
    - { role: mysql }
```
Now it is time to upload the changes into your GitHub:
![image](https://github.com/user-attachments/assets/0b62ada1-542e-452c-84b5-61cb308eaa79)

Save , Now return to playbook which is the playbooks/site.yml and reference the newly created db-servers playbook, add the code below to import it into the main playbook

```
- import_playbook: ../static-assignments/db-servers.yml
```
![image](https://github.com/user-attachments/assets/98dfff3f-09cf-4a33-bb8d-6cc334df6692)

Save and exit, Create a pull request and merge with the main branch of your git hub repository.


Now it is time to upload the changes into your GitHub:

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
![image](https://github.com/user-attachments/assets/bf356fa7-c126-45e4-bb56-c8d80f21b505)

Merging the Code.
![image](https://github.com/user-attachments/assets/0ab74003-f67e-4ff4-8975-959a9323f053)


## Creating roles for load balancer

For this project , we will be making use of NGINX and APACHE as load balancers, so we need to create roles for them using same method as we did for mysql

- Download and install roles for apache , we can get this role from same source as mysql.

```
ansible-galaxy role install geerlingguy.apache
```
- Rename the folder to apache
```
mv geerlingguy.apache/ apache
```
- Download and install roles for nginx also.
```
ansible-galaxy role install geerlingguy.nginx
```
- Rename the folder to nginx
```
mv geerlingguy.nginx/ nginx
```
![image](https://github.com/user-attachments/assets/7296c3de-0398-4d71-a366-8455c63d3a6c)

#### Since we cannot use both apache and nginx load balancer at the same time, it is advisable to create a condition that enables either one of the two, to do this:

- Declare a variable in roles/apache/defaults/main.yml file inside the apache role , name the variable enable_apache_lb

#### Declare another variable that ensures either one of the load balancer is required and set it to false.

```
enable_apache_lb: false
load_balancer_is_required : false
```

- Create a new playbook in ``static-assignments`` and call it ``loadbalancers.yml``, update it with code below:

![image](https://github.com/user-attachments/assets/a6569b32-5673-4353-998d-282f7cafb2cd)


```
---
- hosts: lb
  become: yes
  roles:
    - role: nginx
      when: enable_nginx_lb | bool and load_balancer_is_required | bool
    - role: apache
      when: enable_apache_lb | bool and load_balancer_is_required | bool
```
![image](https://github.com/user-attachments/assets/9efe0564-e11b-4087-93b9-af15c8883a7d)

save and exit.


Now , inside your general playbook (site.yml) file, dynamically import the load balancer playbook so it can use the roles weve created

```
---
- hosts: all
  name: Include dynamic variables
  become: yes
  tasks:
    - include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always

- import_playbook: ../static-assignments/common.yml
- import_playbook: ../static-assignments/uat_webservers.yml
- import_playbook: ../static-assignments/loadbalancers.yml
- import_playbook: ../static-assignments/db-servers.yml

```

Your site.yml should like the output below

![image](https://github.com/user-attachments/assets/13fd3299-dc7a-4ed0-bcf1-6c378c872277)


#### To activate load balancer, and enable either of Apache or Nginx load balancer, we can achieve this by setting these in the respective environment's env-vars file.

- Open the env-vars/uat.yml file and set it .
```
---
load_balancer_is_required: true
enable_nginx_lb: true
enable_apache_lb: false

```
![image](https://github.com/user-attachments/assets/2432cd8f-f5e5-4172-b24a-ca5cc3ca0abd)
- To use apache, we can set the enable_apache_lb variable to true, and enable_nginx_lb to false. do the same thing for nginx if you want to enable nginx load balancer.


## Configuring the apache and Nginx roles to work as load balancer.

## For Apache

- In the ```roles/apache/tasks/main.yml``` file, we need to include a task that tells ansible to first check if nginx is currently running and enabled, if it is, ansible should first stop and disable nginx before proceeding to install and enable apache. this is to avoid confliction and should always free up the port 80 for the required load balancer. use the code beow to achieve this :

```
- name: Check if nginx is running
  ansible.builtin.service_facts:

- name: Stop and disable nginx if it is running
  ansible.builtin.service:
    name: nginx
    state: stopped
    enabled: no
  when: "'nginx' in services and services['nginx'].state == 'running'"
  become: yes
```

![image](https://github.com/user-attachments/assets/c4a9adcb-889c-4ff0-a776-f4a148e91942)

#### To use apache as a load balancer, we will need to allow certain apache modules that will enable the load balancer. this is the APACHE A2ENMOD

- In the roles/apache/tasks/configure-debian.yml file, Create a task to install and enable the required apache a2enmod modules, use the code below :

```
- name: Enable Apache modules
  ansible.builtin.shell:
    cmd: "a2enmod {{ item }}"
  loop:
    - rewrite
    - proxy
    - proxy_balancer
    - proxy_http
    - headers
    - lbmethod_bytraffic
    - lbmethod_byrequests
  notify: restart apache
  become: yes
```

- Create another task to update the apache configurations with required code block needed for the load balancer to function. use the code below :

```
- name: Insert load balancer configuration into Apache virtual host
  ansible.builtin.blockinfile:
  path: /etc/apache2/sites-available/000-default.conf
  block: |
    <Proxy "balancer://mycluster">
      BalancerMember http://<webserver1-ip-address>:80
      BalancerMember http://<webserver2-ip-address>:80
      ProxySet lbmethod=byrequests
    </Proxy>
    ProxyPass "/" "balancer://mycluster/"
    ProxyPassReverse "/" "balancer://mycluster/"
  marker: "# {mark} ANSIBLE MANAGED BLOCK"
  insertbefore: "</VirtualHost>"
notify: restart apache
become: yes
```
![image](https://github.com/user-attachments/assets/0a519201-14f3-4353-8e31-65c697309198)

Enable Apache (in env-vars/uat.yml)
![image](https://github.com/user-attachments/assets/1941d0cb-7545-4b5f-ba17-f8232d803225)
- Save and create a pull request to merge with the main branch of your github repo.

![image](https://github.com/user-attachments/assets/edf96034-b6e6-4420-bd67-5488fefe1384)

![image](https://github.com/user-attachments/assets/7a77e609-ea23-4e46-8218-bdef358a8145)

### For Nginx

- In the roles/nginx/tasks/main.yml file, create a similar task like we did above to check if apache is active and enabled, if it is, it should disable and stop apache before proceeding with the tasks of installing nginx. use the code below:

```
- name: Check if Apache is running
  ansible.builtin.service_facts:
                     
- name: Stop and disable Apache if it is running
  ansible.builtin.service:
    name: apache2 
    state: stopped
    enabled: no
  when: "'apache2' in services and services['apache2'].state == 'running'"
  become: yes
```

![image](https://github.com/user-attachments/assets/6d6d5c92-2757-49bb-8d01-a138df2e8457)

- In the ``roles/nginx/handlers/main.yml`` file, set nginx to always perform the tasks with sudo privileges, use the function : become: yes to achieve this.
- Do the same for all tasks that require sudo privileges

![image](https://github.com/user-attachments/assets/0e17eb0b-e6cb-4fc3-bc34-80f98a6ba737)

- In the ``role/nginx/defaults/main.yml`` file, uncomment the nginx_vhosts, and nginx_upstream section.
- Under the nginx_vhosts section, ensure you have the same code :

```
nginx_vhosts:
 - listen: "80" # default: "80"
    server_name: "example.com" 
    server_name_redirect: "example.com"
    root: "/var/www/html" 
    index: "index.php index.html index.htm" # default: "index.html index.htm"

# filename: "nginx.conf" # Can be used to set the vhost filename.
                   
    locations:
      - path: "/"
        proxy_pass: "http://myapp1"
                   
# Properties that are only added if defined:
    server_name_redirect: "www.example.com" # default: N/A
    error_page: ""
    access_log: ""
    error_log: ""
    extra_parameters: "" # Can be used to add extra config blocks (multiline).
    template: "{{ nginx_vhost_template }}" # Can be used to override the `nginx_vhost_template` per host.
    state: "present" # To remove the vhost configuration.

```
![image](https://github.com/user-attachments/assets/c67d4741-9cfe-40b6-afe8-1faec9afe852)

- Under the nginx_upstream section, you wil need to update the servers address to include your webservers or uat servers.

```
nginx_upstreams: 
- name: myapp1
  strategy: "ip_hash" # "least_conn", etc.
  keepalive: 16 # optional
  servers:
    - "172.31.11.6 weight=5"
    - "172.31.11.238 weight=5"
```
![image](https://github.com/user-attachments/assets/2ab0e1a8-34ac-4890-94bb-78346768c768)

- finally, update the inventory/uat.yml to include the neccesary details for ansible to connect to each of these servers to perform all the roles we have specified. use the code below :

Update ``roles/nginx/templates/nginx.conf.j2`` Comment the line `` include {{ nginx_vhost_path }}/*; ``

```
[uat-webservers]
<server1-ipaddress> ansible_ssh_user=<ec2-username> 
<server2-ip address> ansible_ssh_user=<ec2-username> 
                     
[lb]
<lb-instance-ip> ansible_ssh_user=<ec2-username> 

[db-servers]
<db-isntance-ip> ansible_ssh_user=<ec2-user>  

```
![image](https://github.com/user-attachments/assets/5e5b3b88-8a67-455d-bd67-2262276ab498)


## Step 5 : Configure your webserver roles to install php and all its dependencies , as well as cloning tooling website from  github repo.

- In the roles/webserver/tasks/main.yml , write the following tasks. use the code below :


```
- name: Install Apache
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
      name: "httpd"
      state: present
                   
- name: Install Git
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
     name: "git"
     state: present
                   
- name: Install EPEL release
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
     cmd: sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y
                   
- name: Install dnf-utils and Remi repository
   remote_user: ec2-user
   become: true
   become_user: root
   ansible.builtin.command:
     cmd: sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
                   
- name: Reset PHP module
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
     cmd: sudo dnf module reset php -y
                   
- name: Enable PHP 7.4 module
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
     cmd: sudo dnf module enable php:remi-7.4 -y
                   
- name: Install PHP and extensions
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
   name:
     - php
     - php-opcache
     - php-gd
     - php-curl
     - php-mysqlnd
   state: present
                   
- name: Install MySQL client
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
   name: "mysql"
   state: present
                   
- name: Start PHP-FPM service
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
    name: php-fpm
    state: started
                   
- name: Enable PHP-FPM service
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
   name: php-fpm
   enabled: true
                   
- name: Set SELinux boolean for httpd_execmem
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
   cmd: sudo setsebool -P httpd_execmem 1
                   
- name: Clone a repo
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.git:
   repo: https://github.com/citadelict/tooling.git
   dest: /var/www/html
   force: yes
                   
- name: Copy HTML content to one level up
  remote_user: ec2-user
  become: true
  become_user: root
   command: cp -r /var/www/html/html/ /var/www/
                   
- name: Start httpd service, if not started
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
   name: httpd
   state: started
                   
- name: Recursively remove /var/www/html/html directory
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.file:
   path: /var/www/html/html
   state: absent

```
![image](https://github.com/user-attachments/assets/509a204d-f55a-41ae-bca9-8c32e9dc7f8a)

- The code above tells ansible to install apache on the webservers , install git, install php and all its dependencies, clone the website from out github repo,as well ascopy the website files into the /var/www/html directory.
- Create a pull request and merge with your main branch of your github repo.
- Login to your ansible server via terminal and change directory into your ansible project, pull the recent changes done into your server

```
git pull origin main
```


Finally, run the playbook command against the inventory/uat files

```
ansible-playbook -i inventory/uat.yml playbooks/site.yml
```

![image](https://github.com/user-attachments/assets/b2e567f8-02ef-4161-9a41-b5c3469a1edf)

![image](https://github.com/user-attachments/assets/289bb462-8e32-4629-a970-08f47114b6c2)


Output:

![image](https://github.com/user-attachments/assets/210ce13e-bc0e-4a0d-816a-0df17ef7aa57)


### CONCLUSION:

The implementation of dynamic assignments in our Ansible project has significantly enhanced our configuration management and automation capabilities. By leveraging this approach, we have achieved:

1.Increased Flexibility: Dynamic assignments allow our playbooks to adapt to different environments and scenarios without requiring manual changes to the code.
2. Improved Scalability: As our infrastructure grows, the dynamic nature of our assignments enables easier management of larger and more complex environments.
3. Enhanced Reusability: By separating variables and logic from static playbooks, we've created more modular and reusable code, reducing duplication and improving maintainability.
4. Better Version Control: The use of separate files for variables and dynamic inclusions has improved our ability to track changes and manage different versions of our configurations.
5. Streamlined Workflows: Dynamic assignments have simplified our deployment processes, reducing the time and effort required to manage diverse environments.
6. Increased Efficiency: By dynamically including tasks and variables, we've optimized our Ansible runs, executing only the necessary tasks for each scenario.
7. Improved Error Handling: The implementation of dynamic assignments has allowed for more robust error checking and handling, increasing the reliability of our automation.
8. Enhanced Team Collaboration: The modular structure facilitated by dynamic assignments has made it easier for team members to work on different parts of the project simultaneously.

This project has demonstrated the power and flexibility of Ansible's dynamic assignments feature, providing a solid foundation for future automation efforts. As we continue to evolve our infrastructure, the lessons learned and techniques implemented here will prove invaluable in maintaining an efficient, scalable, and adaptable automation framework.
