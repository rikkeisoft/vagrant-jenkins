# Vagrant Jenkins

## Prerequisites
* [VirtualBox](https://www.virtualbox.org/)
* [Vagrant](https://www.vagrantup.com/)

## Installation
Build the vagrant box

```sh
vagrant up
```

To access the Jenkins server

```sh
http://localhost:6969
```

or, add the following line to the hosts file

```sh
127.0.0.1   jenkins.dev
```

and then run the server with

```sh
http://jenkins.dev:6969
```

## First time accessing Jenkins
Since version 2.0 Jenkins has a security setup wizard when first running it after the installation.

SSH into the machine with

```sh
vagrant ssh
```

Locate the security password

```sh
cat /var/lib/jenkins/secrets/initialAdminPassword
```

and copy it into the password field on the Jenkins server.
