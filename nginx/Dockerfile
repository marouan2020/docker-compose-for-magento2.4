FROM nginx:1.9

MAINTAINER Ben Mansour Marouan <marouan.ben.mansour@gmail.com>

ADD etc/vhost.conf /etc/nginx/conf.d/default.conf

COPY etc/certs/ /etc/nginx/ssl/
ADD bin/* /usr/local/bin/

EXPOSE 443

CMD echo "127.0.0.1      api.local.dev" >> /etc/hosts
EXPOSE 80

ENV UPLOAD_MAX_FILESIZE 98M
ENV FPM_HOST fpm
ENV FPM_PORT 9000
ENV MAGENTO_ROOT /var/www/magento
ENV MAGENTO_RUN_MODE developer
ENV DEBUG false

RUN ["chmod", "+x", "/usr/local/bin/docker-environment"]

ENTRYPOINT ["/usr/local/bin/docker-environment"]
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

CMD ["nginx"]
