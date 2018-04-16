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
few tasks on it and then destroy the instance to clean up. This is
 somewhat elaborate for a first example but for the intended audience
 has most of the features of Ansible that I wanted to showcase.

For this demo the following AWS entities were set up ahead of time:

* AWS account
* AWS keypair (named ansible_demo here)
* VPC with subnet
* Security group (default group used here)
* IAM access key credentials with permissions to create and terminate
 EC2

The `ansible_demo.yml` file contains the following play:

```yaml
---

- name: Instantiate EC2 instance
  hosts: localhost
  connection: local
  gather_facts: False
  vars:
    aws_access_key_var: !vault |
                    $ANSIBLE_VAULT;1.1;AES256
                    66366163333262646135623133356437303563623735623331653236663130636533653966663930
                    3932333832393262623963626637336535386233633534650a633638323730653964346563373861
                    33323262396235356265653938636263623430313362623235633063373964633236303265636637
                    3633643534613135620a393165623932376265316233373563653163343335633266363566616363
                    61656365353361653435633861306135303634656338373065666234653339633137
    aws_secret_key_var: !vault |
                    $ANSIBLE_VAULT;1.1;AES256
                    30663135623538353738323537653064333864613566666261353561333164373136356633323739
                    3436633337373466613730363261383933616663333031630a393364356661353665353237313732
                    38613537623134653664313438376234626164366330643034633036306666313636353438626334
                    6131396461386461630a326633346236656566393133323936636561663538353637333932313039
                    34646530636662303962316131393138633535626134306438363862346237303064653839313563
                    6433623465353962326436663637323437376663393833376138

  tasks:
#    - debug:
#        msg: "{{ aws_secret_key_var }}"

    - name: Provision a set of instances
      ec2:
        key_name: ansible_demo
        aws_access_key: "{{ aws_access_key_var }}"
        aws_secret_key: "{{ aws_secret_key_var }}"
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
  vars:
    aws_access_key_var: !vault |
                    $ANSIBLE_VAULT;1.1;AES256
                    66366163333262646135623133356437303563623735623331653236663130636533653966663930
                    3932333832393262623963626637336535386233633534650a633638323730653964346563373861
                    33323262396235356265653938636263623430313362623235633063373964633236303265636637
                    3633643534613135620a393165623932376265316233373563653163343335633266363566616363
                    61656365353361653435633861306135303634656338373065666234653339633137
    aws_secret_key_var: !vault |
                    $ANSIBLE_VAULT;1.1;AES256
                    30663135623538353738323537653064333864613566666261353561333164373136356633323739
                    3436633337373466613730363261383933616663333031630a393364356661353665353237313732
                    38613537623134653664313438376234626164366330643034633036306666313636353438626334
                    6131396461386461630a326633346236656566393133323936636561663538353637333932313039
                    34646530636662303962316131393138633535626134306438363862346237303064653839313563
                    6433623465353962326436663637323437376663393833376138
  tasks:
    - name: Terminate instances that were previously launched
      ec2:
        aws_access_key: "{{ aws_access_key_var }}"
        aws_secret_key: "{{ aws_secret_key_var }}"
        state: 'absent'
        instance_ids: '{{ ec2.instance_ids }}'
```

To use this play yourself you will need to create a local file and place
 a strong password within it. Then take your AWS access and secret keys
 and generate the ansible vault encrypted values using the newly created
 conda virtualenv and then following executing the following command:

```
source /path/to/miniconda/envs/ansible_demo
ansible-vault encrypt_string --vault_password_file=vault_pass.txt '<AWS ACCESS KEY HERE>' --name 'aws_access_key'
```

and likewise:

```
ansible-vault encrypt_string --vault-password-file=vault_pass.txt '<AWS SECRET KEY HERE>' --name 'aws_secret_key'
```

where `vault_pass.txt` is the local file holding the strong password.
Copy and paste the output into the playbook for the relevant lines.

