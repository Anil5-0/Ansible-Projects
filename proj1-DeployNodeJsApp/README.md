# Deploy Node.js App 

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
    cd Ansible-Projects/proj1-DeployNodeJsApp
```

## Overview of files in Ansible Directory

- `app.js` is a simple Node.js app which runs http server on port 3000 and when a request is sent, it returns a `Hello, Node.js!` msg

- `ansible.cfg` is used to override default cfg and set `host_key_checking=False` under `[defaults]` section to avoid yes/no prompt

- `hosts` file contains a group called `[node_servers]` which contains all public ip's of target nodes. define variables specific to that group under `[node_servers:vars]`.

- `node_vars.yaml` contains variables for linux user name and home dir. Tell ansible to use vars in node_vars.yaml using `vars_files` at play. These variables are referred inside tasks in play as "{{ var_name }}"

- `playbook.yaml` contains a list of plays which gets executed on servers under node_servers section -> Create a new user to run our app instead of root -> done using user module and needs privilege_escalation so `become: yes` -> Install node and npm packages using yum using root -> Create a folder `my-node-app/src` which holds `app.js` file using file module -> Copy `app.js` from control-node to node_servers using copy module -> Run the app using command module `node app.js` -> Ensure app is running using shell module and store the result of task into variable using `register` -> Print the output of variable using debug module.

## Running Playbook

- Run the playbook using command:
```bash
    ansible-playbook -i hosts playbook.yaml
```

- `node app.js` will block the terminal and playbook never exits. So we use async, poll attributes

- command module is more secure than shell module. Use shell module for pipe(|), boolean (&&,||), env_vars, $HOME, etc. These modules are not idempotent so may need to use conditionals.

- Can also pass variables to ansible-playbook cmd or by defining `vars` inside a play. Can't use hyphen in variable name, only underscore.

```bash
    ansible-playbook -i hosts playbook.yaml -e "linux_name=node-app home_dir=/home/{{linux_name}}"
```

## Demo

- Create Control Node and Servers on AWS EC2.

    ![](/docs/images/proj1/01-servers.PNG)

- SSH into control node, clone the git repo and run ansible playbook.

    ![](/docs/images/proj1/02-clone-repo-to-control-node.PNG)

    ![](/docs/images/proj1/03-playbook-op1.PNG)

    ![](/docs/images/proj1/04-playbook-op-2.PNG)

- send http request to the remote server at port 3000 and check if app is accessible.

    ![](/docs/images/proj1/05-browser-op.PNG)


