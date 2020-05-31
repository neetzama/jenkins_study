# Infrasturucture enviroment automation
This is intended for automation of infrastructure environment construction.
The overall configuration diagram is attached below.

![jenkins構成図](https://user-images.githubusercontent.com/60305322/82865800-95f9aa00-9f62-11ea-9c8f-50e696a4ff3c.png)

- Resource construction automation with AWS CloudFormation.
- Automation of middleware installation and various settings by Ansible.
- Testing framework used Severspec.
- CI tool used jenkins.
- Configuration management tool used GitHub.

## Description
I've only published about Jenkins here.
For details on AWS cloudformation, Ansible, Serverspec, please refer to the links attached below.
- Details of AWS CloudFormation are [available here][1].
- Details of Ansible are [available here][2].
- Details of Serverspec are [available here][3].
- GitHub and Jenkins are linked by Webhooks.

## Requirements
- EC2 to install jenkins is Amazon Linux 2
- Jenkins 2.237
- ansible 2.9.5
- rbenv 1.1.2-30-gc879cb0
- ruby 2.6.5p114
- git 2.23.3
- aws-cli/1.16.300 Python/2.7.16 Linux/4.14.173-137.229.amzn2.x86_64 botocore/1.13.36
- Target EC2 is Ubuntu 18.04

### Create S3 IAM and S3 bucket
You need to create an IAM for S3 and an S3 bucket.
This will be used later in Ansible job.
- [See here][7] for how to create an S3 IAM
- [See here][8] for how to create an S3 bucket

### Creating and setting the ssh key
EC2 that installs jenkins expects to run Ansible and Serverspec against the target server. Therefore, it is necessary to create the SSH key in advance.<br>
Also, when creating and setting the ssh key, you need to switch to the jenkins user.<br>
How to switch to jenkins user is [posted here][4].

1. Create `.ssh` directory under `/var/lib/jenkins/`
```
mkdir .ssh
```
2. Create ssh key
```
$ cd /var/lib/jenkins/.ssh
$ ssh-keygen -t rsa
```
3. Create an ssh configuration file and add the information
```
$ cd /var/lib/jenkins/.ssh
$ sudo vi config

  Host payblog
      HostName 176.34.32.51
      User ubuntu
      IdentityFile /var/lib/jenkins/.ssh/id_rsa
```
### Setting up the AWS CLI
EC2 that installs jenkins needs to be able to use the AWS CLI.
Please refer to [this][6] for the details of the setting method.<br>
You also need to be a jenkins user.
Details on how to switch are [posted here][4].

### Create a target EC2 AMI
The AMI created here will be used when creating the AWS CloudFormation template file.

1. Connect to EC2 (Amazon Linux 2) with jenkins installed

2. Copy the contents of `/var/lib/jenkins/.ssh/id_rsa.pub`

3. Prepare EC2 (Ubuntu 18.04)

4. Connect to the prepared EC2

5. Paste the copied contents to `/home/ubuntu/.ssh/authorized_key`

6. Open the EC2 dashboard in the AWS Management Console, select EC2 (Ubuntu 18.04) and create an AMI.

### Switch to jenkins user
1. You will need to rewrite `/etc/passwd` to allow the jenkins user to use the shell
```/etc/passwd
root:x:0:0:root:/root:/bin/bash

~ Omitted ~

jenkins:x:996:994:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
```
2. Switch from ec2-user to jenkins user
```
$ sudo su - jenkins
```
### Allow jenkins users to use sudo
1. Edit `/etc/sudoers` with visudo
```
$ sudo visudo
```
2. Add `jenkins ALL=(ALL) NOPASSWD:ALL`
```
#
# Refuse to run if unable to disable echo on the tty.
#
Defaults   !visiblepw
jenkins ALL=(ALL) NOPASSWD:ALL
```
### Webhook integration of GitHub and Jenkins
It will be a specification to execute jenkins job when pushed to each repository of Github. Therefore, you need to set up GitHub webhook.
#### API token acquisition
In Jenkins, do "Manage Jenkins> Manage users> Settings> API token" to get the API token used for authentication
#### GitHub webhook creation
Set up a webhook by doing "Settings> Webhooks> Add Webhook" on GitHub

## Usage
### Creating a Serverspec job
Select Create New Job and then select Build Freestyle Project.

Set up the job<br>
1. **General**<br>
- Check "GitHub project"<br>
  Enter `https://github.com/neetzama/serverspec_study.git` which is Serverspec's repository in "Project url"

- Check "Build parameterization"<br>
  Select "String" in "Add Parameter" and enter "payload" in the name

2. **Source code management**<br>
Select "Git" and enter `https://github.com/neetzama/serverspec_study.git` for the repository URL

3. **Build environment**<br>
Check "Delete workspace before build" and "Add timestamp to console output"

4. **Build**<br>
Choose to run shell
```
cd /var/lib/jenkins/workspace/Serverspec/serverspec
/var/lib/jenkins/.rbenv/shims/rake spec
```
5. **Post-build processing**<br>
Add "E-mail notification" and enter your email address.<br>
This will send you an email when the build fails.

### Creating an Ansible job
Select Create New Job and then select Build Freestyle Project.

Set up the job<br>
1. **General**<br>
- Check "GitHub project"<br>
  Enter `https://github.com/neetzama/ansible_study.git` which is Ansible's repository in "Project url"

- Check "Build parameterization"<br>
  Select "String" in "Add Parameter" and enter "payload" in the name

2. **Source code management**<br>
Select "Git" and enter `https://github.com/neetzama/ansible_study.git` for the repository URL

3. **Build environment**<br>
Check "Delete workspace before build" and "Add timestamp to console output"

4. **Build**<br>
Choose to run shell.
As an option, we will pass variables here that can not be published to GitHub.<br>
Describes the variable.
- db_name : The name of the database
- db_user : Database master user name
- db_peer_password : Database master password
- db_host : AWS RDS endpoint
- DJANGO_AWS_ACCESS_KEY_ID : IAM created by ["Create S3 IAM and S3 bucket"][9]
Access key
- DJANGO_AWS_SECRET_ACCESS_KEY : IAM created by ["Create S3 IAM and S3 bucket"][9]
Secret access key
- DJANGO_AWS_STORAGE_BUCKET_NAME : Name of the S3 bucket created with ["Create S3 IAM and S3 bucket"][9]
- SECRET_KEY : django secret key
```
cd /var/lib/jenkins/workspace/Ansible
ansible-playbook -i hosts playbook.yml \
 -e '{db_name: "foo"}' \
 -e '{db_user: "foo"}' \
 -e '{db_peer_password: "foo"}' \
 -e '{db_host: "foo"}' \
 -e '{DJANGO_AWS_ACCESS_KEY_ID: foo}' \
 -e '{DJANGO_AWS_SECRET_ACCESS_KEY: foo}' \
 -e '{DJANGO_AWS_STORAGE_BUCKET_NAME: foo}' \
 -e '{SECRET_KEY: foo}' 
```
5. **Post-build processing**<br>
Add "Build another project" and select the Serverspec job.

### Creating an AWS CloudFormation job
Select Create New Job and then select Build Freestyle Project.

Set up the job<br>
1. **General**<br>
- Check "GitHub project"<br>
  Enter `https://github.com/neetzama/cloudformation_study.git/` which is AWS CloudFormation's repository in "Project url"

- Check "Build parameterization"<br>
  Select "String" in "Add Parameter" and enter "payload" in the name.
2. **Source code management**<br>
Select "Git" and enter `https://github.com/neetzama/cloudformation_study.git` for the repository URL

3. **Build environment**<br>
Check "Delete workspace before build" and "Add timestamp to console output"

4. **Build**<br>
Choose to run shell
```
aws cloudformation delete-stack \
--stack-name jenkins-cloudformation \
--region ap-northeast-1

aws cloudformation wait stack-delete-complete \
--stack-name jenkins-cloudformation

aws cloudformation create-stack \
--stack-name jenkins-cloudformation \
--region ap-northeast-1 \
--template-body file:///var/lib/jenkins/workspace/CloudFormation/study.yml

aws cloudformation wait stack-create-complete \
--stack-name jenkins-cloudformation
```
5. **Post-build processing**<br>
Add "Build another project" and select the Ansible job.

## Install
### Install Jenkins on EC2
1. Install JDK 8
```
$ sudo yum install -y java-1.8.0-openjdk-devel.x86_64`
```
2. Add Jenkins yum repository
```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
```
3. Change the baseurl of the connection destination from http to https
```
$ sudo vi /etc/yum.repos.d/jenkins.repo
[jenkins]
name=Jenkins
baseurl=https://pkg.jenkins.io/redhat
gpgcheck=1
```
4. Install Jenkins
```
$ sudo yum install -y jenkins
```

### Install Ansible on EC2
```
$ sudo amazon-linux-extras install -y ansible2
```

### Install Serverspec on EC2
1. Update yum
```
$ sudo yum update -y
```
2. Install git
```
$ sudo yum install git -y
```
3. Change ec2-user to jenkins user here.
Details on how to switch are [posted here][4].<br>
Also, the jenkins user must be able to use sudo.
Details are [posted here][5].

4. Clone rbenv from repository
```
$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
```
5. pass through rbenv PATH
```
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
$ source ~/.bash_profile
```
6. Install ruby-build.
Clone ruby-build from repository
```
$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```
7. Perform the installation
```
$ cd ~/.rbenv/plugins/ruby-build
$ sudo ./install.sh
```
8. Install the packages required for ruby installation
```
$ sudo yum -y install gcc-c++ glibc-headers openssl-devel readline libyaml-devel readline-devel zlib zlib-devel libffi-devel libxml2 libxslt libxml2-devel libxslt-devel sqlite-devel
```
9. Install ruby 2.6.5
```
$ rbenv install 2.6.5
```
10. Specify the version of Ruby used in rbenv
```
$ rbenv global 2.6.5
```
11. Install Serverspec
```
$ gem install serverspec
```

[1]:https://github.com/neetzama/cloudformation_study
[2]:https://github.com/neetzama/ansible_study
[3]:https://github.com/neetzama/serverspec_study
[4]:#switch-to-jenkins-user
[5]:#allow-jenkins-users-to-use-sudo
[6]:https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-configure.html
[7]:https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_users_create.html#id_users_create_console
[8]:https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/user-guide/create-bucket.html
[9]:#create-s3-iam-and-s3-bucket