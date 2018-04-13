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

