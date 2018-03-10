# containerized-wordpress-project

Automagically deploy & run containerized WordPress (PHP7 FPM, Nginx, MariaDB) with Let's Encrypt HTTPS encryption using Ansible + Docker. 

This whole process will be completed in <= 5 minutes and doesn't require any Docker or Ansible knowledge!

Blog post discussion: 
* [Automated way of getting Let’s Encrypt certificates for WordPress using Docker + Ansible](http://foolcontrol.org/?p=2758)
* [Automagically deploy & run containerized WordPress (PHP7 FPM, Nginx, MariaDB) using Ansible + Docker on AWS](http://foolcontrol.org/?p=2002)

## Requirements

* Ubuntu Linux instance running in AWS (preferebaly new one, so new that you haven't even SSH-ed to it)
* Ansible installed on (local) host you'll be running this playbook on

## Demo

* [Console output of running "containerized-wordpress" Ansible Playbook](https://s3.eu-central-1.amazonaws.com/adnan-public-images/blog/containerized-wordpress.yml+ansible+playbook+demo.jpg)

* [Accessing WordPress instance created from "containerized-wordpress" Ansible Playbook](https://s3.eu-central-1.amazonaws.com/adnan-public-images/blog/containerized-wordpress.yml+ansible+playbook+demo+results.jpg)

## HowTo: run containerized-wordpress playbook?

Once you have everything that was mentioned in "Requirements" section, this whole process will consists of 3 steps:

#### 1. Get source code for containerized-wordpress-proejct, i.e:

```
git clone https://github.com/AdnanHodzic/containerized-wordpress-project.git
```

 #### 2. Update containerized-wordpress-project/hosts inventory file with AWS instance URL or Public IP, i.e:

```
[aws-wp]
foolcontrol.org
```

#### 3. Install dependency roles

```
sudo ansible-galaxy install -r requirements.yml
```

#### 4. Run containerized-wordpress playbook, using hosts inventory file, i.e:

```
ansible-playbook containerized-wordpress.yml -i hosts
```

After which all you need to do is follow on screen instructions. Process which in <= 5 minutes, host you defined in "hosts" will be fully updated, configured and running containerized WordPress instance.

Please note that default values are defined in square brackets, which you can use by simply hitting enter, i.e:
```
Specify WordPress database name [wordpress]:
```

In this case your WordPress database name will be: "wordpress".

#### 5. Let's Encrypt certificates (HTTPS encryption)

Example of site stage parameter:

```
Is specified site live (DNS is setup)?

Import info: https://goo.gl/XMbnPH [false]:
```

It's strongly recommended to use `false` (default) with your initial deployment to test potential setup. In this case, a self-signed certificate will be created with [Let's Encrypt's staging environemnt](https://letsencrypt.org/docs/staging-environment/).

Only use `true` if DNS is setup and propagated for the specified domain name. In this case, an actual Let's Encrypt certificate will be registered and in case of failure you may hit rate limit for your domain! For more information, please see [Let's Encrypt Rate Limit](https://letsencrypt.org/docs/rate-limits/)
 
## HowTo: run containerized-wordpress playbook in non interactive mode (parameters)?

If you want to run this playbook in non interactive mode (which is enabled by default) using parametrers, you can do so by:

```
ansible-playbook containerized-wordpress.yml -i hosts --extra-vars "domain=custom.domain2.com wp_version=4.9.4 wp_db_name=wpdb wp_db_tb_pre=wp_ wp_db_host=mysql wp_db_psw=change-M3"
```

## Technical rationale/What is this sorcery?

This project consists of single Ansible playbook, which when run consits of 3 (fully automated) steps.

#### Step 1: Setup local environment to run all necessary roles

It will create roles/ directory inside of containerized-wordpress-project/

#### Step 2: Install roles from requirements.yml to roles/

Roles it will install are:

#### [AdnanHodzic.python-ubuntu-bootstrap](https://galaxy.ansible.com/AdnanHodzic/python-ubuntu-bootstrap/)

This Ansible role will install Python on newly bootstrapped host. This is usually a new host which you never even SSH-ed to. In order for Ansible to work, Python must be installed (if missing).

#### [AdnanHodzic.system-upgrade](https://galaxy.ansible.com/AdnanHodzic/system-upgrade/)

This Ansible role will perform upgrade of all software packages on Ubunty host. After which it will reboot host (only if required). If reboot was performed, it'll wait until host is back-up.

* Update APT cache
* Check if there are any available updates
* Perform upgrade of all packages to the latest version (dist)
* Check if a reboot is required, if it is reboot the host/server
* Wait for server to come back after reboot, and report once it's back-up and running.

#### [AdnanHodzic.docker-compose](https://galaxy.ansible.com/AdnanHodzic/docker-compose/)

This Ansible role will perform all necessary tasks to setup and run Docker and Docker Compose:

* Install packages necessary for APT to use a repository over HTTPS.
* Add and setup official Docker APT repositories.
* Install packages needed for AUFS storage drivers.
* Add user to Docker group.

#### [AdnanHodzic.containerized-wordpress](https://galaxy.ansible.com/AdnanHodzic/containerized-wordpress/)

This Ansible playbook will Deploy & run Docker Compose project for WordPress instance. Which consists of 3 separate containers running:
* WordPress (PHP7 FPM)
* Nginx (enabled with Let's Encrpt HTTPS encryption)
* MariaDB

## Interactive Docker images configuration and deployment

Once run, this (containerized-wordpress) playbook will guide you through interactive setup of all 3 containers, after which it will run all above mentioned Ansible roles. End result is that host you have never even SSH-ed to will be fully configured and running containerized WordPress image out of box.

## Troubleshooting

**Q:** In case of host reboot, will all services and Docker images start automatically on boot?

**A:** Yes

***

**Q:** Are Let's Encrypt certificates automatically renewed?

**A:** Yes


If you have any issues or questions, please feel free to submit an issue.
