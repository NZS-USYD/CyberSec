## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![ELK Stack deployment diagram.](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/ELK-StackDeployment.png)

Fig. 1: ELK stack deployment in the Azure cloud. 

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the .yml files to install only certain pieces of it, such as Filebeat.

  - The ELK stack can be installed to a dedicated ELK server by simply running the `install-elk.yml` file.

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in use
  - Machines being monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the Damn Vulnerable Web Application. Load balancing ensures that the application will be highly efficient as it will distribute network traffic across multiple servers, in addition to restricting in-bound access to the network.
- The load balancer will be able to provide protection against DDoS attack by shifting attack traffic to public cloud server from the corporate server. 

- A jump server or jump host i.e., a special-purpose computer is used to configure DVWA web server and ELK server on the network. The JumpBoxProvisioner as shown in Fig. 1 is used to access and configure devices in two virtual networks. In this project the JumpBoxProvisioner, i.e. a Linux machine is used to configure and provision ELK server, DVWA servers via Ansible docker.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs.

- _Filebeat_: Filebeat consists of two main components: inputs and harvesters. A harvester is responsible for reading the content of a single file. The harvester reads each file, line by line, and sends the content to the output. One harvester is started for each file. An input on the other hand, is responsible for managing the harvesters and finding all sources to read from. In summary, Filebeat monitors the specified log files or locations, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- _Metricbeat_: Metricbeat consists of modules and metricsets. A Metricbeat module defines the basic logic for collecting data from a specific service, such as Redis, MySQL etc. The module specifies details about the service, including how to connect, how often to collect metrics, and which metrics to collect. Metricbeat retrieves metrics by periodically interrogating the host system based on the period value that is specified during the module configuration.

The configuration details of each machine can be found below from Table 1.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.
Table 1: Configuration details of the VMs deployed in the Azure environment. 

| Name               | Function                                           | IP Address                                   | Operating System     |
|--------------------|----------------------------------------------------|----------------------------------------------|----------------------|
| JumpBoxProvisioner | Provisioning, Access Management, Server Deployment | 20.124.203.24 (Public) 10.2.0.4/24 (Private) | Ubuntu 18.04 (Linux) |
| Web-1              | DVWA Container, Web server                         | 10.2.0.6/24 (Private)                        | Ubuntu 18.04 (Linux) |
| Web-2              | DVWA Container, Web server                         | 10.2.0.7/24 (Private)                        | Ubuntu 18.04 (Linux) |
| Web-3              | DVWA Container, Web Server                         | 10.2.0.8/24 (Private)                        | Ubuntu 18.04 (Linux) |
| ELK Server         | ELK Server for log collection  and monitoring      | 20.98.218.27 (Public) 10.3.0.4/24 (Private)  | Ubuntu 18.04 (Linux) |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the _JumpBoxProvisioner_ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _Administrator/Personal Machine: 115.129.151.129_

Machines within the network can only be accessed by Administrator/Personal Machine. In order to allow the Administrator/Personal Machine to access JumpBoxProvisioner via SSH (port 22) for the configuration and provisioning purposes an in-bound security rule is configured in the network security group named RedTeamNSG.
- _Similarly, Administrator/Personal machine (an IP address of 115.129.151.129) was allowed to access ELK Server (via its public IP 20.98.218.27) by configuring an in-bound security rule in the network security group called RedTeamELKNSG. The in-boud security rule was designed in a way that the Administrator/Personal machine can access ELK Server via port 5601._

A summary of the access policies in place can be found in the Table 2.

Table 2: Access policy of the deployed VMs

| Name               | Publicly accessible | Allowed IP Address                                                                   |
|--------------------|---------------------|--------------------------------------------------------------------------------------|
| JumpBoxProvisioner | Yes                 | 115.129.151.129                                                                      |
| Web-1              | No                  | 115.129.151.129 (via load balancer),  10.2.0.4/24 (Private IP of JumpBoxProvisioner) |
| Web-2              | No                  | 115.129.151.129 (via load balancer), 10.2.0.4/24 (Private IP of JumpBoxProvisioner)  |
| Web-3              | No                  | 115.129.151.129 (via load balancer),  10.2.0.4/24 (Private IP of JumpBoxProvisioner) |
| ELK Server         | Yes                 | 115.129.151.129                                                                      |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because of the following reasons:
- _Ansible allows an administrator to control remote machines via SSH._
- _Ansible can save time comparing with manual configuration._
- _Automation via Ansible can reduce bugs and errors._

The ELK stack implementation was carried out by Ansible. After creating a dedicated VM in Azure for ELK stack installation, it is accessed via SSH from the Ansible container (installed at JumpBoxProvisioner). In the next step, an Ansible playbook named `install-elk.yml` is created to finish the ELK stack implementation. The playbook performs the following tasks:

