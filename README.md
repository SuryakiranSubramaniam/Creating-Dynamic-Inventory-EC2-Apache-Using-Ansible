# Creating-Dynamic-Inventory-EC2-Apache-Using-Ansible
Creating-Dynamic-Inventory-EC2-Apache-Using Ansible

## Installing packages we need

```
amazon-linux-extras install ansible2 -y 
amazon-linux-extras install epel -y
yum install python3 -y
yum install python3-pip -y
pip install awscli --upgrade
ansible-galaxy collection install amazon.aws
pip3 install boto &> /dev/null
pip3 install boto3 &> /dev/null
```

## hosts file

> vim hosts

```
localhost ansible_connection=local ansible_python_interpreter=/usr/bin/python3
```

## Playbook

> vim dynamic.yml

```
---
- name: "Creating EC2 Using Ansible"
  hosts: localhost
  vars:
    ak: "AKIAUWVRRVN4HR56HPNL"
    sk: "tv5jQb0NJPw8L1sGIJV8yzfPjHKVH/e8IXMXMEN3"
    region: "ap-south-1"
    pro: "zomato"
    it: "t2.micro"
    ia: "ami-0e0ff68cb8e9a188a"
  tasks:
    
    - name: "Creating SSH-Key Pair"
      amazon.aws.ec2_key:
        aws_access_key: "{{ ak }}"
        aws_secret_key: "{{ sk }}"
        region: "{{ region }}"
        name: "{{pro}}"
        state: present
        tags:
          Name: "{{ pro }}"
          project: "{{ pro }}"
        
      register: ks
        
        
    - name: "Saving Private Key Of Zomato"
      when: ks.changed == true 
      copy:
        content: "{{ ks.key.private_key}}"
        dest: "{{ pro }}.pem"
        mode: 0400

    - name: "Creating Security Group"
      amazon.aws.ec2_group:
        aws_access_key: "{{ ak }}"
        aws_secret_key: "{{ sk }}"
        name: "Zomato-Webserver"
        description: "allows 80,443 from all"
        region: "{{ region }}"
        
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
               
          - proto: tcp
            from_port: 443
            to_port: 443
            cidr_ip: 0.0.0.0/0
        tags:
          Name: "Zomato-webserver"
          project: "{{ pro }}"
      register: webserver

    - name:  "Oppening port 22"
      amazon.aws.ec2_group:
        aws_access_key: "{{ ak }}"
        aws_secret_key: "{{ sk }}"
        name: "remote acess"
        description: "allows 22 from all"
        region: "{{ region }}"
        
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0              
                
        tags:
          Name: "{{ pro }}-remote"
          project: "{{ pro }}"
      register: remote

    - name:  "Creating An Ec2 Instance" 
      amazon.aws.ec2:
        aws_access_key: "{{ ak }}"
        aws_secret_key: "{{ sk }}"
        region: "{{ region }}"
        key_name: "{{ ks.key.name}}"
        instance_type: "{{ it }}"
        image: "{{ ia }}"
        wait: yes
        group_id:
          - "{{ webserver.group_id }}"
          - "{{ remote.group_id }}"
        instance_tags:
          Name: "{{ pro }}-webserver"
          project: "{{ pro }}"
        count_tag:
          Name: "{{ pro }}-webserver"
        exact_count: 2
      register: ec2_status
        
    - debug: var=ec2_status
```

## Checking Syntax

> ansible-playbook -i hosts dynamic.yml --syntax-check

## Running Playbook

> ansible-playbook -i hosts dynamic.yml

