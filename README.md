
# Ansible Role to setup PostgreSQL

![image](https://github.com/avengers-p7/Documentation/assets/79625874/08f84d0b-467a-46f1-b6de-8603020cf116)



|   Authors        |  Created on   |  Version   | Last updated by | Last edited on |
| -----------------| --------------| -----------|---------------- | -------------- |
| Vikram Bisht     |  10 Feb 2024  |     v1     | Vikram Bisht    | 13 Feb 2024    |

***
# Table of Contents
+ [Introduction](#Introduction)
+ [Pre-requisites](#Pre-requisites)
+ [Port-Numbers](#Port-Numbers)
+ [Flow Diagram](#Flow-Diagram)
+ [Steps](#Steps)
+ [Output Verification](#output-Verification)
+ [Conclusion](#conclusion)
+ [Contact Information](#contact-information)
+ [References](#references)

***

# Introduction
This role is designed to automate the installation and configuration of PostgreSQL on target ubuntu servers. By using this role you can acheive that.
***

# Pre-requisites

|  Pre-requisites	                     |        	Description         |
| ------------              | --------------------------------|
| Ansible                   |  Ansible must be installed on the control machine from which you plan to run the playbook. If Ansible is not installed, you can install it using this [link](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)       |
| SSH Access to Target Servers                   | Ensure that you have SSH access to the target servers where PostgreSQL will be installed.  |
| Ansible                   | Here we used version 2.15. |
| AWS                       | AWS account for instance creation  |

# Port Numbers

|   Port Number    |  Purpose      |  
| -----------------| --------------| 
| 22               | For SSH       |
| 5432             | PostdreSQL    |

# Flow Diagram

* This diagram should help you visualize the sequence of tasks in your Ansible playbook for installing Jenkins.

![image](https://github.com/avengers-p7/Documentation/assets/79625874/6fa6a96a-f15b-43b1-9c29-17b0d1491587)


# Steps 
* Before going further check the link for [Ansible Role](https://github.com/vishalkk2805/my_Tool/tree/main/Postgresql_tool)

**Step 1: Dynamic Inventory Setup** 

```yaml
 [defaults]
# some basic default values...
inventory               =       /home/ubuntu/Postgresql_tool/aws_ec2.yaml
private_key_file        =       /home/ubuntu/Downloads/PostgreSQL.pem
remote_user             =       ubuntu
host_key_checking       =       False
[inventory]
enable_plugins          =       aws_ec2
```

> [!NOTE]
>Ensure that for dynamic inventory you have the necessary AWS credentials configured in AWS CLI and attach an IAM role on the master node.

**Step 2:  AWS EC2 Inventory**

```yaml
---
plugin: aws_ec2
regions:
  - ap-northeast-1

groups: 
  ubuntu: "'ubuntu' in tags.OS"
```

1. `plugin: aws_ec2`: Specifies the use of the aws_ec2 plugin as the dynamic inventory source. This plugin is designed to fetch information about EC2 instances in AWS.
2. `regions:   - ap-northeast-1`: Indicates the AWS region(s) from which the dynamic inventory should fetch information.
3. `ubuntu: "'ubuntu' in tags.OS"`: Creates an Ansible group named ubuntu. This group includes EC2 instances where the tag named OS has a value of 'ubuntu'.

**Step 3: Create Ansible Role**

To create an Ansible role, which should follow below directory structure, you can use the ansible-galaxy command-line tool. Here's the command to create a new Ansible role:

```bash
ansible-galaxy init <role_name>
```

  
![image](https://github.com/avengers-p7/Documentation/assets/79625874/53ce10fa-148e-49ad-8b90-51dc109a896b)

**Step 4: playbook.yml**
* This file is defining a set of tasks to be executed on hosts belonging to the ubuntu group.

```yaml
---
- hosts: ubuntu
  become: yes
  gather_facts: yes
  roles:
    - Postgresql_role
```
**Step 5: Tasks**
1. `main.yml`: This main.yml playbook is designed to handle tasks and variables differently based on the operating system family detected by Ansible, providing a way to manage configurations and installations across different types of systems efficiently.

```yaml
---
# Inclueds File 
- name: Include OS-Specific variables
  include_vars: "{{ ansible_os_family }}.yml"

- include_tasks: install_Redhat.yml
  when: ansible_os_family == 'RedHat'

- include_tasks: install_Debian.yml
  when: ansible_os_family == 'Debian'
```
2. `install_Debian.yml`: This file is included in the PostgreSQL_role/tasks/Debian.yml file

```yaml
---
- name: Install Gpg
  apt:
    name: gnupg
    state: present

- name: Add Postgresql repository key
  apt_key:
    url: "{{ postgresql_repo_key_url }}"
    state: present

- name: Add Postgresql repository.
  apt_repository:
    repo: "{{ postgresql_repo_url }}"
    state: present

- name: Install PostgreSQL on Ubuntu
  apt:
    name: "{{ item }}"
    update_cache: true
    state: present
  loop:
    - '{{ debian_package }}'

- name: Start postgresql
  service:
    name: postgresql
    state: started
    enabled: true

- name: Congfigure Postgresql
  template:
    src: ubuntu_postgresql.j2
    dest: /etc/postgresql/{{ version }}/main/postgresql.conf

- name: Restart postgresql
  service:
    name: postgresql
    state: restarted
```

**Step 6: Playbook Execution**

* To set up PostgreSQL on your target servers, you will execute the Ansible playbook using the following command:

```bash
ansible-playbook /home/ubuntu/Postgresql_tool/Postgresql.yml
```
***
# Output

After successful execution of PostgreSQL playbook we will able to see below output:

![image](https://github.com/avengers-p7/Documentation/assets/79625874/6233e8a3-1b30-4049-99ba-67f632adde1c)
![image](https://github.com/avengers-p7/Documentation/assets/79625874/35f90fef-4bc3-45ec-aab8-a40774bd32b4)

In above screenshot we can see our PostgreSQL service is active and running.

# Conclusion 

This guide illustrates the process of deploying PostgreSQL in a server through Ansible. By adhering to these instructions, you can effectively provision and set up PostgreSQL within your AWS infrastructure.


# Contact Information

|  Name                     |        	Email Address         |
| ------------              | --------------------------------|
| Vikram Bisht              |  Vikram.Bisht@opstree.com       |  

# References

| Title                                      | URL                                           |
|--------------------------------------------|-----------------------------------------------|
| Ansible documentation           | https://docs.ansible.com/ansible/latest/index.html    |
| PostgreSQL Document             |  [Link](https://github.com/avengers-p7/Documentation/blob/main/OT%20Micro%20Services/Software/PostgresSQL/README.md) |
| Ansible Dynamic Inventory       | https://www.youtube.com/watch?v=junPdh2yvbU&t=454s | 
