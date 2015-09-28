# Getting started with Docker and Sugar #

Many years after a blog post about how to use PaaS DotCloud (http://developer.sugarcrm.com/2011/08/15/howto-install-sugarcrm-on-cloud-paas-dotcloud/) it's time to continue with the same company and to speak a little bit about Docker.

It's one of the hottest topics in the DevOps world since for two years and most of people in the Sugar community probably already use Docker as their development platform. We will present a quick intro about how to start to use Docker with Sugar. You can find also multiples great resources to setup your development environment by others ways on the support portal http://support.sugarcrm.com/02_Documentation/04_Sugar_Developer/Setting_Up_Development_Environments/ or on the web page of stalwart Sugar Community member like Enrico Simonetti http://enricosimonetti.com/sugar-7-setup-development-environment/ or Jeff Bickart https://www.youtube.com/watch?v=4hioCaOlZO8

## Quick intro to Docker & Sugar  ##

> Docker allows you to package an application with all of its dependencies into a standardized unit for software development.
>
> *https://www.docker.com/whatisdocker*

Docker uses container technology to provide you a quick and easy way to launch an application with its stack of execution. 

Sugar needs at least a web server with PHP, a database server and an search server, following the recommended stack described here http://support.sugarcrm.com/05_Resources/03_Supported_Platforms/ we will consider for the rest of this article that the web server will be Apache with PHP 5.4, the database server will be Percona Server 5.6 and the search server will be Elasticsearch 1.4.

The best way to set up docker on your development environment is to follow the official document available here http://docs.docker.com/linux/started/ after that you also need to install Docker Compose http://docs.docker.com/compose/ which is a tool for launching multiple containers by a single description file usually named docker-compose.yml 

## Build containers ##

Sugar needs a set of PHP modules to be fully functional, by consequence we can't use the official docker image. We need to build our own images, to do that we create a docker file based on the official docker image then we add the right modules and we push a set of PHP settings : 

```bash
FROM php:5.4-apache
MAINTAINER cedric.mourizard@gmail.com

RUN    apt-get -qqy update \
    && apt-get install -y libpng12-dev libjpeg-dev \
    && apt-get -y install re2c libmcrypt-dev \
    && apt-get -y install zlib1g-dev \
    && apt-get -y install libssl-dev libc-client2007e-dev libkrb5-dev \
    && apt-get -y install libcurl4-gnutls-dev \
    && apt-get -y install libxml2-dev libxslt-dev \
    && apt-get -y install libssl-dev \
    && apt-get -y install libcurl4-openssl-dev

RUN    docker-php-ext-install bcmath \
    && docker-php-ext-configure gd --with-jpeg-dir=/usr/lib \
    && docker-php-ext-install gd \
    && docker-php-ext-configure imap --with-imap-ssl --with-kerberos \
    && docker-php-ext-install imap \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install mcrypt \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install zip 

COPY config/crm.php.ini /usr/local/etc/php/conf.d/

RUN a2enmod headers expires deflate rewrite

EXPOSE 80

VOLUME ["/var/www"]
```

For the database and search server we can start with the official image so we can create our docker-compose.yml to describe our full stack:

```bash
web:
    container_name: web
    build: ./images/php5.4-apache
    ports:
        - 80:80
    links:
        - db
        - search        
    volumes:
        - /Users/cedric/sugar-docker-example/data/web:/var/www/html
db:
    container_name: db
    image: percona:5.6
    environment:
        - MYSQL_ROOT_PASSWORD=root
    ports:
        - 3306:3306
search:
    container_name: search
    image: elasticsearch:1.4
    ports:
        - 9200:9200
        - 9300:9300
```

We can see in this file:
* Our 3 applications
* The web server has LINKS to the database server and the search server (https://docs.docker.com/compose/yml/#links)
* Each application use a shared directory with the VOLUME setting. It allows you to keep the code visible on your computer and share it with your containers (https://docs.docker.com/compose/yml/#volumes)
* All of them expose PORTS to provide an easy way to plug your own tools on each piece of the stack. By example you will be able to connect your MySql Workbench to browse your database server (https://docs.docker.com/compose/yml/#ports)

## Launch the platform and install Sugar ##
Now we can launch the platform and start all containers in background:

```bash
$ docker-compose up -d
.
.
Starting search...
Starting db...
Starting web...
```

The first launch will be a little bit long because docker retrieves and build all necessary data to setup the platform. This is only for the first time that you launch the docker-compose command and each time that you will modify your stack.

Now you stack is launched and you can go to your sugar with your browser: http://localhost/sugarpro760/ by example or http://DOCKER_MACHINE_IP/sugarpro760/.
During the install process you can use the NAME setting used in the docker-compose.yml file each time that is necessary. By example the database host name is "db" and the Elasticsearch host name is "search".

That's it, you’re done!

## Few more things ##

If you want to run a PHP Cli script like cron.php by example you can go inside your web container and obtain the bash prompt with this command:
```bash
$ docker exec -it web bash
root@xxxxx:/var/www/html# php -f sugarpro710/cron.php
```

For Windows users, docker-compose is not yet available as a native tool however Docker solves this problem: you can use a container to launch the docker-compose tool with this docker image by example: https://hub.docker.com/r/dduportal/docker-compose/

## Conclusion ##

Now you can start to play with docker and Sugar to:
* develop your customizations
* try the Sugar 7 unit test suite by example
* try advanced stack with multiple database servers by example
* ...

This article provides an introduction to set up Sugar on docker. You must adapt all of these steps to your own context consider this for development platform and not for production platform where you need to take care a lot about performance and security aspects which aren't addressed in this post.
