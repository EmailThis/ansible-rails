# Ansible Rails 

This project is meant to be a starting point for developing and deploying Ruby on Rails applications using Ansible. It uses Vagrant to provision a development environment. Ansistrano is used for deploying our code to staging and production environments.

While this is meant to work out of the box, you can tweak the files in the `roles` directory in order to satisfy your project specific requirements. 

### What's included
* Sensible defaults
* Installation of common packages
* Auto upgrade of all installed packages (TODO)
* Set up a 'deploy' user with passwordless login
* Prevent root login
* SSH hardening
    * Prevent password login
    * Change the default SSH port
* Setup UFW firewall
* Setup Fail2ban
* Logrotate (TODO)
* Nginx with some sensible defaults (thanks to nginxconfig.io)
* Certbot (for Let's encrypt SSL certificates)
* Ruby (using Rbenv). Defaults to `2.6.6`. You can change it in the `vars.yml` file
    * jemmaloc is also installed and configured by default
    * rbenv-vars is also installed by default
* Node.js - defaults to 12.x. You can change it in the `vars.yml` file.
* Yarn
* Redis (latest)
* Postgresql. Defaults to 9.6. You can specify the version that you need in the `vars.yml` file.
* Puma (with Systemd support for restarting automatically)
* Sidekiq (with Systemd support for restarting automatically)
* Ansistrano hooks for performing the following tasks
    * Installing all our gems
    * Precompiling assets
    * Migrating our database (using `run_once`)


### Getting started
Here are the steps that you need to follow in order to get a development environment up and running. Please note that you need to have Vagrant installed for this to work.
```
git clone https://github.com/bharani91/ansible-rails ansible-rails
cd ansible-rails
vagrant up
```
Now open your browser and navigate to 192.168.50.2. You should see a bare-bones Ruby on Rails application. 

If you don't wish to use Vagrant, clone this repo, change the hosts in the `inventories/development.ini` and then run the following command
```
ansible-playbook -i inventories/developmen.ini provision.yml
```


### Questions, comments, suggestions?
Please let me know if you run into any issues or if you have any questions. I'd be happy to help. I would also welcome any improvements/suggestions by way of pull requests.

Thanks,
Bharani
Maker @ [EmailThis.me](https://www.emailthis.me)