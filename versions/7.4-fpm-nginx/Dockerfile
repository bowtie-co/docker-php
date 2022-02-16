FROM php:7.4-fpm-buster

ENV APP_DIR /var/www/app
ENV APP_USER www-data

ENV NGINX_PATH=/etc/nginx

RUN mkdir -p $APP_DIR

# Set working directory
WORKDIR $APP_DIR

# Install apt requirements & dependencies
RUN apt-get update && apt-get install -y \
    build-essential default-mysql-client postgresql \
    libpng-dev libjpeg62-turbo-dev libfreetype6-dev libmagickwand-dev libzip-dev libssl-dev libbz2-dev libicu-dev libpq-dev libmcrypt-dev libonig-dev \
    locales zip jpegoptim optipng pngquant gifsicle vim xvfb unzip git curl cron nginx supervisor \
    #
    # Install imagick, mcrypt, and redis using pecl
    && pecl install imagick-3.4.4 mcrypt-1.0.4 redis \
    #
    # Install composer
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    #
    # Clear apt cache
    && apt-get clean && rm -rf /var/lib/apt/lists/*


# # Install extensions
RUN docker-php-ext-install pdo_mysql pdo_pgsql mbstring zip exif pcntl bcmath bz2 ftp gettext opcache shmop sockets sysvmsg sysvsem sysvshm iconv intl \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd \
    && docker-php-ext-enable imagick mcrypt redis


COPY etc/nginx $NGINX_PATH
COPY etc/php/* /usr/local/etc/php/
COPY etc/php-fpm.d/* /usr/local/etc/php-fpm.d/
COPY etc/supervisord/* /etc/supervisor/conf.d/

# Remove default nginx sites-enabled
RUN rm $NGINX_PATH/sites-enabled/* \
    && ln -s $NGINX_PATH/sites-available/app.conf $NGINX_PATH/sites-enabled/app.conf \
    && nginx -t

# Copy existing application directory contents
COPY . .

# Expose port 80 and start php-fpm server
EXPOSE 80

# CMD ["php-fpm"]
CMD [ "/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf" ]
