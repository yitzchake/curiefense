ARG RUSTBIN_TAG=latest
FROM curiefense/curiefense-rustbuild-bionic:${RUSTBIN_TAG} AS rustbin
FROM docker.io/openresty/openresty:1.19.9.1-1-bionic
COPY conf.d /etc/nginx/conf.d
COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY curieproxy/lua /lua
COPY curieproxy/lua/shared-objects/*.so /usr/local/lib/lua/5.1/
COPY --from=rustbin /root/curiefense.so /usr/local/lib/lua/5.1/

COPY curieproxy/cf-config /bootstrap-config/config
COPY curieproxy/rust/static/libgrasshopper.so /usr/lib/

RUN apt-get update && apt-get install -y libhyperscan4 logrotate cron
RUN mkdir /cf-config && chmod a+rwxt /cf-config
RUN openssl req -new -newkey rsa:4096 -days 3650 -nodes -x509 -subj "/C=fr/O=curiefense/CN=testsystem" -keyout /etc/ssl/certificate.key -out /etc/ssl/certificate.crt

RUN mkfifo /nginx-accesslogs

COPY start.sh /usr/bin/

RUN curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.13.3-amd64.deb && dpkg -i filebeat-7.13.3-amd64.deb && rm filebeat-7.13.3-amd64.deb
COPY filebeat/filebeat-nginx.yml /etc/filebeat.yml
COPY nginx.logrotate /etc/logrotate.d/nginx
COPY main.logrotate /etc/logrotate.conf
RUN chmod go-w /etc/filebeat.yml

CMD service cron start && /bin/bash /usr/bin/start.sh
