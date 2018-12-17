[![N|Solid](https://s3.amazonaws.com/ejf3-public/hosted_files/ejf_io/docker_nginx.png)](https://www.docker.com/)

# Esse Dockerfile cria uma imagem de um servidor web NGINX, ALPINE com PHP

* [Dockerfile](https://github.com/PalamarSolutionIT/Dockerfile/blob/master/Dockerfile)

## Uso

Para adicionar seus próprios arquivos estáticos e executá-lo como um contêiner do Windows, use este Dockerfile

### Dockerfile

```Dockerfile
# Instala o NGINX x ALPINE.
FROM nginx:mainline-alpine

# Responsavel pela Imagem.
LABEL maintainer="Palamar <palamar@palamarsolutionit.com.br>"

# INSTALA ALGUNS PACOTES DO SISTEMA.
RUN apk --update --no-cache add ca-certificates \
    bash \
    supervisor

# chave pública do projeto.
ADD https://php.codecasts.rocks/php-alpine.rsa.pub /etc/apk/keys/php-alpine.rsa.pub

# Argumentos padrão da imagem.
ARG PHP_VERSION=7.2
ARG ALPINE_VERSION=3.7
ARG COMPOSER_HASH=544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061
ARG NGINX_HTTP_PORT=80
ARG NGINX_HTTPS_PORT=443

# CONFIGURE OS REPOSITÓRIOS ALPINE E O PHP BUILD DIR.
RUN echo "http://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VERSION}/main" > /etc/apk/repositories && \
    echo "http://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VERSION}/community" >> /etc/apk/repositories && \
    echo "@php https://php.codecasts.rocks/v${ALPINE_VERSION}/php-${PHP_VERSION}" >> /etc/apk/repositories

# INSTALAR PHP E ALGUMAS EXTENSÕES: https://github.com/codecasts/php-alpine
RUN apk add --no-cache --update php-fpm@php \
    php@php \
    php-openssl@php \
    php-pdo@php \
    php-pdo_mysql@php \
    php-mbstring@php \
    php-phar@php \
    php-session@php \
    php-dom@php \
    php-ctype@php \
    php-zlib@php \
    php-json@php \
    php-xml@php && \
    ln -s /usr/bin/php7 /usr/bin/php

# Configura o Servidor Web.
RUN mkdir -p /var/www && \
    mkdir -p /run/php && \
    mkdir -p /run/nginx && \
    mkdir -p /var/log/supervisor && \
    mkdir -p /etc/nginx/sites-enabled && \
    mkdir -p /etc/nginx/sites-available && \
    rm /etc/nginx/nginx.conf && \
    rm /etc/php7/php-fpm.d/www.conf && \
    rm /etc/php7/php.ini

# Instala o COMPOSER.
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php -r "if (hash_file('SHA384', 'composer-setup.php') === '${COMPOSER_HASH}') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" && \
    php composer-setup.php --install-dir=/usr/bin --filename=composer && \
    php -r "unlink('composer-setup.php');"

# ADICIONA SCRIPT, SUPERVISOR CONFIG, NGINX CONFIG E EXECUTAR OS SCRIPTS.
ADD start.sh /start.sh
ADD config/supervisor/supervisord.conf /etc/supervisord.conf
ADD config/nginx/nginx.conf /etc/nginx/nginx.conf
ADD config/nginx/site.conf /etc/nginx/sites-available/default.conf
ADD config/php/php.ini /etc/php7/php.ini
ADD config/php-fpm/www.conf /etc/php7/php-fpm.d/www.conf
RUN chmod 755 /start.sh

# DEFINE A PORTA DO SERVIDOR WEB.
EXPOSE ${NGINX_HTTPS_PORT} ${NGINX_HTTP_PORT}

# DEFINE O DIRETORIO PADRAO.
WORKDIR /var/www

# INICIA O SERVIDOR
CMD ["/start.sh"]
```

### Crie a própria imagem do Docker

Em seguida, crie sua imagem do Docker com

```
docker build -t webserver .
```

### Iniciar o container

Execute seu contêiner com

```
docker run -d --name=webserver -p 80:80 webserver
```

## License
APACHE 2.0
