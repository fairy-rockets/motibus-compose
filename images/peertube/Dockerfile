# From: https://github.com/Chocobozzz/PeerTube/blob/develop/support/docker/production/Dockerfile.buster

FROM node:12-buster-slim

# Allow to pass extra options to the npm run build
# eg: --light --light-fr to not build all client languages
#     (speed up build time if i18n is not required)
ARG NPM_RUN_BUILD_OPTS

# Install dependencies
RUN apt update \
 && apt install -y --no-install-recommends openssl ffmpeg python ca-certificates gnupg gosu \
 && gosu nobody true \
 && rm /var/lib/apt/lists/* -fR

# Add peertube user
RUN groupadd -r peertube \
    && useradd -r -g peertube -m peertube

# Install PeerTube

# Copy sources
COPY --chown=peertube:peertube ./src /app

# Build
WORKDIR /app
USER peertube
RUN yarn install --pure-lockfile \
    && npm run build -- $NPM_RUN_BUILD_OPTS \
    && rm -r ./node_modules ./client/node_modules \
    && yarn install --pure-lockfile --production \
    && yarn cache clean

USER root

RUN mkdir /data /config
RUN chown -R peertube:peertube /data /config

ENV NODE_ENV production
ENV NODE_CONFIG_DIR /config

VOLUME /data
VOLUME /config

COPY ./src/support/docker/production/entrypoint.sh /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

# Expose API and RTMP
EXPOSE 9000 1935

# Run the application
CMD ["npm", "start"]
