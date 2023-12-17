# Ansible

Configuration Management tool - To automate configuring servers, installing tools, deploy apps, etc.

## Content

This contains sections about:

-   Intro to Ansible
-   Project 1 Deploy Node.js App 


## Intro to Ansible 

- tool to automate any IT task. (eg: updating docker version on set of nodes, etc)
- manual approach: to ssh into each node and do task.
- configuration, deployment or installation steps in a single reusable file
- need ssh access to target servers from control machine.
- agentless
- modules: small programs that does actual work. moved to remote server -> executed and removed.
- Play: tasks, with which user and on which hosts
- Playbook: one or more plays in a YAML. orchestrates module execution.
- hosts are defined in inventory yaml -> can group multiple hosts together and execute tasks on them.
- Ansible Tower: UI Dashboard -> centrally store automation tasks
- Ansible vs (Chef, Puppet): Ruby is difficult to learn -> need to install agent and manage updates 
- control node is not supported on windows   
- brew install ansible (or) pip install ansible
- default location for hosts file: /etc/ansible/hosts -> /etc/ansible/ansible.cfg
- ansible_user, ansible_ssh_private_key_file, ansible_python_interpreter can be configured for each host
- adhoc commands: fast way to interact with desired servers
- eg: ansible [pattern] -m [module] -a [module_options]
- eg: ansible 18.30.83.10 -i hosts -m ping ( can target group of hosts by replacing ip with group name )
- define ansible_ssh_private_key_file, ansible_python_interpreter and ansible_user under [ group: vars ] in inventory file, if there are more hosts
- ansible needs python3 as interpreter, so set ansible_python_interpreter=/usr/bin/python3
- Host Key Check prompt (yes/no): guards againt server spoofing and MIM attacks -> don't want interactive mode
- for long-lived servers : update ~/.ssh/known_hosts file on with target host Ip. ssh-keyscan -H <ip_address_of_target_host> >> ~/.ssh/known_hosts -> also remote server needs to authenticate control node so add client public key in ~/.ssh/authorized_keys
-> ssh-copy-id root@Ip will add public key to remote
- for ephemeral or temporary servers : Disable host key checking. less secure. update ansible default config file with `host_key_checking=False`
- Play is a group of tasks. Each has name, hosts and tasks.
- Idempotency ( checks current state with desired state )
- ansible 2.9 & earlier : single package called ansible ( code, modules, plugins )
- ansible 2.10 & later: modules & plugins are moved to various collections
- collection: A packaging format for bundling & distributing ansible content.
- name of module: apt vs ansible.builtin.apt
- Ansible Galaxy: online hub which has all collections. `ansible-galaxy collections list` lists collections that are local on control node. To install or upgrade `ansible-galaxy install <coll_name>`


## Project 1 Deploy Node.js App

