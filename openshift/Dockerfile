FROM php:5.6-apache

RUN a2enmod rewrite

# install the PHP extensions we need
RUN apt-get update && apt-get install -y libpng12-dev libjpeg-dev && rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
	&& docker-php-ext-install gd
RUN docker-php-ext-install mysqli

VOLUME /var/www/html

ENV WORDPRESS_VERSION 4.3.1
ENV WORDPRESS_SHA1 b2e5652a6d2333cabe7b37459362a3e5b8b66221

# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
RUN curl -o wordpress.tar.gz -SL https://wordpress.org/wordpress-${WORDPRESS_VERSION}.tar.gz \
	&& echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c - \
	&& tar -xzf wordpress.tar.gz -C /usr/src/ \
	&& rm wordpress.tar.gz \
	&& tar cf - --one-file-system -C /usr/src/wordpress /var/www/html/ | tar xf - \
	&& chown -R www-data:www-data /usr/src/wordpress \
	&& chown -R www-data:www-data /var/www \
	&& chown -R www-data:www-data /var/www/html \
	&& chown -R www-data:www-data /var/www/html/* \
	&& chmod +x /var/www/html \
	&& chmod +x /var/www \
	&& chmod +x /usr/src/wordpress \
	&& chmod 777 /usr/src/wordpress \
	&& chmod 777 /usr/src/wordpress/* \
	&& chmod 777 /var/www \
	&& chmod 777 /var/www/html \
	&& chmod 777 /var/www/html/*

COPY docker-entrypoint.sh /entrypoint.sh

RUN chown www-data:www-data /entrypoint.sh
RUN chmod +x /entrypoint.sh
# grr, ENTRYPOINT resets CMD now
USER 1000090000
RUN useradd -r -g 1000090000 www-data

ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
