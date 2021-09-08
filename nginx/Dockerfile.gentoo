FROM gentoo/portage:20210908 as portage
FROM gentoo/stage3:hardened-20210908 as gentoo
COPY --from=portage /var/db/repos/gentoo /var/db/repos/gentoo
RUN mkdir -p /destination

RUN USE="-aio -rtmp -doc -gpm minimal -profile -test -cxx -nls -readline -installkernel " \
    NGINX_MODULES_MAIL="" \
    NGINX_MODULES_STREAM="" \
    ROOT=/destination \
     emerge --quiet www-servers/nginx

RUN ROOT=/destination emerge --quiet -C \
      app-arch/bzip2 \
      app-arch/gzip \
      sys-apps/gawk \
      sys-apps/debianutils\
      sys-libs/zlib \
      sys-apps/grep \
      sys-apps/gentoo-functions \
      virtual/awk


RUN ls -la /destination/var/db/pkg/*

RUN rm -rf \
        /destination/usr/bin \
        /destination/usr/share/doc \
        /destination/usr/share/gtk-doc \
        /destination/usr/share/eselect \
        /destination/usr/share/info \
        /destination/usr/share/man \
        /destination/var/db/pkg \
        /destination/usr/share/aclocal \
        /destination/usr/share/gettext \
        /destination/usr/include \
        /destination/var/lib/gentoo \
        /destination/var/lib/portage \
        /destination/var/cache/edb

RUN mkdir -p /destination/run && touch /destination/run/nginx.pid && chown nginx:nginx /destination/run/nginx.pid
RUN mkdir -p /destination/var/www

RUN mkdir -p /destination/var/log/nginx && ln -sf /dev/stdout /destination/var/log/nginx/error_log && chown nginx /destination/var/log/nginx/error_log

RUN ls -Rla /destination/*

FROM scratch
LABEL maintainer="Aleksandr Muravja <alex@kyberorg.io>"

COPY --from=gentoo /destination /

COPY --from=gentoo /etc/passwd /etc/passwd
COPY --from=gentoo /etc/group /etc/group

COPY nginx.conf /etc/nginx/nginx.conf
COPY site.conf /etc/nginx/conf.d/site.conf
COPY index.html /var/www/index.html

EXPOSE 8080
STOPSIGNAL SIGTERM
USER nginx
CMD ["nginx", "-g", "daemon off;"]
