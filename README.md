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
