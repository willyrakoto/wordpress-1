FROM php:5.6-apache

RUN a2enmod rewrite expires

# install the PHP extensions we need
RUN apt-get update && apt-get install -y libpng12-dev libjpeg-dev && rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
	&& docker-php-ext-install gd mysqli opcache

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

ENV WORDPRESS_VERSION 4.4.1
ENV WORDPRESS_SHA1 89bcc67a33aecb691e879c818d7e2299701f30e7

#VOLUME /var/www/html
# Openshift v3 : Add 3 volumes to let apache process write on it.
#VOLUME /var/lock/apache2
#VOLUME /var/run/apache2
#VOLUME /var/log/apache2


# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
RUN curl -o wordpress.tar.gz -SL https://wordpress.org/wordpress-${WORDPRESS_VERSION}.tar.gz \
	&& echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c - \
	&& tar -xzf wordpress.tar.gz -C /usr/src/ \
	&& rm wordpress.tar.gz \
	&& chown -R www-data:www-data /usr/src/wordpress

RUN chmod 777 /var/www/html
 && chmod 777 /var/lock/apache2 \
 && chmod 777 /var/run/apache2 \
 && chmod 777 /var/log/apache2
	
# Openshift v3 Add custom logs to avoid Permission Denied on /proc/self/fd/1 or /proc/self/fd/2 
# Openshift v3 Change default Apache2 listening port to 8080
COPY apache.log /var/log/apache2/custom.log
COPY apache.log /var/log/apache2/error.log
RUN sed -e 's/Listen 80/Listen 8080/' -i /etc/apache2/apache2.conf /etc/apache2/ports.conf \
 && sed -i 's/ErrorLog .*/ErrorLog \/var\/log\/apache2\/error.log/' /etc/apache2/apache2.conf \
 && sed -i 's/CustomLog .*/CustomLog \/var\/log\/apache2\/custom.log combined/' /etc/apache2/apache2.conf \
 && sed -i 's/LogLevel .*/LogLevel info/' /etc/apache2/apache2.conf
 
EXPOSE 8080

COPY docker-entrypoint.sh /entrypoint.sh
# Openshift v3 Add execution right on EntryPoint
RUN chmod +x /entrypoint.sh

# grr, ENTRYPOINT resets CMD now
ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
