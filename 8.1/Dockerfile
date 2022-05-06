#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

FROM php:8.1-fpm

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

#
#--------------------------------------------------------------------------
# Software's Installation
#--------------------------------------------------------------------------
#
# Installing tools and PHP extentions using "apt", "docker-php", "pecl",
#

# Install "curl", "libmemcached-dev", "libpq-dev", "libjpeg-dev",
#         "libpng-dev", "libfreetype6-dev", "libssl-dev", "libmcrypt-dev",
RUN set -eux; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    apt-get upgrade -y && \
    apt-get install -yqq --no-install-recommends \
            apt-utils \
            gnupg2 \
            git \
            curl \
            libicu-dev \
            libmemcached-dev \
            libz-dev \
            libpq-dev \
            libjpeg-dev \
            libpng-dev \
            libfreetype6-dev \
            libssl-dev \
            libwebp-dev \
            libxpm-dev \
            libmcrypt-dev \
            libzip-dev zip unzip \
            libonig-dev; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    # Install the zip extension
    docker-php-ext-configure zip && \
    docker-php-ext-install zip && \
    php -m | grep -q 'zip'; \
    # Install the intl extension
    docker-php-ext-configure intl && \
    docker-php-ext-install intl; \
    # Install the PHP pdo_mysql extention
    docker-php-ext-install pdo_mysql; \
    # Install the PHP pdo_pgsql extention
    docker-php-ext-install pdo_pgsql; \
    # bcmath
    docker-php-ext-install bcmath; \
    # Opcache
    docker-php-ext-install opcache; \
    # Install PGSQL
    docker-php-ext-install pgsql; \
    # Install the PHP gd library
    docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-webp \
            --with-xpm \
            --with-freetype; \
    docker-php-ext-install gd; \
    php -r 'var_dump(gd_info());'

###########################################################################
# pgsql client
###########################################################################

RUN apt-get install -yqq gnupg \
    && . /etc/os-release \
    && echo "deb http://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && curl -sL https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
    && apt-get update -yqq \
    && apt-get install -yqq postgresql-client-12 postgis \
    && apt-get install -yqq postgis \
    && apt-get purge -yqq gnupg;


###########################################################################
# PHP REDIS EXTENSION
###########################################################################
RUN pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis;

###########################################################################
# AMQP:
###########################################################################
RUN set -eux && \
      # # Install the amqp extension
      apt-get -yqq install librabbitmq-dev && \
      pecl install amqp-1.11.0beta && \
      docker-php-ext-enable amqp && \
      php -m | grep -oiE '^amqp$';

###########################################################################
# Image optimizers:
###########################################################################

USER root
RUN apt-get install -yqq jpegoptim optipng pngquant gifsicle;


###########################################################################
# ImageMagick:
###########################################################################

USER root
RUN apt-get install -yqq libmagickwand-dev imagemagick && \
      cd /tmp && \
      git clone https://github.com/Imagick/imagick && \
      cd imagick && \
      phpize && \
      ./configure && \
      make && \
      make install && \
      rm -r /tmp/imagick && \
    docker-php-ext-enable imagick; \
    php -m | grep -q 'imagick';

###########################################################################
# YAML:
###########################################################################

USER root
RUN apt-get install -yqq libyaml-dev \
    && echo '' | pecl install yaml \
    && docker-php-ext-enable yaml;


###########################################################################
# PHP DECIMAL:
###########################################################################

USER root
RUN apt-get install -yqq libmpdec-dev \
          && pecl install decimal \
          && docker-php-ext-enable decimal \
          && php -m | grep -q 'decimal';

###########################################################################
# COMPOSER
###########################################################################
RUN curl -sS https://getcomposer.org/installer | php -- --version 2.0.9 --install-dir=/usr/local/bin --filename=composer;

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog