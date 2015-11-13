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
	&& chown -R 1000080000:www-data /usr/src/wordpress \
        && sed -e 's/Listen 80/Listen 8080/' -i /etc/apache2/apache2.conf /etc/apache2/ports.conf \
        && sed -e 's/ErrorLog \/proc\/self\/fd\/2/ErrorLog \/var\/www\/html\/error/' -i /etc/apache2/apache2.conf \
        && chown -R 1000080000:www-data /var/www \
        && chown -R 1000080000:www-data /var/www/html

EXPOSE 8080        

COPY docker-entrypoint.sh /entrypoint.sh	
RUN chmod +x /entrypoint.sh && chown 1000080000:www-data /entrypoint.sh
# grr, ENTRYPOINT resets CMD now

ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
