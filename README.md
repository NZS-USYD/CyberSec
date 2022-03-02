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

The configuration details of each machine can be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

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

A summary of the access policies in place can be found in the table below.

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
- _While being in the Ansible container, `touch /etc/ansible/install-elk.yml` command was used to create YAML script. In the next step, a YAML script was written as shown in Fig. 3 in the `install-elk.yml` file._
- _ Finally, the install-elk.yml file was run to finish the ELK stack installation on the dedicated RedTeamELK Server._ 

![elk-install.yml configration](https://github.com/NZS-USYD/CyberSec/blob/main/Diagrams/YAML%20script%20for%20ELK%20stack%20configuration.png)

Fig. 3: YAML script for ELK stack configuration.  

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_

We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the _____ file to _____.
- Update the _____ file to include...
- Run the playbook, and navigate to ____ to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
