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
