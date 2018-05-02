ARG PHP_VERSION={PHP_VERSION}
FROM laradock/php-fpm:2.2-${PHP_VERSION}

LABEL name="ouranoshong"
LABEL email="ouranoshong@outlook.com"

#####################################
# xDebug:
#####################################

ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    pecl install xdebug && \
    docker-php-ext-enable xdebug \
;fi

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini


#####################################
# PHP REDIS EXTENSION FOR PHP 7.0
#####################################

ARG INSTALL_PHPREDIS=false
RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    printf "\n" | pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis \
;fi

#####################################
# Swoole EXTENSION FOR PHP 7
#####################################

ARG INSTALL_SWOOLE=false
RUN if [ ${INSTALL_SWOOLE} = true ]; then \
    pecl install swoole \
    &&  docker-php-ext-enable swoole \
;fi

#####################################
# MongoDB:
#####################################

ARG INSTALL_MONGO=false
RUN if [ ${INSTALL_MONGO} = true ]; then \
    pecl install mongodb && \
    docker-php-ext-enable mongodb \
;fi

#####################################
# AMQP:
#####################################

ARG INSTALL_AMQP=false
RUN if [ ${INSTALL_AMQP} = true ]; then \
    apt-get update && \
    apt-get install librabbitmq-dev -y && \
    pecl install amqp && \
    docker-php-ext-enable amqp \
;fi

#####################################
# ZipArchive:
#####################################

ARG INSTALL_ZIP_ARCHIVE=false
RUN if [ ${INSTALL_ZIP_ARCHIVE} = true ]; then \
    docker-php-ext-install zip \
;fi

#####################################
# bcmath:
#####################################

ARG INSTALL_BCMATH=false
RUN if [ ${INSTALL_BCMATH} = true ]; then \
    docker-php-ext-install bcmath \
;fi

#####################################
# GMP (GNU Multiple Precision):
#####################################

ARG INSTALL_GMP=false
RUN if [ ${INSTALL_GMP} = true ]; then \
	apt-get update -yqq && \
	apt-get install -y libgmp-dev && \
    docker-php-ext-install gmp \
;fi

#####################################
# Exif:
#####################################

ARG INSTALL_EXIF=false
RUN if [ ${INSTALL_EXIF} = true ]; then \
    docker-php-ext-install exif \
;fi

#####################################
# Opcache:
#####################################

ARG INSTALL_OPCACHE=false
RUN if [ ${INSTALL_OPCACHE} = true ]; then \
    docker-php-ext-install opcache \
;fi

# Copy opcache configration
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini


#####################################
# Tokenizer Modifications:
#####################################

ARG INSTALL_TOKENIZER=false
RUN if [ ${INSTALL_TOKENIZER} = true ]; then \
    docker-php-ext-install tokenizer \
;fi

#####################################
# Human Language and Character Encoding Support:
#####################################

ARG INSTALL_INTL=false
RUN if [ ${INSTALL_INTL} = true ]; then \
    apt-get update -yqq && \
    apt-get install -y zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl \
;fi

#####################################
# Image optimizers:
#####################################
USER root
ARG INSTALL_IMAGE_OPTIMIZERS=false
ENV INSTALL_IMAGE_OPTIMIZERS ${INSTALL_IMAGE_OPTIMIZERS}
RUN if [ ${INSTALL_IMAGE_OPTIMIZERS} = true ]; then \
    apt-get update -yqq && \
    apt-get install -y --force-yes jpegoptim optipng pngquant gifsicle \
;fi

#####################################
# ImageMagick:
#####################################
USER root
ARG INSTALL_IMAGEMAGICK=false
ENV INSTALL_IMAGEMAGICK ${INSTALL_IMAGEMAGICK}
RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get update -y && \
    apt-get install -y libmagickwand-dev imagemagick && \
    pecl install imagick && \
    docker-php-ext-enable imagick \
;fi

#####################################
# IMAP:
#####################################
ARG INSTALL_IMAP=false
ENV INSTALL_IMAP ${INSTALL_IMAP}
RUN if [ ${INSTALL_IMAP} = true ]; then \
    apt-get update && \
    apt-get install -y libc-client-dev libkrb5-dev && \
    rm -r /var/lib/apt/lists/* && \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-install imap \
;fi

#####################################
# Check PHP version:
#####################################

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

ADD ./symfony.ini /usr/local/etc/php/conf.d
ADD ./symfony.pool.conf /usr/local/etc/php-fpm.d/

#RUN rm -r /var/lib/apt/lists/*

RUN usermod -u 1000 www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000