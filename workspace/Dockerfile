FROM laradock/workspace:2.0-72

LABEL name="ouranoshong"
LABEL email="ouranoshong@outlook.com"

# Remove Faillog and Lastlog to reduce the size of the final image.
RUN rm /var/log/lastlog /var/log/faillog

#####################################
# Non-Root User:
#####################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ARG PGID=1000

ENV PUID ${PUID}
ENV PGID ${PGID}

RUN groupadd -g ${PGID} symdock && \
    useradd -u ${PUID} -g symdock -m symdock


#####################################
# IMAP:
#####################################

ARG INSTALL_IMAP=false
ENV INSTALL_IMAP ${INSTALL_IMAP}

RUN if [ ${INSTALL_IMAP} = true ]; then \
    apt-get update -yqq && \
    apt-get install -y php7.2-imap \
;fi

#####################################
# Set Timezone
#####################################

ARG TZ=Asia/Shanghai
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

#####################################
# Composer:
#####################################

# Add the composer.json
COPY ./composer.json /home/symdock/.composer/composer.json

# Make sure that ~/.composer belongs to symdock
RUN chown -R symdock:symdock /home/symdock/.composer
USER symdock

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}
RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    composer global install \
;fi

#####################################
# Crontab
#####################################
USER root

COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

#####################################
# User Aliases
#####################################

USER symdock
COPY ./aliases.sh /home/symdock/aliases.sh
RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source /home/symdock/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc && \
	sed -i 's/\r//' /home/symdock/aliases.sh && \
	sed -i 's/^#! \/bin\/sh/#! \/bin\/bash/' /home/symdock/aliases.sh

USER root
RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source /home/symdock/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc && \
	sed -i 's/\r//' /home/symdock/aliases.sh && \
	sed -i 's/^#! \/bin\/sh/#! \/bin\/bash/' /home/symdock/aliases.sh

#####################################
# xDebug:
#####################################

ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    apt-get update && \
    apt-get install -y --force-yes php7.2-xdebug && \
    sed -i 's/^;//g' /etc/php/7.2/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/vendor/bin/phpunit'" >> ~/.bashrc \
;fi
# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/7.2/cli/conf.d/xdebug.ini

#####################################
# ssh:
#####################################
ARG INSTALL_WORKSPACE_SSH=false
ENV INSTALL_WORKSPACE_SSH ${INSTALL_WORKSPACE_SSH}

ADD insecure_id_rsa /tmp/id_rsa
ADD insecure_id_rsa.pub /tmp/id_rsa.pub

RUN if [ ${INSTALL_WORKSPACE_SSH} = true ]; then \
    rm -f /etc/service/sshd/down && \
    cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
        && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
        && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
        && rm -f /tmp/id_rsa* \
        && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa \
    && cp -rf /root/.ssh /home/symdock \
    && chown -R symdock:symdock /home/symdock/.ssh \
;fi

#####################################
# MongoDB:
#####################################

# Check if Mongo needs to be installed
ARG INSTALL_MONGO=false
ENV INSTALL_MONGO ${INSTALL_MONGO}
RUN if [ ${INSTALL_MONGO} = true ]; then \
    pecl -q install mongodb && \
    echo "extension=mongodb.so" >> /etc/php/7.2/mods-available/mongodb.ini && \
    ln -s /etc/php/7.2/mods-available/mongodb.ini /etc/php/7.2/cli/conf.d/30-mongodb.ini \
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
# PHP REDIS EXTENSION FOR PHP 7.2
#####################################

ARG INSTALL_PHPREDIS=false
ENV INSTALL_PHPREDIS ${INSTALL_PHPREDIS}
RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    printf "\n" | pecl -q install -o -f redis && \
    echo "extension=redis.so" >> /etc/php/7.2/mods-available/redis.ini && \
    phpenmod redis \
;fi

#####################################
# Swoole EXTENSION FOR PHP 7
#####################################

ARG INSTALL_SWOOLE=false
RUN if [ ${INSTALL_SWOOLE} = true ]; then \
    pecl -q install swoole && \
    echo "extension=swoole.so" >> /etc/php/7.2/mods-available/swoole.ini && \
    ln -s /etc/php/7.2/mods-available/swoole.ini /etc/php/7.2/cli/conf.d/20-swoole.ini \
;fi


#####################################
# Non-root user : PHPUnit path
#####################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER symdock

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

#####################################
# Linuxbrew:
#####################################
USER root

ARG INSTALL_LINUXBREW=false
ENV INSTALL_LINUXBREW ${INSTALL_LINUXBREW}

RUN if [ ${INSTALL_LINUXBREW} = true ]; then \
    apt-get upgrade -y && \
    apt-get install -y build-essential make cmake scons curl git \
      ruby autoconf automake autoconf-archive \
      gettext libtool flex bison \
      libbz2-dev libcurl4-openssl-dev \
      libexpat-dev libncurses-dev && \
    git clone --depth=1 https://github.com/Homebrew/linuxbrew.git ~/.linuxbrew && \
    echo "" >> ~/.bashrc && \
    echo 'export PKG_CONFIG_PATH"=/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig:/usr/lib64/pkgconfig:/usr/lib/pkgconfig:/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/lib64/pkgconfig:/usr/share/pkgconfig:$PKG_CONFIG_PATH"' >> ~/.bashrc && \
    echo 'export LINUXBREWHOME="$HOME/.linuxbrew"' >> ~/.bashrc && \
    echo 'export PATH="$LINUXBREWHOME/bin:$PATH"' >> ~/.bashrc && \
    echo 'export MANPATH="$LINUXBREWHOME/man:$MANPATH"' >> ~/.bashrc && \
    echo 'export PKG_CONFIG_PATH="$LINUXBREWHOME/lib64/pkgconfig:$LINUXBREWHOME/lib/pkgconfig:$PKG_CONFIG_PATH"' >> ~/.bashrc && \
    echo 'export LD_LIBRARY_PATH="$LINUXBREWHOME/lib64:$LINUXBREWHOME/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc \
;fi

#####################################
# Image optimizers:
#####################################
USER root
ARG INSTALL_IMAGE_OPTIMIZERS=false
ENV INSTALL_IMAGE_OPTIMIZERS ${INSTALL_IMAGE_OPTIMIZERS}
RUN if [ ${INSTALL_IMAGE_OPTIMIZERS} = true ]; then \
    apt-get install -y --force-yes jpegoptim optipng pngquant gifsicle && \
    if [ ${INSTALL_NODE} = true ]; then \
        . ~/.bashrc && npm install -g svgo \
    ;fi\
;fi

USER symdock

#####################################
# ImageMagick:
#####################################
USER root
ARG INSTALL_IMAGEMAGICK=false
ENV INSTALL_IMAGEMAGICK ${INSTALL_IMAGEMAGICK}
RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -y --force-yes imagemagick php-imagick \
;fi


#####################################
# Check PHP version:
#####################################

RUN php -v | head -n 1 | grep -q "PHP 7.2."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

# Clean up
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set default work directory
WORKDIR /var/www
