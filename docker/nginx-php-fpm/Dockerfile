# Multi-stage builds are a new feature requiring Docker 17.05 or higher on the daemon and client. 

# stage
FROM composer:latest as composer
LABEL stage=intermediate

COPY ./src /app
RUN composer install
# stage END

FROM amazonlinux

RUN yum update -y \
    && amazon-linux-extras install epel php7.3 \
    && yum install -y nginx supervisor mysql gd php-ldap php-soap php-xml php-mbstring php-zip php-bcmath php-pecl-imagick \
    && yum clean all

WORKDIR /usr/share/nginx/html
COPY --from=composer /app /usr/share/nginx/html
# COPY ./docker/nginx-php-fpm/nginx.conf /etc/nginx/nginx.conf
COPY ./docker/nginx-php-fpm/override-php-fpm.conf /etc/php-fpm.d/override-php-fpm.conf
COPY ./docker/nginx-php-fpm/supervisord.ini /etc/supervisord.d/supervisord.ini

# forward request and error logs to docker log collector
# https://docs.docker.com/v17.09/engine/admin/logging/view_container_logs/
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    && ln -sf /dev/stderr /var/log/php-fpm/error.log \
    && ln -sf /dev/stderr /var/log/php-fpm/www-error.log

CMD ["/usr/bin/supervisord", "-n"]