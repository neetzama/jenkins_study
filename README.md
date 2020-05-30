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

## Requirement
- EC2 to install jenkins is Amazon Linux 2
- Target EC2 is Ubuntu 18.04
- Jenkins 2.237
- ansible 2.9.5
- rbenv 1.1.2-30-gc879cb0
- ruby 2.6.5p114
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
      HostName IP address 
      User ubuntu
      IdentityFile /var/lib/jenkins/.ssh/id_rsa
```
### Create a target EC2 AMI
1. Connect to EC2 (Amazon Linux 2) with jenkins installed

2. Copy the contents of `/var/lib/jenkins/.ssh/id_rsa.pub`

3. Prepare EC2 (Ubuntu 18.04)

4. Connect to the prepared EC2

5. Paste the copied contents to `/home/ubuntu/.ssh/authorized_key`

6. Open the EC2 dashboard in the AWS Management Console, select EC2 (Ubuntu 18.04) and create an AMI.

## Usage

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

[1]:https://github.com/neetzama/cloudformation_study
[2]:https://github.com/neetzama/ansible_study
[3]:https://github.com/neetzama/serverspec_study
[4]:#switch-to-jenkins-user
[5]:#allow-jenkins-users-to-use-sudo