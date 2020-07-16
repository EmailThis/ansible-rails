<h1 align="center">Ansible Rails</h1>
<p align="center">
    <img src="./images/ansible-rails-promo.jpg" alt="Ansible Rails Promo Image" style="max-width:100%;">
<p>

Ansible Rails is a playbook for easily deploying Ruby on Rails applications. It uses Vagrant to provision an environment where you can test your deploys. [Ansistrano](https://github.com/ansistrano/deploy) is used for finally deploying our app to staging and production environments.

While this is meant to work out of the box, you can tweak the files in the `roles` directory in order to satisfy your project-specific requirements. 

> **Shameless plug:** If you're looking for a simple bookmarking tool, try [EmailThis.me](https://www.emailthis.me) - a simpler alternative to Pocket that helps you *save ad-free articles and web pages to your email inbox*.

---

### What does this do?
* Configure our server with some sensible defaults
* Install required/useful packages. See notes below for more details.
* Auto upgrade all installed packages (TODO)
* Create a new deployment user (called 'deploy') with passwordless login
* Prevent root login
* SSH hardening
    * Prevent password login
    * Change the default SSH port
    * Prevent root login
* Setup UFW (firewall)
* Setup Fail2ban
* Install Logrotate
* Setup Nginx with some sensible config (thanks to nginxconfig.io)
* Certbot (for Let's encrypt SSL certificates)
* Ruby (using Rbenv). 
    * Defaults to `2.6.6`. You can change it in the `app-vars.yml` file
    * [jemmaloc](https://github.com/jemalloc/jemalloc) is also installed and configured by default
    * [rbenv-vars](https://github.com/rbenv/rbenv-vars) is also installed by default
* Node.js 
    * Defaults to 12.x. You can change it in the `app-vars.yml` file.
* Yarn
* Redis (latest)
* Postgresql. 
    * Defaults to v12. You can specify the version that you need in the `app-vars.yml` file.
* Puma (with Systemd support for restarting automatically) **See Puma Config section below**
* Sidekiq (with Systemd support for restarting automatically)
* Ansistrano hooks for performing the following tasks - 
    * Installing all our gems
    * Precompiling assets
    * Migrating our database (using `run_once`)

---

### Getting started
Here are the steps that you need to follow in order to get up and running with Ansible Rails. 

#### Step 1. Installation

```
git clone https://github.com/EmailThis/ansible-rails ansible-rails
cd ansible-rails
```

#### Step 2. Configuration
Open `app-vars.yml` and change the following variables. Additionally, please review the `app-vars.yml` and see if there is anything else that you would like to modify (e.g.: install some other packages, change ruby, node or postgresql versions etc.)

```
app_name:           YOUR_APP_NAME   // Replace with name of your app
app_git_repo:       "YOUR_GIT_REPO" // e.g.: github.com/EmailThis/et
app_git_branch:     "master"        // branch that you want to deploy (e.g: 'production')

postgresql_db_user:     "{{ deploy_user }}_postgresql_user"
postgresql_db_password: "{{ vault_postgresql_db_password }}" # from vault (see next section)
postgresql_db_name:     "{{ app_name }}_production"

nginx_https_enabled: false # change to true if you wish to install SSL certificate 
```


#### Step 3. Storing sensitive information
Create a new `vault` file to store sensitive information
```
ansible-vault create group_vars/all/vault.yml
```

Add the following information to this new vault file
```
vault_postgresql_db_password: "XXXXX_SUPER_SECURE_PASS_XXXXX"
vault_rails_master_key: "XXXXX_MASTER_KEY_FOR_RAILS_XXXXX"
```

#### Step 4. Deploy

Now that we have configured everything, lets see if everything is working locally. Run the following command -
```
vagrant up
```
Now open your browser and navigate to 192.168.50.2. You should see your Rails application.

If you don't wish to use Vagrant, clone this repo, modify the `inventories/development.ini` file to suit your needs, and then run the following command
```
ansible-playbook -i inventories/development.ini provision.yml
```

To deploy this app to your production server, create another file inside `inventories` directory called `production.ini` with the following contents. For this, you would need a VPS. I've used [DigitalOcean](https://m.do.co/c/031c76b9c838) and [Vultr](https://www.vultr.com/?ref=8597223) in production for my apps and both these services are top-notch.
```
[web]
192.168.50.2 # replace with IP address of your server.

[all:vars]
ansible_ssh_user=deployer
ansible_python_interpreter=/usr/bin/python3
```

---

### Additional Configuration

####  Installing additional packages
By default, the following packages are installed. You can add/remove packages to this list by changing the `required_package` variable in `app-vars.yml`
```
    - curl
    - ufw
    - fail2ban
    - git-core
    - apt-transport-https
    - ca-certificates
    - software-properties-common
    - python3-pip
    - virtualenv
    - python3-setuptools
    - zlib1g-dev 
    - build-essential 
    - libssl-dev 
    - libreadline-dev 
    - libyaml-dev 
    - libxml2-dev 
    - libxslt1-dev 
    - libcurl4-openssl-dev
    - libffi-dev 
    - dirmngr 
    - gnupg
    - autoconf
    - bison
    - libreadline6-dev
    - libncurses5-dev
    - libgdbm5 
    - libgdbm-dev
    - libpq-dev # postgresql client
    - libjemalloc-dev # jemalloc
```

####  Enable UFW
You can enable UFW by adding the role to `provision.yml` like so - 
```
roles:
    ...
    ...
    - role: ufw
      tags: ufw
```

Then you can set up the UFW rules in `app-vars.yml` like so -
```
ufw_rules:
  - { rule: "allow", proto: "tcp", from: "any", port: "80" }
  - { rule: "allow", proto: "tcp", from: "any", port: "443" }
```

#### Enable Certbot (Let's Encrypt SSL certificates)

Add the role to `provision.yml`
```
roles:
    ...
    ...
    - role: certbot
      tags: certbot
```

Add the following variables to `app-vars.yml`
```
nginx_https_enabled: true

certbot_email: "you@email.me"
certbot_domains:
  - "domain.com"
  - "www.domain.com"
```

#### PostgreSQL Database Backups
By default, daily backup is enabled in the `app-vars.yml` file. In order for this to work, the following variables need to be set. If you do not wish to store backups, remove (or uncomment) these lines from `app-vars.yml`.

```
aws_key: "{{ vault_aws_key }}" # store this in group_vars/all/vault.yml that we created earlier
aws_secret: "{{ vault_aws_secret }}"

postgresql_backup_dir: "{{ deploy_user_path }}/backups"
postgresql_backup_filename_format: >-
  {{ app_name }}-%Y%m%d-%H%M%S.pgdump
postgresql_db_backup_healthcheck: "NOTIFICATION_URL (eg: https://healthcheck.io/)" # optional
postgresql_s3_backup_bucket: "DB_BACKUP_BUCKET" # name of the S3 bucket to store backups
postgresql_s3_backup_hour: "3"
postgresql_s3_backup_minute: "*"
postgresql_s3_backup_delete_after: "7 days" # days after which old backups should be deleted
```


#### Puma config

Your Rails app needs to have a puma config file (usually in `/config/puma.rb`). Here's a sample - 

```
threads_count = ENV.fetch("RAILS_MAX_THREADS") { 5 }
threads threads_count, threads_count

port ENV.fetch("PORT") { 3000 }

rails_env = ENV.fetch("RAILS_ENV") { "development" }
environment rails_env

if %w[production staging].member?(rails_env)
    app_dir = ENV.fetch("APP_DIR") { "YOUR_APP/current" }
    directory app_dir

    shared_dir = ENV.fetch("SHARED_DIR") { "YOUR_APP/shared" }

    # Logging
    stdout_redirect "#{shared_dir}/log/puma.stdout.log", "#{shared_dir}/log/puma.stderr.log", true
    
    pidfile "#{shared_dir}/tmp/pids/puma.pid"
    state_path "#{shared_dir}/tmp/pids/puma.state"
    
    # Set up socket location
    bind "unix://#{shared_dir}/sockets/puma.sock"
    
    workers ENV.fetch("WEB_CONCURRENCY") { 2 }
    preload_app!

elsif rails_env == "development"
    plugin :tmp_restart
end
```

---

### Motivation
I use Heroku to deploy my Rails apps. It makes deployment really easy and I've got no complaints. However, I always wanted to learn how it all works under the hood. Over the last couple of months, I decided to learn more about how to set up a server and deploy a Rails app to production. This project is a consolidation of my learnings.

--- 

### Credits
* [Geerling Guy](https://github.com/geerlingguy) (for his wonderful book on Ansible)
* [dresden-weekly/ansible-rails](https://github.com/dresden-weekly/ansible-rails)

---

### Questions, comments, suggestions?
Please let me know if you run into any issues or if you have any questions. I'd be happy to help. I would also welcome any improvements/suggestions by way of pull requests.


Bharani <br/>
Founder @ [EmailThis.me](https://www.emailthis.me)
