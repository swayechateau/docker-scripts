FROM alpine:3.18

RUN set -eux; \
	addgroup -g 70 -S postgres; \
	adduser -u 70 -S -D -G postgres -H -h /var/lib/postgresql -s /bin/sh postgres; \
	mkdir -p /var/lib/postgresql; \
	chown -R postgres:postgres /var/lib/postgresql

# make the "en_GB.UTF-8" locale so postgres will be utf-8 enabled by default
ENV LANG en_GB.utf8

RUN mkdir /docker-entrypoint-initdb.d

ENV PG_MAJOR 15
ENV PG_VERSION 15.4
ENV PG_SHA256 baec5a4bdc4437336653b6cb5d9ed89be5bd5c0c58b94e0becee0a999e63c8f9

ENV DOCKER_PG_LLVM_DEPS \
		llvm15-dev \
		clang15

RUN set -eux; \
	\
	wget -O postgresql.tar.bz2 "https://ftp.postgresql.org/pub/source/v$PG_VERSION/postgresql-$PG_VERSION.tar.bz2"; \
	echo "$PG_SHA256 *postgresql.tar.bz2" | sha256sum -c -; \
	mkdir -p /usr/src/postgresql; \
	tar \
		--extract \
		--file postgresql.tar.bz2 \
		--directory /usr/src/postgresql \
		--strip-components 1 \
	; \
	rm postgresql.tar.bz2; \
	\
	apk add --no-cache --virtual .build-deps \
		$DOCKER_PG_LLVM_DEPS \
		bison \
		coreutils \
		dpkg-dev dpkg \
		flex \
		g++ \
		gcc \
		krb5-dev \
		libc-dev \
		libedit-dev \
		libxml2-dev \
		libxslt-dev \
		linux-headers \
		make \
		openldap-dev \
		openssl-dev \
		perl-dev \
		perl-ipc-run \
		perl-utils \
		python3-dev \
		tcl-dev \
		util-linux-dev \
		zlib-dev \
		icu-dev \
		lz4-dev \
		zstd-dev \
	; \
	\
	cd /usr/src/postgresql; \
	awk '$1 == "#define" && $2 == "DEFAULT_PGSOCKET_DIR" && $3 == "\"/tmp\"" { $3 = "\"/var/run/postgresql\""; print; next } { print }' src/include/pg_config_manual.h > src/include/pg_config_manual.h.new; \
	grep '/var/run/postgresql' src/include/pg_config_manual.h.new; \
	mv src/include/pg_config_manual.h.new src/include/pg_config_manual.h; \
	gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	wget -O config/config.guess 'https://git.savannah.gnu.org/cgit/config.git/plain/config.guess?id=7d3d27baf8107b630586c962c057e22149653deb'; \
	wget -O config/config.sub 'https://git.savannah.gnu.org/cgit/config.git/plain/config.sub?id=7d3d27baf8107b630586c962c057e22149653deb'; \
	export LLVM_CONFIG="/usr/lib/llvm15/bin/llvm-config"; \
	export CLANG=clang-15; \
# configure options taken from:
	./configure \
		--enable-option-checking=fatal \
		--build="$gnuArch" \
		--enable-integer-datetimes \
		--enable-thread-safety \
		--enable-tap-tests \
		--disable-rpath \
		--with-uuid=e2fs \
		--with-gnu-ld \
		--with-pgport=5432 \
		--with-system-tzdata=/usr/share/zoneinfo \
		--prefix=/usr/local \
		--with-includes=/usr/local/include \
		--with-libraries=/usr/local/lib \
		--with-gssapi \
		--with-ldap \
		--with-tcl \
		--with-perl \
		--with-python \
		--with-openssl \
		--with-libxml \
		--with-libxslt \
		--with-icu \
		--with-llvm \
		--with-lz4 \
		--with-zstd \
	; \
	make -j "$(nproc)" world; \
	make install-world; \
	make -C contrib install; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
# Remove plperl, plpython and pltcl dependencies by default to save image size
# To use the pl extensions, those have to be installed in a derived image
			| grep -v -e perl -e python -e tcl \
	)"; \
	apk add --no-cache --virtual .postgresql-rundeps \
		$runDeps \
		bash \
		su-exec \
		tzdata \
		zstd \
		icu-data-full \
		$([ "$(apk --print-arch)" != 'ppc64le' ] && echo 'nss_wrapper') \
	; \
	apk del --no-network .build-deps; \
	cd /; \
	rm -rf \
		/usr/src/postgresql \
		/usr/local/share/doc \
		/usr/local/share/man \
	; \
	\
	postgres --version

RUN set -eux; \
	cp -v /usr/local/share/postgresql/postgresql.conf.sample /usr/local/share/postgresql/postgresql.conf.sample.orig; \
	sed -ri "s!^#?(listen_addresses)\s*=\s*\S+.*!\1 = '*'!" /usr/local/share/postgresql/postgresql.conf.sample; \
	grep -F "listen_addresses = '*'" /usr/local/share/postgresql/postgresql.conf.sample

RUN mkdir -p /var/run/postgresql && chown -R postgres:postgres /var/run/postgresql && chmod 3777 /var/run/postgresql

ENV PGDATA /var/lib/postgresql/data

RUN mkdir -p "$PGDATA" && chown -R postgres:postgres "$PGDATA" && chmod 1777 "$PGDATA"
VOLUME /var/lib/postgresql/data

COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT ["entrypoint.sh"]

STOPSIGNAL SIGINT

EXPOSE 5432
CMD ["postgres"]