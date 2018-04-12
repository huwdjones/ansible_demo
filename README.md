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

For this an AWS IAM keypair will be required.

```yaml
---

- hosts: localhost
  connection: local
  gather_facts: False

  tasks:

    - name: Provision a set of instances
      ec2:
         key_name: my_key
         group: test
         instance_type: t2.micro
         image: "{{ ami_id }}"
         wait: true
         exact_count: 5
         count_tag:
            Name: Demo
         instance_tags:
            Name: Demo
      register: ec2

    - name: Add all instance public IPs to host group
      add_host: hostname={{ item.public_ip }} groups=ec2hosts
      loop: "{{ ec2.instances }}"
```

# YAML Syntax

# The Ansible Project Structure

# The Ansible Vault

# Ansible Reference

# Examples

* Copying a file
* Creating an EC2 instance

