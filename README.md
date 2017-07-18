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

DASHBOARD CREATION:USING DATADOG 
To collect Docker metrics about all your containers, one needs to run one Datadog Agent on every host. There are two ways to run the Agent: directly on each host, or within a docker-dd-agent container. I shall be using using the latter

The hosts need to have cgroup memory management enabled for the Docker check to succeed.

Container Installation
Ensure Docker is running on the host.
docker run -d --name dd-agent \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /proc/:/host/proc/:ro \
  -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
  -e API_KEY={YOUR API KEY} \
  datadog/docker-dd-agent:latest
  
  Environment variables

Note that in the command above, you are able to pass your API key to the Datadog Agent using Docker’s -e environment variable flag. Some other variables you can pass include:API-KEY which sets the Datadog API key, DD_HOSTNAME sets the hostname in the Agent container's datadog.conf file, etc.
  
  METRICS
  docker.cpu.system: displays the fraction of time the CPU is executing system calls on behalf of processes of the container
docker.containers.running: shows the number of containers running on the host
docker.containers.stopped: The number of containers running on this host  and many more.
  
  CONFIGURATION FILES

One can also mount YAML configuration files in the /conf.d folder, they will automatically be copied to /etc/dd-agent/conf.d/ when the container starts. The same can be done for the /checks.d folder. Any Python files in the /checks.d folder will automatically be copied to /etc/dd-agent/checks.d/ when the container starts.

Create a configuration folder on the host and write the YAML files in it. The examples below uses the /checks.d folder

mkdir /opt/dd-agent-conf.d
touch /opt/dd-agent-conf.d/nginx.yaml
When creating the container, mount this new folder to /conf.d.

docker run -d --name dd-agent \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /proc/:/host/proc/:ro \
  -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
  -v /opt/dd-agent-conf.d:/conf.d:ro \
  -e API_KEY={your_api_key_here} \
  datadog/docker-dd-agent
The important part here is -v /opt/dd-agent-conf.d:/conf.d:ro

Now when the container starts, all files in /opt/dd-agent-conf.d with a .yaml extension will be copied to /etc/dd-agent/conf.d/. Please note that to add new files you will need to restart the container.

  BUILDING AN IMAGE:
To configure specific settings of the agent directly in the image, I need to build a Docker image on top of it.

Creating a Dockerfile to set the specific configuration or to install dependencies.

FROM datadog/docker-dd-agent
# Example: MySQL
ADD conf.d/mysql.yaml /etc/dd-agent/conf.d/mysql.yaml
Build it.

docker build -t dd-agent-image .

Then run it like the datadog/docker-dd-agent image.

docker run -d --name dd-agent \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /proc/:/host/proc/:ro \
  -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
  -e API_KEY={your_api_key_here} \
  dd-agent-image

  
