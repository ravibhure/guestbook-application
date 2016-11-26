guestbook-application
=========

* postgresql
* redis
* ruby-install
* nginx + passenger
* guestbook application

### Requirements

Ubuntu 14.04 LTS

## Introduction

Novadays, you can get your own dedicated server up and running in a seconds. Once you get it up, do you really spent several hours to configure it for your application needs ?  Do you really want to repeat the same steps with each new server ?  In this article I will give you an idea on automated installation with Ansible, a Simple IT Automation toolkit and Ubuntu 14.04 LTS server as box OS.

## Background

You would need basic understanding of ansible files syntax. if you did not play with Ansible yet, I would recommend to review some intro articles like [http://docs.ansible.com/ansible/intro.html](http://docs.ansible.com/ansible/intro.html)

## Cooking

We would need to deploy following components:   PostgreSQL, Redis, Ruby, Web server with Passenger, your application itself. For purposes of the demo, we will install well known starter guestbook [https://github.com/askcharlie/guestbook.git](https://github.com/askcharlie/guestbook.git)

### PostgreSQL

Install PostgreSQL (version of your choice), create user and database.

```
    - role: postgresql
      postgresql_version: 9.5
      postgresql_databases:
        - name: guestbook
          owner: charlie
      postgresql_users:
        - name: charlie
          pass: charlie
      postgresql_user_privileges:
        - name: charlie
          db: guestbook
```


### Redis

Install Redis.

```
    - role: redis
      redis_bind_interface: 127.0.0.1   # Set the interface to 0.0.0.0 to listen on all interfaces.
```

### Ruby

Now it is time to install Ruby.  Small comment here: if you deploy saying on shared server, you most likely would like to have an ability to have multiple ruby versions and switch between them. From other hand, if you deploy your application to the dedicated host - usually I also replace default system ruby with the same ruby version.

```
    - role: ruby
      rvm1_rubies: 
        - 'ruby-2.3.0'
      rvm1_user: root
      rvm1_install_flags: '--auto-dotfiles'
      rvm1_install_path: /usr/local/rvm
```

With tools above, ruby installation recipe is compact & clear:

### Webserver & passenger

Thanks to Phusion Passenger team, they did a great job to provide pre-built binaries for most of the popular platforms and configurations at [https://oss-binaries.phusionpassenger.com/](https://oss-binaries.phusionpassenger.com/). This allows us to skip steps of compiling phusion passengers from source, recomplining  webserver, etc  & use pre-built binary instead.

I prefer Nginx over classic Apache, thus we will install pre-build Nginx with passenger:

```
    - role: application
      option_install_nginx_passenger: true
      app_repository: https://github.com/askcharlie/guestbook.git
      app_redis_host: localhost
      app_db_host: localhost
      app_db_user: charlie
      app_db_password: charlie
      app_db_name: guestbook
```

Everything looks good. :-)

## Application setup itself

Let's define application parameters: in particular: required OS packages to build gems, application environment parameters, redis and database connection details.

## Running the code

Let's execute provisioning & test it

```
ansible-playbook site.yml
```

So, depending on network speed, you have your application installed.

Let's check by ip address:

![Deployed application](https://raw.githubusercontent.com/ravibhure/guestbook/master/guestbook.png)

## Points of Interest

Now you aware of another way to deploy your ruby applications.
