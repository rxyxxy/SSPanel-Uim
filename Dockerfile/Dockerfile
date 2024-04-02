FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive

RUN mkdir -p /run/php && mkdir app && apt update && apt install -y git wget curl gnupg2 ca-certificates lsb-release ubuntu-keyring software-properties-common tzdata && \
    ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata

WORKDIR /app
ADD main.sh .

RUN chmod 777 main.sh && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor | \
    tee /usr/share/keyrings/nginx-archive-keyring.gpg > /dev/null && \
    echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" | \
    tee /etc/apt/sources.list.d/nginx.list && \
    printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" > /etc/apt/preferences.d/99nginx


RUN curl -fsSL https://packages.redis.io/gpg |  gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg && echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/redis.list


# 安装 PHP 8.3 和一些扩展
RUN add-apt-repository -y ppa:ondrej/php && add-apt-repository -y ppa:redislabs/redis && apt update -y && apt install -y nginx redis php8.3 php8.3-bcmath php8.3-bz2 php8.3-cli php8.3-common php8.3-curl \
    php8.3-fpm php8.3-gd php8.3-igbinary php8.3-mbstring php8.3-mysql \
    php8.3-opcache php8.3-readline php8.3-redis php8.3-xml php8.3-yaml php8.3-zip

RUN sed -i 's/user nginx;/user www-data;/' /etc/nginx/nginx.conf && sed -i 's/user  nginx;/user www-data;/' /etc/nginx/nginx.conf

RUN sed -i 's@^disable_functions.*@disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_restore,dl,readlink,symlink,popepassthru,stream_socket_server,fsocket,popen@' /etc/php/8.3/fpm/php.ini && sed -i 's@^disable_functions.*@disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_restore,dl,readlink,symlink,popepassthru,stream_socket_server,fsocket,popen@' /etc/php/8.3/cli/php.ini

RUN echo 'zend_extension=opcache.so' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.file_cache=/tmp' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.interned_strings_buffer=64' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.jit=on' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.jit_buffer_size=256M' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.max_accelerated_files=65535' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.memory_consumption=512' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.revalidate_freq=60' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.validate_permission=on' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini \
    && echo 'opcache.validate_root=on' >> /etc/php/8.3/fpm/conf.d/10-opcache.ini


# 暴露端口
EXPOSE 80

# 启动 PHP-FPM 服务
CMD ["./main.sh"]
