## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Diagram of ELK solution:](Images/diagram_filename.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the *all_playbooks.yml* file may be used to install only certain pieces of it, such as Filebeat.

  - _Enter the playbook file._
    - Answer:
[all_playbooks.yml](https://raw.githubusercontent.com/aenagy/UTOR-VIRT-CYBER-PT-06-2021-U-LOL/main/all_playbooks.yml)


```yaml
---
  - name: Class 12.3 Activity 3
    hosts: webservers
    become: true
    tasks:
    - name: docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: Install Python Docker module
      pip:
        name: docker
        state: present

    - name: Download and launch a docker web container
      docker_container:
        name: dvwa
        image: cyberxsecurity/dvwa
        state: started
        restart_policy: always
        published_ports: 80:80

    - name: Enable docker service
      systemd:
        name: docker
        enabled: yes

- name: Class 13.1
  hosts: elk
  become: true
  tasks:
  - name: set vm.max_map_count to 262144 in sysctl
    sysctl: name={{ item.key }} value={{ item.value }}
    with_items:
      - { key: "vm.max_map_count", value: "262144" }

  - name: Install docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install PIP 3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker Python module
    pip:
      name: docker
      state: present

  - name: download and launch docker elk container
    docker_container:
     name: elk
     image: sebp/elk:761
     state: started
     published_ports:
       - 5601:5601
       - 9200:9200
       - 5044:5044

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes
      state: restarted

- name: Setup filebeats
  hosts: webservers
  remote_user: ansibleadmin

  tasks:
  - name: Create /home/ansibleadmin/files folder on managed nodes
    file:
      path: /home/ansibleadmin/files
      state: directory

  - name: Copy deb file from Ansible Server to managed node
    ansible.builtin.copy:
      src: /etc/ansible/files/filebeat-7.4.0-amd64.deb
      dest: /home/ansibleadmin/filebeat-7.4.0-amd64.deb
      owner: ansibleadmin
      group: ansibleadmin
      mode: u=rw,g=r,o=r
      remote_src: no

  - name: Copy deb file from Ansible Server to managed node
    ansible.builtin.copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /home/ansibleadmin/filebeat.yml
      owner: ansibleadmin
      group: ansibleadmin
      mode: u=rw,g=r,o=r
      remote_src: no

  - name: copy /home/ansibleadmin/filebeat.yml to /etc/filebeat/filebeat.yml
    command: sudo cp /home/ansibleadmin/filebeat.yml /etc/filebeat/filebeat.yml

  - name: set owner of /etc/filebeat/filebeat.yml
    command: sudo chown root:root /etc/filebeat/filebeat.yml

  - name: sudo dpkg -i filebeat-7.4.0-amd64.deb
    command: sudo dpkg -i filebeat-7.4.0-amd64.deb

  - name: sudo filebeat modules enable system
    command: sudo filebeat modules enable system

  - name: sudo filebeat setup
    command: sudo filebeat setup

  - name: sudo service filebeat start
    command: sudo service filebeat start

  - name: Enable filebeat
    become: yes
    become_method: sudo
    service:
      name: filebeat
      enabled: yes
      state: started
      use: service

- name: Setup metricbeats
  hosts: webservers
  remote_user: ansibleadmin

  tasks:
  - name: Create /home/ansibleadmin/files folder on managed nodes
    file:
      path: /home/ansibleadmin/files
      state: directory

  - name: Create /etc/metricbeat
    become: yes
    file:
      path: /etc/metricbeat
      state: directory
      owner: root

  - name: Copy metricbeat-7.4.0-amd64.deb metric from Ansible Server to managed node
    ansible.builtin.copy:
      src: /etc/ansible/files/metricbeat-7.4.0-amd64.deb
      dest: /home/ansibleadmin/files/metricbeat-7.4.0-amd64.deb
      owner: ansibleadmin
      group: ansibleadmin
      mode: u=rw,g=r,o=r
      remote_src: no

  - name: Copy metricbeat-config.yml from Ansible Server to managed node
    ansible.builtin.copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /home/ansibleadmin/files/metricbeat.yml
      owner: ansibleadmin
      group: ansibleadmin
      mode: u=rw,g=r,o=r
      remote_src: no

  - name: copy /home/ansibleadmin/files/metricbeat.yml to /etc/metricbeat/metricbeat.yml
    command: sudo cp -p /home/ansibleadmin/files/metricbeat.yml /etc/metricbeat/metricbeat.yml

  - name: set owner of /etc/metricbeat/metricbeat.yml
    command: sudo chown root:root /etc/metricbeat/metricbeat.yml

  - name: fix
    command: sudo rm -f /var/lib/dpkg/lock

  - name: install
    command: sudo apt-get -o DPkg::Options::="--force-confnew" -y install /home/ansibleadmin/metricbeat-7.4.0-amd64.deb

  - name: sudo metricbeat modules enable system
    command: sudo metricbeat modules enable system

  - name: sudo metricbeat setup
    command: sudo metricbeat setup

  - name: sudo service metricbeat start
    command: sudo service metricbeat start

  - name: Enable metricbeat
    become: yes
    become_method: sudo
    service:
      name: metricbeat
      enabled: yes
      state: started
      use: service
```


This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly *available*, in addition to restricting *access* to the network.
- _What aspect of security do load balancers protect?_
  - Answer: Availability

- _What is the advantage of a jump box?_
  - Answer: Reduced attack surface due to only one entry point for management.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the *files* and system *metrics*.

- _What does Filebeat watch for?_
  - Answer:
    - Filebeat watches for changes to files, typically text files.
    - "Filebeat collects data about the file system." https://utoronto.bootcampcontent.com/utoronto-bootcamp/utor-virt-cyber-pt-06-2021-u-lol/-/blob/master/13-Elk-Stack-Project/StudentGuide.md

- _What does Metricbeat record?_
  - Answer:
    - Metricbeat records ativity within the operating system or hardware such as processor or memory or storage utilization.
    - "Metricbeat collects machine metrics, such as uptime. A metric is simply a measurement about an aspect of a system that tells analysts how "healthy" it is." https://utoronto.bootcampcontent.com/utoronto-bootcamp/utor-virt-cyber-pt-06-2021-u-lol/-/blob/master/13-Elk-Stack-Project/StudentGuide.md


The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function                                   | IP Address | Operating System     |
|----------|--------------------------------------------|------------|----------------------|
| jump-box | Managementy gateway                        | 10.0.0.4   | Linux (ubuntu 20.04) |
| web1     | DVWA web server                            | 10.0.0.7   | Linux (ubuntu 20.04  |
| web2     | DVWA web server                            | 10.0.0.8   | Linux (ubuntu 20.04) |
| web3     | DVWA web server                            | 10.0.0.9   | Linux (ubuntu 20.04) |
| ELK1     | ELK (ElasticSearch Logstash Kibana) server | 10.1.0.4   | Linux (ubuntu 20.04) |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the *jump-box* machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _Add whitelisted IP addresses_
  - Answer: You are probably expecting an IPv4 address here. Instead, for security reasons, all I will say is that the output from `what is my ipv4 address` in Chrome running on my home PC will be used here.

Machines within the network can only be accessed by *jump-box*.
- _Which machine did you allow to access your ELK VM?_
  - Answer: jump-box via ssh and  output from `what is my ipv4 address`

- _What was its IP address?_
  - Answer: 10.0.0.4

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 |  output from `what is my ipv4 address` |
| web1     | No                  |  10.0.0.7 |
| web2     | No                  |  10.0.0.8 |
| web3     | No                  |  10.0.0.9 |
| ELK1     | Yes                 | 10.0.0.4, output from `what is my ipv4 address` |


### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _What is the main advantage of automating configuration with Ansible?_
  - Answer: Consistency and Repeatability.


The playbook implements the following tasks:
- _In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
  - Answer: from 'Class 12.3 Activity 3' in [all_playbooks.yml](https://raw.githubusercontent.com/aenagy/UTOR-VIRT-CYBER-PT-06-2021-U-LOL/main/all_playbooks.yml)
    - Set vm.max_map_count to 256 MB
    - Install docker
    - Install Python
    - Download and launch ELK container
    - Enable docker service

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![Output from 'docker ps':](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _List the IP addresses of the machines you are monitoring_
  - Answer: 10.0.0.7, 10.0.0.8, 10.0.0.9

We have installed the following Beats on these machines:
- _Specify which Beats you successfully installed_
  - Answer: filebeat and metricbeat

These Beats allow us to collect the following information from each machine:
- _In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._
  - Answer: "Beats is a free and open platform for single-purpose data shippers. They send data from hundreds or thousands of machines and systems to Logstash or Elasticsearch." https://www.elastic.co/beats/
    - filebeat: "Filebeat helps you keep the simple things simple by offering a lightweight way to forward and centralize logs and files." https://www.elastic.co/beats/filebeat
    - metricbeat: "Metricbeat is a lightweight way to send system and service statistics." https://www.elastic.co/beats/metricbeat


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- _Copy the *id_rsa.pub* file to *each managed node, i.e. web server (web1, web2 and web3)*._
- _Update the */etc/ansible/ansible.cfg* file to include..._
  - Answer:
    - `remote_user=ansibleadmin` # uncomment and edit this line, same user associated with *id_rsa.pub* above.

- _Update the */etc/ansible/hosts* file to include..._
  - Answer:
```
[webservers] # uncomment this line
10.0.0.7 ansible_python_interpreter=/usr/bin/python3
10.0.0.8 ansible_python_interpreter=/usr/bin/python3
10.0.0.9 ansible_python_interpreter=/usr/bin/python3
```

- Run the playbook, and navigate to *each managed node, i.e. web server (web1, web2 and web3)* to check that the installation worked as expected.

_Answer the following questions to fill in the blanks:_
- _Which file is the playbook?_
  - Answer:
    - [all_playbooks.yml](https://raw.githubusercontent.com/aenagy/UTOR-VIRT-CYBER-PT-06-2021-U-LOL/main/all_playbooks.yml)

- _Where do you copy it?_
  - Answer:
    - /etc/ansible/playbooks

- _Which file do you update to make Ansible run the playbook on a specific machine?_
  - Answer:
    - /etc/ansible/hosts

- _How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
  - Answer:
    1. Edit /etc/ansible/hosts
    2. Add a group name in square brackets and enter each node hostname or IP on each following line like so:

```yaml
[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
```

    3. In the Ansible Playbook YAML file include the `hosts` keyword like so:

```yaml
  hosts: elk
```

- _Which URL do you navigate to in order to check that the ELK server is running?_
  - Answer:
    - `http://<IP address of ELK1 server>:5601/app/kibana`


_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
  - Answer:
    - Run: `ansible-playbook /etc/ansible/pentest.yml`
    - Download consolidated Playbook: `cd /etc/ansible && curl https://raw.githubusercontent.com/aenagy/UTOR-VIRT-CYBER-PT-06-2021-U-LOL/main/all_playbooks.yml`
    - Update consolidated Playbook file: `vi /etc/ansible/all_playbooks.yml`
    - Create consolidated Playbook file: `find /etc/ansible/playbooks/ /etc/ansible/roles/ -iname *.yml -exec cat {}>> all_playbooks.txt \;`
