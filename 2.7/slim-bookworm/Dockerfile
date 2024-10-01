FROM debian:bookworm-slim

LABEL \
    maintainer="César Román <cesar@coatl.dev>" \
    repository="https://github.com/coatl-dev/docker-python" \
    vendor="coatl.dev"

ENV DEBIAN_FRONTEND=noninteractive

ENV PIP_NO_CACHE_DIR=1
ENV PIP_NO_PYTHON_VERSION_WARNING=1
ENV PIP_ROOT_USER_ACTION=ignore

# ensure local python is preferred over distribution python
ENV PATH=/usr/local/bin:$PATH

# cannot remove LANG even though https://bugs.python.org/issue19846 is fixed
# last attempted removal of LANG broke many users:
# https://github.com/docker-library/python/pull/570
ENV LANG=C.UTF-8

# https://github.com/docker-library/python/issues/147
ENV PYTHONIOENCODING=UTF-8

# runtime dependencies
RUN set -eux; \
    \
    apt-get update --quiet; \
    apt-get upgrade --yes; \
	apt-get install --yes --no-install-recommends \
		ca-certificates \
		netbase \
		tzdata \
	; \
	rm -rf /var/lib/apt/lists/*

ENV PYTHON_VERSION=2.7.18
ENV PYTHON_TAR_SHA256=b62c0e7937551d0cc02b8fd5cb0f544f9405bafc9a54d3808ed4594812edef43

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# hadolint ignore=DL3003,SC2086
RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update --quiet; \
	apt-get install --yes --no-install-recommends \
		dpkg-dev \
		gcc \
		gnupg \
		libbz2-dev \
		libc6-dev \
		libdb-dev \
		libgdbm-dev \
		libncursesw5-dev \
		libreadline-dev \
		libsqlite3-dev \
		libssl-dev \
		make \
		tk-dev \
		wget \
		xz-utils \
		zlib1g-dev \
	; \
	\
	wget -q -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz"; \
	echo "$PYTHON_TAR_SHA256 *python.tar.xz" | sha256sum --check --strict -; \
	mkdir -p /usr/src/python; \
	tar --extract --directory /usr/src/python --strip-components=1 --file python.tar.xz; \
	rm python.tar.xz; \
	\
	cd /usr/src/python; \
	gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	./configure \
		--build="$gnuArch" \
		--enable-optimizations \
		--enable-option-checking=fatal \
		--enable-shared \
        --enable-unicode=ucs4 \
	; \
    make -s -j "$(nproc)" \
# setting PROFILE_TASK makes "--enable-optimizations" reasonable: https://bugs.python.org/issue36044 / https://github.com/docker-library/python/issues/160#issuecomment-509426916
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
    ; \
	make install; \
	ldconfig; \
	\
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec ldd '{}' ';' \
		| awk '/=>/ { so = $(NF-1); if (index(so, "/usr/local/") == 1) { next }; gsub("^/(usr/)?", "", so); printf "*%s\n", so }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge --yes --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*; \
	\
	cd /; \
	rm -rf /usr/src/python; \
	\
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name 'libpython*.a' \) \) \
		\) -exec rm -rf '{}' + \
	; \
	\
	python2 --version

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION=20.3.4
ENV PYTHON_SETUPTOOLS_VERSION=44.1.1
ENV PYTHON_WHEEL_VERSION=0.37.1
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL=https://raw.githubusercontent.com/pypa/get-pip/HEAD/public/2.7/get-pip.py
ENV PYTHON_GET_PIP_SHA256=40ee07eac6674b8d60fce2bbabc148cf0e2f1408c167683f110fd608b8d6f416

# hadolint ignore=SC2086
RUN set -ex; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update --quiet; \
	apt-get install --yes --no-install-recommends wget; \
	\
	wget -q -O get-pip.py "$PYTHON_GET_PIP_URL"; \
	echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum --check --strict -; \
	\
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
	apt-get purge --yes --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*; \
	\
	python get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
		"pip==$PYTHON_PIP_VERSION" \
	; \
	pip --version; \
	\
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +; \
	rm -f get-pip.py

CMD ["python2"]