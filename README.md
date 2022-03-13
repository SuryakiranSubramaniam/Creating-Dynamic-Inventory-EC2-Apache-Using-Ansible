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