To execute the play then run the following command from the command line
assuming you still have the conda virtualenv loaded:

```
ansible-playbook /path/to/ansible_demo.yml --vault-password-file /path/to/vault_pass.txt
```

# The Ansible Vault

The vault works by decrypting anything encrypted within the run time
scope if a decryption method is provided in the ansible-playbook command
 such as a password file or password itself.

In the above example individual string variables have been encrypted however
Ansible can handle entire files containing encrypted data also.

Projects I have worked on have also used a password manager beloning to
an OS as the means of authentication for unlocking the vault using a
shell script and setting this as the method of authentication via the
`ANSIBLE_VAULT_PASSWORD_FILE` variable set in the `ansible.cfg` file.
This file sits in the root of all playbook folders by default and is a
means of controlling the local Ansible run time environment.

# YAML & Ansible Syntax

* For full documentation see [http://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html](http://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)
* Ansible plays always begin with `---` at the start of the file.
* All tasks should start with a `-name`. This aids with understanding on
examining run time output e.g.

```
PLAY [Instantiate EC2 instance] ***************************************************************************************************************************************************************

TASK [Provision a set of instances] ***********************************************************************************************************************************************************
changed: [localhost]

TASK [Add new instance to host group] *********************************************************************************************************************************************************
changed: [localhost] => (item={u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-10-0-0-230.eu-west-1.compute.internal', u'public_ip': u'', u'private_ip': u'10.0.0.230', u'id': u'', u'ebs_optimized': False, u'state': u'running', u'virtualization_type': u'hvm', u'architecture': u'x86_64', u'ramdisk': None, u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u''}}, u'key_name': u'ansible_demo', u'image_id': u'ami-f90a4880', u'tenancy': u'default', u'groups': {u'': u'default'}, u'public_dns_name': u'', u'state_code': 16, u'tags': {}, u'placement': u'eu-west-1a', u'ami_launch_index': u'0', u'dns_name': u'', u'region': u'eu-west-1', u'launch_time': u'', u'instance_type': u't2.micro', u'root_device_name': u'/dev/sda1', u'hypervisor': u'xen'})

TASK [Wait for SSH to come up] ****************************************************************************************************************************************************************
ok: [localhost] => (item={u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-10-0-0-230.eu-west-1.compute.internal', u'public_ip': u'', u'private_ip': u'10.0.0.230', u'id': u'', u'ebs_optimized': False, u'state': u'running', u'virtualization_type': u'hvm', u'architecture': u'x86_64', u'ramdisk': None, u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u''}}, u'key_name': u'ansible_demo', u'image_id': u'ami-f90a4880', u'tenancy': u'default', u'groups': {u'': u'default'}, u'public_dns_name': u'', u'state_code': 16, u'tags': {}, u'placement': u'eu-west-1a', u'ami_launch_index': u'0', u'dns_name': u'', u'region': u'eu-west-1', u'launch_time': u'', u'instance_type': u't2.micro', u'root_device_name': u'/dev/sda1', u'hypervisor': u'xen'})

PLAY [Terminate EC2 instance] *****************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************
ok: [localhost]

TASK [Terminate instances that were previously launched] **************************************************************************************************************************************
changed: [localhost]

PLAY RECAP ************************************************************************************************************************************************************************************
localhost                  : ok=5    changed=3    unreachable=0    failed=0
```

* Lists:

```yaml
fruits:
    - Apple
    - Orange
    - Strawberry
    - Mango
```

* Dictionaries:
```yaml
martin:
    name: Martin D'vloper
    job: Developer
    skill: Elite
```

* Comments:
```yaml
# This is a comment
````

* Variable - Ansible uses `“{{ var }}”` for variables:
```yaml
foo: "{{ variable }}"
```

# The Ansible Project Structure

# Ansible Galaxy

# Ansible Reference

# Examples

* Copying a file
* Creating an EC2 instance

