## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.
 

![ELK Complete Diagram](Images/elk-final-diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the ansible playbook files may be used to install only certain pieces of it, such as Filebeat.

  - _[PenTest Playbook](Ansible/pentest.yml)_
  ```yaml
  ---
  - name: Config Web VM with Docker
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
        name: python3-pip
        state: present

    - name: Install Python Docker Module
      pip:
        name: docker
        state: present

    - name: download and launch a docker web container
      docker_container:
        name: dvw
        image: cyberxsecurity/dvwa
        state: started
        restart_policy: always
        published_ports: 80:80

    - name: enable docker at startup
      service:
        name: docker
        enabled: true

  ```
  - _[Elk Playbook](Ansible/elk.yml)_
  ```yaml
  ---
  - name: Config ELK VM with Docker
    hosts: elkservers
    become: true
    tasks:
    # Usee apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

    # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    # Use pip module (It will default to pip3)
    - name: Install Python Docker Module
      pip:
        name: docker
        state: present

    # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: '262144'
        state: present
        reload: yes

    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

    - name: enable docker at startup
      service:
        name: docker
        enabled: true
  ```
  - _[FileBeat Playbook](Ansible/filebeat-playbook.yml)_
  ```yaml
  ---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

    # Use command moudle
  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
    
    # Use command moudle
  - name: install filebeat deb
    command: dpkg -i filebeat-7.6.1-amd64.deb

    # Use copy moudle
  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command moudle
  - name: enable and configure system module
    command: filebeat modules enable system 

    # Use command moudle
  - name: setup filebeat
    command: filebeat setup

    # Use command moudle
  - name: start filebeat service
    command: service filebeat start

    # Use systemd module
  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
  ```
  - _[MetricBeat Playbook](Ansible/metricbeat-playbook.yml)_
  ```yaml
  ---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command moudle
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.6.1-amd64.deb

    # Use copy moudle
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module 
  - name: enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes 

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

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.

- _The Load Balancer distributes incoming internet requests to all three Web Server instances. DDoS Protection Standard is enabled on the virtual network of the Azure (internet) load balancer that has the public IP associated with it.  Jumphosts aks Bastion servers are part of the "security by obsecurity" approach, they are usually part of the infrastructure but outside of the assumed attach vector. In our example of using Web Servers, Web Servers would only accept requests via a list of specific servers, Web Servers are easily found on the internet as they expose ports and host services, like a website. The Jumpbox's only purpose is to provide access to those Web Servers, and does NOT offer any public services._

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the network and system logs & metrics.

- _Filebeat is a lightweight shipper for forwarding and centralizing log data. Installed as an agent on your servers, Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to ELK server for indexing._

- _Metricbeat is a lightweight shipper that you can install on your servers to periodically collect metrics from the operating system and from services/docker running on the Web Servers. Metricbeat takes the metrics and statistics that it collects and ships them to the ELK server._

The configuration details of each machine may be found below.

| Name     | Function   | IP Address               | Operating System |
|----------|------------|--------------------------|------------------|
| Jump Box | Gateway    | 10.0.0.4/40.121.163.132  | Linux            |
| Web-1    | Web Server | 10.0.0.5                 | Linux            |
| Web-2    | Web Server | 10.0.0.6                 | Linux            |
| Web-3    | Web Server | 10.0.0.10                | Linux            |
| ELKVM    | ELK Server | 10.1.0.7/52.151.46.191   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _99.244.XXX.XXX/32 - Personal Home IP Address_

Machines within the network can only be accessed by SSH protocol.
- _TODO: Which machine did you allow to access your ELK VM? What was its IP address?_
- _ELK VM can only be access from the Jump Box using SSH from the IP address 10.0.0.4 (Jump Box).  And ELK dashboard (via Port 5601) can only be access via a public IP address of the ELK VM from the Personal Home IP Address?_

A summary of the access policies in place can be found in the table below.

| Name          | Port | Publicly Accessible   | Allowed IP Address             |
|---------------|------|-----------------------|--------------------------------|
| Jump Box      | 22   | Yes                   | *Personal IP*                  |
| Web-1         | 22   | No                    | 10.0.0.4 - Jump Box            |
| Web-1         | 80   | Yes vai Load Balancer | 40.121.151.226 - Load Balancer |
| Web-2         | 22   | No                    | 10.0.0.4 - Jump Box            |
| Web-2         | 80   | Yes vai Load Balancer | 40.121.151.226 - Load Balancer |
| Web-3         | 22   | No                    | 10.0.0.4 - Jump Box            |
| Web-3         | 80   | Yes vai Load Balancer | 40.121.151.226 - Load Balancer |
| ELKVM         | 22   | No                    | 10.0.0.4 - Jump Box            |
| ELKVM         | 5601 | Yes                   | *Personal IP*                  |
| Load Balancer |      | Yes                   | Open                           |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _it allows us make deploying entire application environments easy, predictable, and repeatable. and also updates multiple identifical servers with a single playbook yaml file quickly without having to go into each servers. Instead of relying on a software agent on each remote managed host, Ansible relies on the trusted management ports already in use by IT teams to manage servers and infrastructure: secure shell (SSH) on Linux, and Windows Remote Management (WinRM) on Microsoft -based systems._

The playbook implements the following tasks:
- _Installs docker.io, python3-pip
- _Starts docker
- _Pulls the cyberxsecurity/dvwa docker image and enable docker on startup_
- _Increases the virtual memory for the ELK server using sysctl module_
- _Downloads the sebp/elk:761 docker image and launches the ELK container_
- _Installs and launches filebeat & metricbeat on the ELK VM_

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![ELK VM screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats

This ELK server is configured to monitor the following machines:

| Name     | Function   | IP Address | Operating System |
|----------|------------|------------|------------------|
| Web-1    | Web Server | 10.0.0.5   | Linux            |
| Web-2    | Web Server | 10.0.0.6   | Linux            |
| Web-3    | Web Server | 10.0.0.10  | Linux            |

We have installed the following Beats on these machines:
- _Filebeat_
- _Metricbeat_

These Beats allow us to collect the following information from each machine:
- _Filebeat is a log data shipper for local files. Installed as an agent on your servers, Filebeat monitors the log directories or specific log files, tails the files, and forwards them either to Elasticsearch or Logstash for indexing._
- _Metricbeats module fetches metrics from Docker containers. The default metricsets are: container, cpu, diskio, healthcheck, info, memory and network. The image metricset is not enabled by default._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the ansible playbook files (pentest.yml, elk.yml, filebeat-playbook.yml, metricbeat-playbook.yml) to /etc/ansible/roles folder.
- Copy the metricbeat-config.yml and filebeat-config.yml files to /etc/ansible/files folder
- Update the /etc/ansible/hosts file to include the webservers and elk server as follows:

```ini
[webservers]
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3
10.0.0.10 ansible_python_interpreter=/usr/bin/python3

[elkservers]
10.1.0.7 ansible_python_interpreter=/usr/bin/python3
```
- Update the /etc/ansible/ansible.cfg to have the remote_user as follows:

```ini
remote_user = sysadmin
```

- Update the /etc/ansible/files/filebeat-config.yml to point to the Elasticsearch as follows:

```ini
1104 output.elasticsearch:
1105 hosts: ["10.1.0.7:9200"] 
1106 username: "elastic"
1107 password: "changeme"
```

- Update the /etc/ansible/files/metricbeat-config.yml with the Kibana endpoint configuration as follows:

```ini
1804 setup.kibana:
1805 host: "10.1.0.7:5601"
```

- Update the /etc/ansible/files/metricbeat-config.yml to point to the Elasticsearch as follows:

```ini
93 output.elasticsearch:
94 hosts: ["10.1.0.7:9200"] 
95 username: "elastic"
97 password: "changeme"
```

- Update the /etc/ansible/files/filebeat-config.yml with the Kibana endpoint configuration as follows:

```ini
61 setup.kibana:
62 host: "10.1.0.7:5601"
```
- Run the following ansible command to make sure ansible can ping all the servers in the /etc/ansible/hosts file:

```bash
ansible -m ping all

output:

10.1.0.7 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
10.0.0.6 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
10.0.0.5 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
10.0.0.10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
- Run the `elk.yml` playbook using the following command:
```bash
ansible-playbook elk.yml
```
- Navigate to http://52.151.46.191:5601/app/kibana to check that the installation worked as expected.

![Kibana Dashboard](Images/kibana_dashboard.png)

- Run the `/etc/ansible/roles/elk.yml` playbook using the following command:
```bash
ansible-playbook elk.yml
```
- Navigate to http://52.151.46.191:5601/app/kibana to check that the installation worked as expected.

![Kibana Dashboard](Images/kibana_dashboard.png)

- Run the `/etc/ansible/roles/filebeat-playbook.yml` playbook using the following command:
```bash
ansible-playbook filebeat-playbook.yml
```
![Filebeat playbook](Images/filebeat-playbook.png)

- Navigate to dashboard and check that the installation worked as exected:
![Filebeat output](Images/filebeat-playbook-output.png)



_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._