FROM alpine:3.7

ENV LUAJIT_LIB=/usr/local/luajit/lib
ENV LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
ENV LUAJIT_VERSION=2.0.5
ENV NGINX_VERSION=1.16.1
ENV LUA_NGINX_MODULE_VERSION=0.10.12
ENV NGX_DEVEL_KIT_VERSION=0.3.0

RUN builddir=$(mktemp -d) \
    && cd $builddir \
    && CONFIG="\
        --with-ld-opt=-Wl,-rpath,$LUAJIT_LIB \
        --with-threads \
        --with-compat \
        --with-file-aio \
        --with-http_ssl_module \
        --with-http_realip_module \
        --with-http_addition_module \
        --with-http_sub_module \
	    --with-http_dav_module \
        --with-http_flv_module \
        --with-http_mp4_module \
        --with-http_gunzip_module \
        --with-http_gzip_static_module \
        --with-http_random_index_module \
        --with-http_secure_link_module \
        --with-http_stub_status_module \
        --with-http_auth_request_module \
        --with-http_xslt_module=dynamic \
        --with-http_image_filter_module=dynamic \
        --with-http_geoip_module=dynamic \
        --with-http_slice_module \
        --with-http_v2_module \
        --with-stream \
        --with-stream_ssl_module \
        --with-stream_ssl_preread_module \
        --with-stream_realip_module \
        --with-stream_geoip_module=dynamic \
        --with-mail \
        --with-mail_ssl_module \
        --add-module=../lua-nginx-module-${LUA_NGINX_MODULE_VERSION} \
        --add-module=../ngx_devel_kit-${NGX_DEVEL_KIT_VERSION} \
    " \
    && apk add --no-cache --virtual .build-deps \
        gcc \
        libc-dev \
        make \
        openssl-dev \
        pcre-dev \
        zlib-dev \
        linux-headers \
        curl \
        gnupg \
        libxslt-dev \
        gd-dev \
        geoip-dev \
    && curl -fSL -o LuaJIT-${LUAJIT_VERSION}.tar.gz http://luajit.org/download/LuaJIT-${LUAJIT_VERSION}.tar.gz \
    && tar xzf LuaJIT-${LUAJIT_VERSION}.tar.gz \
    && cd LuaJIT-${LUAJIT_VERSION} \
    && make \
    && make install PREFIX=/usr/local/luajit \
    && curl -fSL -o lua-nginx-module-${LUA_NGINX_MODULE_VERSION}.tar.gz https://github.com/openresty/lua-nginx-module/archive/v${LUA_NGINX_MODULE_VERSION}.tar.gz \
    && tar xzf lua-nginx-module-${LUA_NGINX_MODULE_VERSION}.tar.gz \
    && curl -fSL -o ngx_devel_kit-${NGX_DEVEL_KIT_VERSION}.tar.gz https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz \
    && tar xzf ngx_devel_kit-${NGX_DEVEL_KIT_VERSION}.tar.gz \
    && curl -fSL -o nginx-${NGINX_VERSION}.tar.gz http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
    && tar xzf nginx-${NGINX_VERSION}.tar.gz \
    && cd nginx-${NGINX_VERSION} \
    && ./configure $CONFIG \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && make install \
    && mkdir /usr/local/nginx/conf/conf.d \
    && strip /usr/local/nginx/sbin/nginx \
    && strip /usr/local/nginx/modules/*.so \
    \
    # Clean
    && rm -rf $builddir \
    && apk add --no-cache --virtual .gettext gettext \
    && mv /usr/bin/envsubst /tmp/ \
    \
    && runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' /usr/local/nginx/sbin/nginx /usr/local/nginx/modules/*.so /tmp/envsubst \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )" \
    && apk add --no-cache --virtual .nginx-rundeps $runDeps \
    && apk del .build-deps \
    && apk del .gettext \
    && mv /tmp/envsubst /usr/local/bin/

WORKDIR /usr/local/nginx

STOPSIGNAL SIGTERM

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
