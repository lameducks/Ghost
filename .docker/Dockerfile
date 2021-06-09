## 1st stage

FROM node:14-alpine3.12 as build

RUN set -eux; \
    apk update; \
    apk upgrade; \
    apk add git; \
    yarn config set network-timeout 1200000

COPY core/client/package.json core/client/yarn.lock /work/sources/core/client/
RUN set -eux; \
    yarn global add grunt-cli \
    cd /work/sources/core/client; \
    yarn install

COPY package.json yarn.lock /work/sources/
RUN set -eux; \
    cd /work/sources; \
    yarn install --prod; \
    cp -rp node_modules /work/node_modules-prod; \
    yarn install

COPY . /work/sources/
RUN set -eux; \
    cd /work/sources; \
    grunt release; \
    cp -rpT /work/sources/.docker/overlay /work/sources/.build/release/

## 2nd stage

FROM node:14-alpine3.12

ENV NODE_ENV production
ENV GHOST_INSTALL /var/lib/ghost
ENV GHOST_CONTENT /var/lib/ghost/content

EXPOSE 2368
VOLUME ["$GHOST_CONTENT/images", "$GHOST_CONTENT/settings"]
WORKDIR $GHOST_INSTALL
CMD ["node", "index.js"]

RUN set -eux; \
    apk upgrade --no-cache; \
    rm -rfv /tmp/*

COPY --from=build /work/node_modules-prod $GHOST_INSTALL/node_modules
COPY --from=build /work/sources/.build/release $GHOST_INSTALL/