![hosts file configuration](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/hosts%20file%20configuration%20for%20Ansible%20to%20run%20playbook%20to%20dedicated%20ELK%20server.png)

Fig. 2: Configuring the `hosts` file to run playbook on the dedicated ELK VM. 

- _Step 1: In the first step as shown in Fig. 2, the `hosts` file in the `/etc/ansible/` directory was configured so that Ansible can discover ELK VM (RedTeamELK Server)._
- _A group called `[elk]` was added and the IP address of the ELK VM (10.3.0.4) was added._
- _While being in the Ansible container, `touch /etc/ansible/install-elk.yml` command was used to create YAML script. In the next step, a YAML script was written as shown in Fig. 3 by editing into the `install-elk.yml` playbook._
- _Finally, the playbook was run by using `ansible-playbook install-elk.yml` command to finish the ELK stack installation on the dedicated RedTeamELK Server, as shown in Fig. 4._ 

![elk-install.yml configration](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/YAML%20script%20for%20ELK%20stack%20configuration.png)

Fig. 3: YAML script for ELK stack configuration. 

![ELK stack installation via playbook](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/Installing%20ELK%20Stack.png)

FIg. 4: ELK stack installation via Ansible playbook.

Now `SSH` into the ELK server and issue `docker ps` from the ELK server's terminal to verify that the ELK instance is successfully running . 

![Validation: docker ps output to verify that ELK docker is running.](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/elk%20docker%20installation%20is%20successful.png)

Fig. 5: Validation: `docker ps` output to verify that ELK docker is running.

### Target Machines & Beats
This ELK server is configured to monitor the following machines given by Table 3.

Table 3: Machines detail monitored by ELK server.

| Name of the machine | IP address of the monitored machine |
|---------------------|-------------------------------------|
| Web-1               | 10.2.0.6/24                         |
| Web-2               | 10.2.0.7/24                         |
| Web-3               | 10.2.0.8/24                         |

The following Beats on these machines have been installed to collect logs:
- _Filebeat_
- _Metricbeat_

These Beats allow us to collect the following information from each machine:
- _Filebeat_ : FIlebeat collects log data and forwards the data to either Logstash or directly into Elasticsearch for indexing.

![Filebeat dashboard](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/Kibana%20DashBoard(FileBeat).PNG)

Fig. 6: Filebeat collecting syslogs.

Filebeat also collects `SSH` authentication logs, syslogs, web server data etc. Fig. 7 shows such an example where it collects accepted `SSH`logs:

![Filebeat collecting accepted SSH authentication log](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/Kibana%20DashBoard(FileBeat)%20collecting%20Accepted%20SSH%20information.PNG)

Fig. 7: Filebeat collecting `SSH` authentication logs.

Here is another example of Filebeat collecting system log.

![syslog collection](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/Kibana%20DashBoard(FileBeat)%20collecting%20syslog.PNG)

FIg. 8: Filebeat collecting syslog  

- _Metricbeat_: Metricbeat monitors servers by collecting metrics/statistics from the system and services running on the servers as shown in Fig. 9 and Fig. 10.

![Metricbeat collecting docker status](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/Kibana%20DashBoard(MetricBeat)%20Web-2.PNG)

Fig. 9: Metricbeat monitoring Web-2 

![Metricbeat dashborad](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/Kibana%20DashBoard(MetricBeat)%20Web-3.PNG)

Fig. 10: Metricbeat monitoring Web-3 

### Using the Playbook.
To use the playbook, an Ansible control node is configured in the JumpBoxProvisioner VM. 

SSH into the JumpBoxProvisioner and follow the steps below:
- `sudo su` to gain root privilege
- `docker container  list -a` to identify the Ansible controle node/docker
- `docker start exciting_moltancini` where exciting_montalcini is the Ansible docker 
- `docker attach exciting_montalcini` to access the Ansible docker for ELK server provisiong
- Now go to Ansible directory and configure the host file  as given in Fig .11.
![Ansible hosts file configuration](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/hosts%20file%20configuration%20for%20Ansible%20to%20run%20playbook%20to%20dedicated%20ELK%20server.png)

Fig. 11: Configure and update the hosts file to include paths for Web VMs and ELK Server. This will allow Ansible playbook to run ELK server and Filebeat & Metricbeat in the appropriate VM.

- _Note that `install-elk.yml`, `filebeat-playbook.yml` and `metricbeat-playbook.yml`_ are the Ansible playbooks, should be placed in the Ansible directory `/etc/ansible`.
- Run these playbooks by using `ansible-playbook install-elk.yml`, `ansible-playbook filebeat-playbook.yml` and `ansible-playbook metricbeat-playbook.yml` commands to install ELK stack, Filebeat and Metricbeat respectively. 

- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?In order to access the Kibana dashboard via ELK server use the following url:_ `http://ELK-VM's.public.IP:5601/` i.e. `http://20.98.218.27:5601/`
