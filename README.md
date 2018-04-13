<!---Command: python -m markdown -x toc -x fenced_code -x def_list -x codehilite -x meta -c md_cfg.json README.md > README.html)--->

# Overview

# Install Anaconda

Anaconda® is a package manager, an environment manager, a Python
distribution, and a collection of over 1,000+ open source
packages. I used the Miniconda version of it as a reliable clean
environment within which to set up Ansible.

[https://conda.io/miniconda.html](https://conda.io/miniconda.html)

This downloads a shell script wizard from which you can set up the
Miniconda environment manager.

# Create Conda Virtualenv

`/path/to/miniconda2/bin/conda create -n ansible_demo`

# Install Ansible

Install Ansible using the conda-forge conda channel

`/path/to/miniconda2/bin/conda install -c conda-forge -n ansible_demo ansible`

# The First Playbook

As an example the playbook will create an AWS EC2 instance, run a
few tasks on it and then destroy the instance to clean up.

First add an IAM keypair to your local environment:



For this an AWS IAM keypair will be required and also a VPC and
corresponding subnet. The `ansible_demo.yml` file contains the following
playbook:


```yaml
---

- name: Instantiate EC2 instance
  hosts: localhost
  connection: local
  gather_facts: False

  tasks:

    - name: Provision a set of instances
      ec2:
         key_name: ansible_demo
         aws_access_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66366163333262646135623133356437303563623735623331653236663130636533653966663930
          3932333832393262623963626637336535386233633534650a633638323730653964346563373861
          33323262396235356265653938636263623430313362623235633063373964633236303265636637
          3633643534613135620a393165623932376265316233373563653163343335633266363566616363
          61656365353361653435633861306135303634656338373065666234653339633137
         aws_secret_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30663135623538353738323537653064333864613566666261353561333164373136356633323739
          3436633337373466613730363261383933616663333031630a393364356661353665353237313732
          38613537623134653664313438376234626164366330643034633036306666313636353438626334
          6131396461386461630a326633346236656566393133323936636561663538353637333932313039
          34646530636662303962316131393138633535626134306438363862346237303064653839313563
          6433623465353962326436663637323437376663393833376138
         group: default
         instance_type: t2.micro
         image: ami-f90a4880
         wait: true
         region: eu-west-1
         vpc_subnet_id: subnet-c8b91dae
         assign_public_ip: yes
      register: ec2

    - name: Add new instance to host group
      add_host:
          hostname: "{{ item.public_ip }}"
          groupname: launched
      with_items: "{{ ec2.instances }}"

    - name: Wait for SSH to come up
      wait_for:
          host: "{{ item.public_dns_name }}"
          port: 22
          delay: 60
          timeout: 320
          state: started
      with_items: "{{ ec2.instances }}"

- name: Terminate EC2 instance
  hosts: localhost
  connection: local
  tasks:
    - name: Terminate instances that were previously launched
      ec2:
        aws_access_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66366163333262646135623133356437303563623735623331653236663130636533653966663930
          3932333832393262623963626637336535386233633534650a633638323730653964346563373861
          33323262396235356265653938636263623430313362623235633063373964633236303265636637
          3633643534613135620a393165623932376265316233373563653163343335633266363566616363
          61656365353361653435633861306135303634656338373065666234653339633137
        aws_secret_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30663135623538353738323537653064333864613566666261353561333164373136356633323739
          3436633337373466613730363261383933616663333031630a393364356661353665353237313732
          38613537623134653664313438376234626164366330643034633036306666313636353438626334
          6131396461386461630a326633346236656566393133323936636561663538353637333932313039
          34646530636662303962316131393138633535626134306438363862346237303064653839313563
          6433623465353962326436663637323437376663393833376138
        state: 'absent'
        instance_ids: '{{ ec2.instance_ids }}'
```

To execute this

# YAML Syntax

# The Ansible Project Structure

# The Ansible Vault

# Ansible Reference

# Examples

* Copying a file
* Creating an EC2 instance

