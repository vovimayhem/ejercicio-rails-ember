FROM node:8-alpine AS runtime

# 2: We'll set the application path as the working directory
WORKDIR /usr/src

# 3: We'll add the app's binaries path to $PATH and set the $HOME to the app folder:
ENV HOME=/usr/src PATH=/usr/src/bin:$PATH

# 4: Install yarn and ember-cli:
RUN apk add --no-cache yarn openssl

RUN npm install -g ember-cli

# Stage 2: Development
FROM runtime AS development

RUN apk add --no-cache \
  chromium --repository http://dl-cdn.alpinelinux.org/alpine/edge/community

RUN npm install -g check-dependencies

# 6: Install watchman:
RUN set -ex \
 && apk add --no-cache --virtual .build-deps ca-certificates openssl automake autoconf linux-headers build-base python-dev \
 && cd /tmp \
 && export WATCHMAN_VERSION=4.7.0 \
 && export WATCHMAN_SHA256=77c7174c59d6be5e17382e414db4907a298ca187747c7fcb2ceb44da3962c6bf \
 && wget -O watchman.tar.gz "https://github.com/facebook/watchman/archive/v${WATCHMAN_VERSION}.tar.gz" \
 && echo "$WATCHMAN_SHA256 *watchman.tar.gz" | sha256sum -c - \
 && tar -xz -f watchman.tar.gz -C /tmp/ \
 && cd /tmp/watchman-${WATCHMAN_VERSION} \
 && ./autogen.sh \
 && ./configure \
 && make \
 && make install \
 && apk del .build-deps \
 && cd $HOME \
 && rm -rf /tmp/*

 # 8: Set the default command:
 CMD ["ember", "server", "--live-reload-port", "35730"]