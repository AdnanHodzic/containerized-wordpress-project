# containerized-wordpress-project

_If you're interested in running WordPress on Kubernetes please refer to my [wp-k8s project](https://github.com/AdnanHodzic/wp-k8s)._

Automagically deploy & run containerized WordPress with Let's Encrypt HTTPS encryption using Ansible + Docker.

This whole process will be completed in <= 5 minutes and doesn't require any Docker or Ansible knowledge!

Supported platforms:
* Ubuntu
* Debian
* CentOS
* RedHat

Blog post discussion: 
* [Automated way of getting Let’s Encrypt certificates for WordPress using Docker + Ansible](http://foolcontrol.org/?p=2758)
* [Automagically deploy & run containerized WordPress (PHP7 FPM, Nginx, MariaDB) using Ansible + Docker on AWS](http://foolcontrol.org/?p=2002)

## Requirements

* Ubuntu/Debian or CentOS/RedHat Linux instance (preferebaly new one, so new that you haven't even SSH-ed to it).
* Port 80 (HTTP) and 443 (HTTPS) must be enabled on target Linux instance.
* Target linux instance should have >= 1G of RAM (due to MySQL requirements).
* Ansible installed on (local) host where you'll be running this playbook on (preferably with Ansible version => 2.7)

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
ansible-galaxy install -r requirements.yml
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

* [Console output of running "containerized-wordpress" Ansible Playbook](http://foolcontrol.org/wp-content/uploads/2018/03/containerized-wordpress-run-with-lets-encrypt-2.png)

#### 5. Let's Encrypt certificates (HTTPS encryption)

Example of site stage parameter:

```
Is specified site live (DNS is setup)?

Import info: https://goo.gl/XMbnPH [staging]:
```

It's strongly recommended to use `staging` (default) with your initial deployment to test potential setup. In this case, a self-signed certificate will be created with [Let's Encrypt's staging environemnt](https://letsencrypt.org/docs/staging-environment/).

* [Demo of test instance running with `staging` argument](https://foolcontrol.org/wp-content/uploads/2018/03/stage-false-test3.png)

Only use `production` if DNS is setup and propagated for the specified domain name. In this case, an actual Let's Encrypt certificate will be registered and in case of failure you may hit rate limit for your domain! For more information, please see [Let's Encrypt Rate Limit](https://letsencrypt.org/docs/rate-limits/)

* [Demo of test instance running with `production` argument](https://foolcontrol.org/wp-content/uploads/2018/03/stage-live-test3.png)
 
## HowTo: run containerized-wordpress playbook in non interactive mode (parameters)?

If you want to run this playbook in non interactive mode (which is enabled by default) using parameters, you can do so by running i.e:

```
ansible-playbook containerized-wordpress.yml -i hosts --extra-vars \
"distribution=1 system_user=ubuntu domain=custom.domain2.com stage=staging 
wp_version=5.2.3 wp_db_user=admin wp_db_psw=change-M3 db_root_psw=change-M3 
wp_db_name=wpdb wp_db_tb_pre=wp_ wp_db_host=mysql"
```

## Technical rationale/What is this sorcery?

Once run, this (containerized-wordpress) playbook will guide you through interactive setup of all 3 containers (WordPress, Nginx with Let's Encrypt for HTTPS encryption and MySQL). After which it will run all above mentioned Ansible roles. End result is that host you have never even SSH-ed to will be fully configured and running containerized WordPress image out of box.

#### Step 1: Setup local environment to run all necessary roles

It will create roles/ directory inside of containerized-wordpress-project/

#### Step 2: Install roles from requirements.yml to roles directory (roles/)

Roles it will install are:

#### [AdnanHodzic.python-ubuntu-bootstrap](https://galaxy.ansible.com/AdnanHodzic/python-ubuntu-bootstrap/)

This Ansible role will install Python on newly bootstrapped Ubuntu/Debian host. This is usually a new host which you never even SSH-ed to. In order for Ansible to work, Python must be installed (if missing).

#### [ansible-role-python-centos-bootstrap](https://github.com/iMartzen/ansible-role-python-centos-bootstrap)

This Ansible role will install Python on newly bootstrapped CentOS/RedHat host. This is usually a new host which you never even SSH-ed to. In order for Ansible to work, Python must be installed (if missing).

#### [AdnanHodzic.system-upgrade](https://galaxy.ansible.com/AdnanHodzic/system-upgrade/)

This Ansible role will perform upgrade of all software packages on Ubuntu/Debian host. After which it will reboot host (only if required). If reboot was performed, it'll wait until host is back-up.

* Update APT cache
* Check if there are any available updates
* Perform upgrade of all packages to the latest version (dist)
* Check if a reboot is required, if it is reboot the host/server
* Wait for server to come back after reboot, and report once it's back-up and running.

#### [ansible-role-centos-system-upgrade](https://github.com/iMartzen/ansible-role-centos-system-upgrade)

This Ansible role will perform upgrade of all software packages on CentOS/RedHat host. After which it will reboot host (only if required). If reboot was performed, it'll wait until host is back-up.

* Perform upgrade of all packages to the latest version (dist)
* Check if a reboot is required, if it is reboot the host/server
* Wait for server to come back after reboot, and report once it's back-up and running.


#### [AdnanHodzic.docker-compose](https://galaxy.ansible.com/AdnanHodzic/docker-compose/)

This Ansible role will perform all necessary tasks to setup and run Docker and Docker Compose on Ubuntu/Debian:

* Install packages necessary for APT to use a repository over HTTPS.
* Add and setup official Docker APT repositories.
* Install packages needed for AUFS storage drivers.
* Add user to Docker group.

#### [ansible-role-centos-docker-compose-setup](https://github.com/iMartzen/ansible-role-centos-docker-compose-setup)

This Ansible role will perform all necessary tasks to setup and run Docker and Docker Compose on CentOS/RedHat:

* Install packages necessary for YUM
* Add and setup official Docker YUM repositories.
* Add user to Docker group.
* Start de Docker Daemon and enables it at start up

#### [AdnanHodzic.containerized-wordpress](https://galaxy.ansible.com/AdnanHodzic/containerized-wordpress/)

This Ansible playbook will Deploy & run [Docker Compose project for WordPress instance](https://github.com/AdnanHodzic/ansible-role-containerized-wordpress/blob/master/templates/docker-compose.j2). It will also configure Let's Encrypt certificates for specified domain. It consists of 3 separate (mutually connected) containers running: WordPress, Nginx (Let's Encrypt) and MySQL
* WordPress
* Nginx (enabled with Let's Encypt HTTPS encryption)
* MySQL

## Troubleshooting

**Q:** In case of host reboot, will all services and Docker images start automatically on boot?

**A:** Yes

**Q:** Are Let's Encrypt certificates automatically renewed?

**A:** Yes

**Q:** Are multiple subdomains supported?

**A:** Yes, as part of deployed `docker-compose.yml` file simply extend it to:

```
DOMAINS: foolcontrol.org -> http://wordpress:80, test.foolcontrol.org -> http://wordpress:80'
```

This will allow you to add as many subdomains as necessary.

**Q:** Updating WordPress is requesting FTP connection information, can this be avoided?

**A:** Yes, on deployed host add: 
```
define('FS_METHOD','direct');
```

line to the bottom of `wp-config.php` file, i.e : `~/compose-wordpress/wordpress/wp-config.php`

Once the file is saved, changes are immidate. This will allow you to seamlessly upgrade Wordpress through web interface.

If you have any issues or questions, please feel free to submit an issue.

## Donate

Since I'm working on this project in free time, please consider supporting this project by making a donation of any amount!

##### PayPal
[![paypal](https://www.paypalobjects.com/en_US/NL/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=7AHCP5PU95S4Y&item_name=Contribution+for+work+on+containerized-wordpress-project&currency_code=EUR&source=url)

##### BitCoin
[bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87](bitcoin:bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87)

[![bitcoin](https://foolcontrol.org/wp-content/uploads/2019/08/btc-donate-displaylink-debian.png)](bitcoin:bc1qlncmgdjyqy8pe4gad4k2s6xtyr8f2r3ehrnl87)