```
PLAY [Creating EC2 Using Ansible] **********************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Creating SSH-Key Pair] ***************************************************
ok: [localhost]

TASK [Saving Private Key Of Zomato] ********************************************
skipping: [localhost]

TASK [Creating Security Group] *************************************************
ok: [localhost]

TASK [Oppening port 22] ********************************************************
ok: [localhost]

TASK [Creating An Ec2 Instance] ************************************************
[DEPRECATION WARNING]: The 'ec2' module has been deprecated and replaced by the
 'ec2_instance' module'. This feature will be removed in version 4.0.0. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in 
ansible.cfg.
ok: [localhost]

TASK [Creating Dynamic Inventory] **********************************************
changed: [localhost] => (item={u'ramdisk': None, u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-31-1-53.ap-south-1.compute.internal', u'block_device_mapping': {u'/dev/xvda': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u'vol-08e2e4ac5b15dcad4'}}, u'key_name': u'zomato', u'public_ip': u'65.0.11.140', u'image_id': u'ami-0e0ff68cb8e9a188a', u'tenancy': u'default', u'private_ip': u'172.31.1.53', u'groups': {u'sg-0596e695580ece52e': u'remote acess', u'sg-09cf6f1c63258512d': u'Zomato-Webserver'}, u'public_dns_name': u'ec2-65-0-11-140.ap-south-1.compute.amazonaws.com', u'state_code': 16, u'id': u'i-056a35ed9f9dc6c8b', u'tags': {u'project': u'zomato', u'Name': u'zomato-webserver'}, u'placement': u'ap-south-1b', u'ami_launch_index': u'0', u'dns_name': u'ec2-65-0-11-140.ap-south-1.compute.amazonaws.com', u'region': u'ap-south-1', u'ebs_optimized': False, u'launch_time': u'2022-03-12T09:49:17.000Z', u'instance_type': u't2.micro', u'state': u'running', u'root_device_name': u'/dev/xvda', u'hypervisor': u'xen', u'virtualization_type': u'hvm', u'architecture': u'x86_64'})
changed: [localhost] => (item={u'ramdisk': None, u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-31-8-125.ap-south-1.compute.internal', u'block_device_mapping': {u'/dev/xvda': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u'vol-0b73619077c64e2bd'}}, u'key_name': u'zomato', u'public_ip': u'35.154.5.45', u'image_id': u'ami-0e0ff68cb8e9a188a', u'tenancy': u'default', u'private_ip': u'172.31.8.125', u'groups': {u'sg-0596e695580ece52e': u'remote acess', u'sg-09cf6f1c63258512d': u'Zomato-Webserver'}, u'public_dns_name': u'ec2-35-154-5-45.ap-south-1.compute.amazonaws.com', u'state_code': 16, u'id': u'i-09e89f3b8d6d4d7f1', u'tags': {u'project': u'zomato', u'Name': u'zomato-webserver'}, u'placement': u'ap-south-1b', u'ami_launch_index': u'1', u'dns_name': u'ec2-35-154-5-45.ap-south-1.compute.amazonaws.com', u'region': u'ap-south-1', u'ebs_optimized': False, u'launch_time': u'2022-03-12T09:49:17.000Z', u'instance_type': u't2.micro', u'state': u'running', u'root_device_name': u'/dev/xvda', u'hypervisor': u'xen', u'virtualization_type': u'hvm', u'architecture': u'x86_64'})

TASK [Waiting 120 seconds] *****************************************************
skipping: [localhost]

PLAY [Ec2 Instances Provisioning] **********************************************

TASK [Gathering Facts] *********************************************************
[WARNING]: Platform linux on host 65.0.11.140 is using the discovered Python
interpreter at /usr/bin/python, but future installation of another Python
interpreter could change this. See https://docs.ansible.com/ansible/2.9/referen
ce_appendices/interpreter_discovery.html for more information.
ok: [65.0.11.140]
[WARNING]: Platform linux on host 35.154.5.45 is using the discovered Python
interpreter at /usr/bin/python, but future installation of another Python
interpreter could change this. See https://docs.ansible.com/ansible/2.9/referen
ce_appendices/interpreter_discovery.html for more information.
ok: [35.154.5.45]

TASK [Installing Apacher Webserver] ********************************************
changed: [65.0.11.140]
changed: [35.154.5.45]

TASK [Index.html] **************************************************************
changed: [65.0.11.140]
changed: [35.154.5.45]

TASK [Restart & enable httpd] **************************************************
changed: [65.0.11.140]
changed: [35.154.5.45]

TASK [Website: URL] ************************************************************
ok: [65.0.11.140] => (item=65.0.11.140) => {
    "msg": "http://65.0.11.140"
}
ok: [65.0.11.140] => (item=35.154.5.45) => {
    "msg": "http://35.154.5.45"
}

PLAY RECAP *********************************************************************
35.154.5.45                : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
65.0.11.140                : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0 
```

## OutPut

**The following were created:**
- EC2 Instance
- Security Group
- Keypair
- Tags
- Apache Using Dynamic Inventory

![alt text]()
![alt text]()
