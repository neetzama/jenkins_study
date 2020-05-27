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
- EC2 that installs jenkins expects to run Ansible and Serverspec against the target server. Therefore, it is necessary to create the SSH key in advance.

## Requirement
- EC2 to install jenkins is Amazon Linux 2.
- Jenkins 2.237
- ansible 2.9.7
- rbenv 1.1.2-30-gc879cb0
- ruby 2.6.5p114

## Install
### Install Jenkins on EC2
1. Install JDK 8.
```
$ sudo yum install -y java-1.8.0-openjdk-devel.x86_64`
```

2. Add Jenkins yum repository.
```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
```
3. Change the baseurl of the connection destination from http to https.
```
$ sudo vi /etc/yum.repos.d/jenkins.repo
[jenkins]
name=Jenkins
baseurl=https://pkg.jenkins.io/redhat
gpgcheck=1
```


[1]:https://github.com/neetzama/cloudformation_study
[2]:https://github.com/neetzama/ansible_study
[3]:https://github.com/neetzama/serverspec_study