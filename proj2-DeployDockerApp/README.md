# Deploy Docker App 

## Content

This contains instructions about how to:

- Setup Control Node 
- Copy Ansible Directory to Control Node
- Overview of files in Ansible Directory
- Running Playbook
- Demo

##  Setup Control Node 

- Create EC2 instance (linux ami) and name it Ansible-control-node -> SSH into control node -> Install python, pip, git and ansible

```bash
    ssh -i <path_to_pem_file_for_control_node> ec2-user@<public_ip_of_control_node>
```

```bash
    sudo yum install python
    curl -O https://bootstrap.pypa.io/get-pip.py
    python3 get-pip.py --user
    pip install ansible
    pip install git
```

- Create EC2 instances (managed hosts) and generate a key-pair for them, so that control node can ssh into hosts. Copy the key-pair to `/home/ec2-user/.ssh/` location on control node.

```bash
scp -i '<path_to_pem_file_for_control_node>'/Path/to/pem_file/for/node_servers ec2-user@<public_ip_of_control_node>:/home/ec2-user/.ssh/ansible-target.pem
```

## Copy Ansible Directory to Control Node

- ssh into control node and clone the git repo and run the playbook

```bash
    git clone https://github.com/Anil5-0/Ansible-Projects.git
    cd Ansible-Projects/proj2-DeployDockerApp
```

## Overview of files in Ansible Directory

- `ansible.cfg` is used to override default cfg and set `host_key_checking=False` under `[defaults]` section to avoid yes/no prompt and set `inventory=./hosts` so that we don't need to pass `-i hosts` in ansible-playbook cmd.

- `aws_vars.yaml` contains variables for linux user name, user group, home dir and pwd for docker to login to ecr. Tell ansible to use vars in aws_vars.yaml using `vars_files` at play. These variables are referred inside tasks in play as "{{ var_name }}"

- `docker-compose.yaml` is a Docker Compose file written in YAML format. Docker Compose is a tool for defining and running multi-container Docker applications. This file is used to define a service called app that runs a Docker image named `482114220585.dkr.ecr.ap-southeast-1.amazonaws.com/python-hello-world` with the tag `0.1.1`

- `hosts` file contains a group called `[docker_servers]` which contains all public ip's of target nodes. define variables specific to that group under `[docker_servers:vars]`.

- `playbook.yaml` contains a list of plays which gets executed on servers under docker_servers section -> Install docker and pip packages using yum using root -> Install Docker-Compose using `get_url` module -> Create a new user to run our app instead of root -> done using user module and needs privilege_escalation so `become: yes` -> Start docker using `systemd` module -> Copy the Docker compose yaml file to target server using `copy` module -> Inorder to make the docker client pull images from private repo, need to do docker login. That's done using `docker_login` module -> Run the app using `docker-compose` module.

## Running Playbook

- Run the playbook using command:
```bash
    ansible-playbook playbook.yaml
```

- uname: command line utility that print basic info about os name & system hardware.
```bash
    uname -s # Linux
    uname -m # x86_64
```
- pipe lookup plugin: Allow ansible to fetch data from outside sources. Calculates output of shell command & pipes it to left side of lookup. Lookup is part of jinja2 template that Ansible uses.

- systemd module: control systemd services on remote host

- Need to add current user to docker grp, in order for this user to run docker cmds.

- cmd & shell modules doesn't offer state management or idempotency. So we'll use Docker community collection.

- Fully Qualified Collection Names :
```bash
    <namespace>.<collection>.<module> 
    <namespace>.<collection>.<plugin> # community.docker.docker_image
```
- `vars_prompt` is another way to pass variables to playbook. It gives a prompt at runtime, so we can enter the login-password for ecr. It's defined at level similar to `vars` at a play.


## Demo

- SSH into control node, clone the git repo and run ansible playbook.

    ![](/docs/images/proj2/01-ansible-output.PNG)

- Connect to cloudshell and check the logs of docker container

    ![](/docs/images/proj2/02-container-logs.PNG)
