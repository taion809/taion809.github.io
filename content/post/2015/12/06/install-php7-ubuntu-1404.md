---
date: 2015-12-06T14:24:50-05:00
draft: false
title: Install PHP7
description: PHP7 was released this month, exciting! So how do you get ahold of it?
tags: [ "php" ]
Section: blog
categories:
  - "Development"
slug: install-php7
---

[PHP7 was released this month](http://php.net/archive/2015.php#id2015-12-03-1), exciting! So how do you get ahold of it?

<!--more-->

There are a couple of options available depending on what operating system you are running.

## Automated Install
### PuPHPet
The easiest way to get going with php7 is to head over to [puphpet.com](https://puphpet.com) and configure a build.  Unfortunately at the time of this writing the only available operating system is Ubuntu 14.04.

## Manual
If you wish to install php7 yourself there are more options.

### Ubuntu
Ondřej Surý has provided [a great PPA](https://launchpad.net/~ondrej/+archive/ubuntu/php-7.0) for ubuntu since php5, it's easy to use and provides good default configurations.  At the time of this writing Mr. Surý's PPA supports Ubuntu >= 14.04.

```bash
sudo apt-add-repository ppa:ondrej/php-7.0
sudo apt-get update
sudo apt-get install php7.0
```

### CentOS
With CentOS >= 6.5 there are a couple of additional places to find PHP7.

#### Remi
PHP7 with extensions is available on the popular [remi yum repository](http://rpms.famillecollet.com/).

##### Example for CentOS 7
```
sudo rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo yum install --enablerepo=remi --enablerepo=remi-php70 php
```

#### Webtatic
[Webtatic](https://webtatic.com/packages/php70/) has PHP7 available for install for CentOS >= 6.7

##### Example for CentOS 7
```
sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
sudo yum install --enablerepo=remi --enablerepo=remi-php70 php
```

## Example Configuration
The following example configurations setup php7 fpm, apache 2.4, and [lumen](http://lumen.laravel.com).

#### Install PHP7
```bash
sudo apt-add-repository ppa:ondrej/php-7.0
sudo apt-get update
sudo apt-get install php7.0-fpm php7.0-common php7.0-intl
```

#### Composer
You need [composer](https://getcomposer.org/download) to get started.

```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

#### Lumen
Lumen is a "microframework" provided by the Laravel project.

```bash
composer create-project laravel/lumen /var/www/app --prefer-dist
```

#### php-fpm pool
```ini
listen = 127.0.0.1:9000
```

#### Apache vhost
Additional options and steps can be found [here](https://wiki.apache.org/httpd/PHP-FPM).

```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/app/public
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        <Directory "/var/www/app/public">
                Order allow,deny
                Allow from all
                Require all granted
        </Directory>

        ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/var/www/app/public/$1
        DirectoryIndex /index.php index.php
</VirtualHost>
```

#### Vagrantfile
This is a sample vagrantfile I used to do the above.

```ruby
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "forwarded_port", guest: 80, host: 5555
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.synced_folder "./app", "/var/www/app", type: "nfs"
  config.vm.provider "virtualbox" do |vb|
   vb.memory = "1024"
  end
end
```

I hope it helps :)
