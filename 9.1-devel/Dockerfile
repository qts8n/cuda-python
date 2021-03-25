FROM nvidia/cuda:9.1-cudnn7-devel-ubuntu16.04
LABEL maintainer="qts8n <efisher9680@gmail.com>"

# nvidia cuda 9.1 paths
ENV LD_LIBRARY_PATH=/usr/local/cuda-9.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
ENV LIBRARY_PATH=${LIBRARY_PATH}:/usr/local/cuda-9.1/lib64

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

ENV LANG C.UTF-8

# ensure annoying packages like `tzdata` won't ask a lot of questions
ARG DEBIAN_FRONTEND=noninteractive

# runtime dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates curl netbase wget && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get install -y build-essential

ENV GPG_KEY 0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
ENV PYTHON_VERSION 3.7.5

RUN mkdir ~/.gnupg
RUN echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf
RUN set -ex \
	&& savedAptMark="$(apt-mark showmanual)" \
	&& echo "Europe/Moscow" > /etc/timezone \
	&& apt-get update \
    && apt-get install -y --no-install-recommends \
        dpkg-dev \
        gcc \
        libbz2-dev \
        libc6-dev \
        libexpat1-dev \
        libffi-dev \
        libgdbm-dev \
        liblzma-dev \
        libncursesw5-dev \
        libreadline-dev \
        libsqlite3-dev \
        make \
        tk-dev \
        uuid-dev \
        xz-utils \
        zlib1g-dev \
        $(command -v gpg > /dev/null || echo 'gnupg dirmngr') \
    && wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
	&& wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
	&& gpg --batch --verify python.tar.xz.asc python.tar.xz \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
	&& rm -rf "$GNUPGHOME" python.tar.xz.asc \
	&& mkdir -p /usr/src/python \
	&& tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
	&& rm python.tar.xz \
	&& cd /usr/src/python \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--enable-loadable-sqlite-extensions \
		--enable-optimizations \
		--enable-shared \
		--with-system-expat \
		--with-system-ffi \
		--without-ensurepip \
	&& make -j "$(nproc)" \
    PROFILE_TASK='-m test.regrtest --pgo \
        test_array \
        test_base64 \
        test_binascii \
        test_binhex \
        test_binop \
        test_bytes \
        test_c_locale_coercion \
        test_class \
        test_cmath \
        test_codecs \
        test_compile \
        test_complex \
        test_csv \
        test_decimal \
        test_dict \
        test_float \
        test_fstring \
        test_hashlib \
        test_io \
        test_iter \
        test_json \
        test_long \
        test_math \
        test_memoryview \
        test_pickle \
        test_re \
        test_set \
        test_slice \
        test_struct \
        test_threading \
        test_time \
        test_traceback \
        test_unicode \
    ' \
	&& make install \
	&& ldconfig \
	&& apt-mark auto '.*' > /dev/null \
	&& apt-mark manual $savedAptMark \
	&& find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
	&& rm -rf /var/lib/apt/lists/* \
	&& find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' + \
	&& rm -rf /usr/src/python \
	&& python3 --version

# make some useful symlinks that are expected to exist
RUN cd /usr/local/bin \
	&& ln -s idle3 idle \
	&& ln -s pydoc3 pydoc \
	&& ln -s python3 python \
	&& ln -s python3-config python-config