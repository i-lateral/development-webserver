FROM php:7.4-apache-buster
LABEL author="Al Twohill <al@twohill.nz>"

# Remove and re-create www-data users and map to local system ID
ARG USER_ID
ARG GROUP_ID

RUN if [ ${USER_ID:-0} -ne 0 ] && [ ${GROUP_ID:-0} -ne 0 ]; then \
    userdel -f www-data &&\
    if getent group www-data ; then groupdel www-data; fi &&\
    groupadd -g ${GROUP_ID} www-data &&\
    useradd -l -u ${USER_ID} -g www-data www-data &&\
    install -d -m 0755 -o www-data -g www-data /home/www-data &&\
    chown --changes --silent --no-dereference --recursive \
          --from=33:33 ${USER_ID}:${GROUP_ID} \
        /home/www-data \
;fi

# Install components
RUN apt-get update -y && apt-get install -y \
        net-tools \
		curl \
		g++ \
		git-core \
		gzip \
		less \
		libcurl4-openssl-dev \
		libgd-dev \
		libjpeg-dev \
		libpng-dev \
		libfreetype6-dev \
		libldap2-dev \
		libicu-dev \
		libmagickwand-dev \
		libmcrypt-dev \
		libtidy-dev \
		libxslt-dev \
		libzip-dev \
		nano \
		openssh-client \
		unzip \
		xfonts-75dpi \
		xfonts-base \
		wget \
		zip \
		zlib1g-dev \
		mariadb-client \
	--no-install-recommends && \
	curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer && \
	curl -sS http://silverstripe.github.io/sspak/install | php -- /usr/local/bin && \
	pecl install imagick-3.4.3 && \
	apt-get autoremove -y && \
	rm -r /var/lib/apt/lists/* && \
	cd /root && \
    wget -q https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.stretch_amd64.deb && \
    dpkg -i wkhtmltox_0.12.6-1.stretch_amd64.deb

# Install PHP Extensions
RUN docker-php-ext-configure intl && \
	docker-php-ext-configure mysqli --with-mysqli=mysqlnd && \
	docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
	docker-php-ext-configure gd --with-jpeg && \
	docker-php-ext-enable imagick && \
	docker-php-ext-install -j$(nproc) \
		intl \
		gd \
		ldap \
		mysqli \
		pdo \
		pdo_mysql \
		soap \
		tidy \
		xsl \
		zip

# Apache configuration
RUN a2enmod rewrite expires remoteip cgid
COPY vhost.conf /etc/apache2/sites-available/000-default.conf
RUN echo "ServerName localhost" > /etc/apache2/conf-available/fqdn.conf

COPY php-custom.ini /usr/local/etc/php/conf.d/custom.ini

# Install and configure xdebug must be 2.9.x for settings to work
RUN pecl install -f xdebug-2.9.8 && docker-php-ext-enable xdebug
COPY php-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Run and configure sendmail (for email testing)
# Thanks to: https://r.je/sendmail-php-docker for the info
RUN apt-get update -y && apt-get install -y \
	mailutils \
	sendmail \
	sendmail-bin \
	sensible-mda

# Set up entrypoint enabling host.docker.internal for Linux
COPY entrypoint.sh /usr/bin/entrypoint.sh
RUN chmod 777 /usr/bin/entrypoint.sh

ENTRYPOINT ["/usr/bin/entrypoint.sh"]
CMD ["apache2-foreground"]