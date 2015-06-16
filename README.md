Creates an Amazon AWS Cluster using using CloudFormation template and 
Ansible to provision ELK stack on Ubuntu 14.04 Trusty

REQUIREMENTS:
 - Python 2.7.x
 - Ansible
 - AWS CLI tools
 - Python Boto is setup and configured to use the AWS keys. 
   the ~/.boto file contains aws_access_key and aws_secret_key
 - A Valid AWS KeyFile and added to the proper security role. The .pem file should loaded into
   the ROOT folder of the project  

CREATE INSTANCE using a CloudFormation Template:
# Create an AWS stack using command line (preferred method) 
aws cloudformation create-stack --stack-name ansible-elk --template-body file://files/cloudformation-ansible.json --parameters ParameterKey=KeyName,ParameterValue={Personal VALID AWS KeyName} ParameterKey=DiskType,ParameterValue=ephemeral ParameterKey=InstanceType,ParameterValue=t1.micro ParameterKey=ClusterSize,ParameterValue=1 --capabilities CAPABILITY_IAM

ex.) testweb is the keypair that is permissioned for my account and should be accessible (my testweb.pem file is located in same directory)
aws cloudformation create-stack --stack-name ansible-elk --template-body file://files/cloudformation-ansible.json --parameters ParameterKey=KeyName,ParameterValue=testweb ParameterKey=DiskType,ParameterValue=ephemeral ParameterKey=InstanceType,ParameterValue=t1.micro ParameterKey=ClusterSize,ParameterValue=1 --capabilities CAPABILITY_IAM

DELETE INSTANCE :
# Delete an AWS stack using command line (preferred method)
aws cloudformation delete-stack --stack-name ansible-elk
 
PROVISION INSTANCE ELK Stack using ansible:
# Provision the stack using ansible playbook using ec2.py Inventory script 
 (requires boto installation and setup using valid aws credentials from the control machine) (preferred method)
ansible-playbook -i plugins/ec2.py   --private-key={Personal VALID AWS KeyName}.pem provision_stack.yml 

ex.) ansible-playbook -i plugins/ec2.py   --private-key=testweb.pem provision_stack.yml

# OR Provision the stack using ansible playbook using a "hosts" file - Must manually put the created hosts name in the launched category
ansible-playbook -i hosts  provision_stack.yml

# Connect to Kibana on port 80 of newly create AWS instance using the follwoing credentials
  - username: kibanaadmin
  - password: test123
# Forward logs on port 5000 and use valid security cert  


NSTRUCTIONS TO CREATE a new valid certificate to be used with logstash on server
# To create a new cert on the logstash server - used for forwarder secure connection
sudo vi /etc/ssl/openssl.cnf
# Find the [ v3_ca ] section in the file, and add this line under it (substituting in the Logstash Server's private IP address):
subjectAltName = IP: <logstash_server_private_ip>
c mple/etc/pki/tls
sudo openssl req -config /etc/ssl/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout logstash-forwarder-example.key -out logstash-forwarder-example.crt
# Copy the new crt file to the forwarder

# ANSIBLE
http://docs.ansible.com/intro_installation.html#getting-ansible (full installation dependencies per os)
sudo pip install ansible # easiest way

# AWS CLI Tools
sudo pip install awscli

 -1) add the following to your ~/.profile
     export LC_ALL=en_US.UTF-8
     export LANG=en_US.UTF-8
 -2) run the "aws configure" command and add "Access Key ID and Secret Access Key credentials when prompted 

  # Useful link for CLI and Cloudformation
  http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-cli.html

# Getting Started with BOTO
http://boto.readthedocs.org/en/latest/getting_started.html

# Dynamic Inventory with Ansible
http://docs.ansible.com/intro_dynamic_inventory.html
