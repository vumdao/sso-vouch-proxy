FROM nginx:alpine

ADD nginx.conf /etc/nginx/
ADD conf.d /etc/nginx/conf.d

ENV USER=nginx \
    GROUP=nginx

RUN chmod +r /etc/resolv.conf && \
    mkdir -p /var/log/nginx /var/cache/nginx /var/run/nginx && \
    chown -R ${USER}:${GROUP} /etc/nginx /var/log/nginx /var/cache/nginx /var/run/nginx

WORKDIR /etc/nginx

USER $USER

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]