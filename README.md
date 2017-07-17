# CakeSegvicky

Cake's Project

Prerequisites
 - OS: Ubuntu Server 14.04 64-bit

- App Server: Gunicorn/Nginx

- Python: 2.7						

- Ansible

- Docker 1.10
- Git
- Wordpress official Docker image https://hub.docker.com/_/wordpress/ 

- ElasticSearch

- Logstash

- Kibana						

This project creates a Docker composed based ELK setup, uses ansible for provisioning the minimal AWS infrastructure to run the ELK based logging infrastructure, then deploy dockerized WordPress based blogging application. 
Monitor the AWS resources and the blogging application using the ELK based centralized logging and monitoring infrastructure.

$ sudo apt-get install python-minimal -y

Steps:

$ git clone https://github.com/segvicky/cakesegvicky $ cd cakesegvicky

Configuring Hosts

Create and add the following lines at /etc/ansible/hosts file :

[local]
127.0.0.1

[ec2]
<EC2-IP> ansible_user=ubuntu
I chose the EC2 IP by choosing an elastic Ip for my instance.This is a great alternative to fixing IP as it may change after a reboot of EC2 instance. This setting can be observed at EC2 Dashboard > Elastic IP.

Moreso, I equally created another file at /etc/ansible/ansible.cfg.This is the main configuration file for ansible to run.

[defaults]

# some basic default values...

inventory      = /etc/ansible/hosts
library        = /usr/share/my_modules/
remote_tmp     = $HOME/.ansible/tmp
local_tmp      = $HOME/.ansible/tmp
forks          = 5
poll_interval  = 15
sudo_user      = root
#ask_sudo_pass = True
#ask_pass      = True
#transport      = smart
remote_port    = 22
#module_lang    = C
#module_set_locale = True

Creating EC2

This can be done using the provision.yml file present in the ansible dir.It requires one to put your AWS credentials there.The specifics.yml file stated the region,AMI and instance type.The command to run the ansible playbook is as follows:

$ sudo ansible-playbook provision.yml -i hosts -vv

I associated the Elastic IP with the EC2 instance.This can be observed under Elastic IP > Associate.

Configuring EC2

Configuring the EC2 instance by running some basic commands such as:

//Installing Docker
$ sudo apt-get clean && update -y
$ apt-get install apt-transport-https ca-certificates curl software-properties-common -y
$ apt-get install curl -y
$ sudo apt-get install docker.io -y
$ sudo usermod -aG docker ${USER}
$ sudo service docker restart

//Installing docker compose
$ sudo sudo apt-get install python-pip python-dev build-essential -y
$ sudo pip install docker-compose==1.3.0
This can be done using the ec2-configure.yml file present in the repo.The command would be:

$sudo ansible-playbook ec2-configure.yml -vv --private-key  <path-to-keypair>
Deploying ELK Stack using Docker Compose

It can be done using the ansible playbook.

$ sudo ansible-playbook elk-deploy.yml -vv --private-key <keypair>
The playbook consists of the below commands:

$ cp elk1/docker-compose.yml ~/home/ubuntu/
$ sudo docker-compose -f ~/home/ubuntu/docker-compose.yml up -d
$ sudo rm -f ~/home/ubuntu/docker-compose.yml 

Deploying Wordpress and mysql Containers using Docker Compose

It was done using the ansible playbook.

$ sudo ansible-playbook app-deploy.yml -vv --private-key <keypair>
The playbook consists of the following below commands:

$ cp app1/docker-compose.yml ~/home/ubuntu/
$ sudo docker-compose -f ~/home/ubuntu/docker-compose.app.yml up -d
$ sudo rm -f ~/home/ubuntu/docker-compose.app.yml 
Output

The output can be observed using the ip address of the ec2 instance.The Public DNS would be like ec2-xxx-xxx-xxx.compute.amazonaws.com.

Service | Port | Wordpress : 8080 Kibana : 5601

Please check the inbound and outbound rules in case of any page loading and reloading errors.You can check it out at EC2 Dashboard > Security Groups.