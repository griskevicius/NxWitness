# Use LSIO Ubuntu Bionic
FROM lsiobase/ubuntu:bionic

# Labels
ARG LABEL_NAME="NxWitness-LSIO"
ARG LABEL_DESCRIPTION="Nx Witness VMS Docker based on LinuxServer"

# Download URL and version
ARG DOWNLOAD_URL="https://updates.networkoptix.com/default/31767/linux/nxwitness-server-4.1.0.31767-linux64-patch.deb"
ARG DOWNLOAD_VERSION="4.1.0.31767"

# NxWitness (networkoptix) or DWSpectrum (digitalwatchdog) or NxMeta (networkoptix-metavms)
ARG RUNTIME_NAME="networkoptix"

# The RUN wget command will be cached unless we change the cache tag
# Use the download version for the cache tag
ARG CACHE_DATE=${DOWNLOAD_VERSION}

# Prevent EULA and confirmation prompts in installers
ENV DEBIAN_FRONTEND=noninteractive

# Media server user and directory name
ENV COMPANY_NAME=${RUNTIME_NAME}

# Labels
LABEL name=${LABEL_NAME} \
    description=${LABEL_DESCRIPTION} \
    version=${DOWNLOAD_VERSION} \
    download=${DOWNLOAD_URL} \
    maintainer="Pieter Viljoen <ptr727@users.noreply.github.com>"

# Install tools
RUN apt-get update \
    && apt-get install --no-install-recommends --yes \
        mc \
        nano \
        strace \
        wget \
    && apt-get clean \
    && apt-get autoremove --purge \
    && rm -rf /var/lib/apt/lists/*

# Download the DEB installer file
RUN wget --no-verbose --no-check-certificate --output-document=./vms_server.deb ${DOWNLOAD_URL}

# LSIO: Replace the abc usernames with the mediaserver names
# The mediaserver calls "chown ${COMPANY_NAME}" at runtime
# We have to match the ${COMPANY_NAME} username with the LSIO "abc" usernames
# https://github.com/linuxserver/docker-baseimage-alpine/blob/master/root/etc/cont-init.d/10-adduser
# Change user "abc" to ${COMPANY_NAME}
RUN usermod -l ${COMPANY_NAME} abc \
# Change group "abc" to ${COMPANY_NAME}
    && groupmod -n ${COMPANY_NAME} abc \
# Replace "abc" with ${COMPANY_NAME}
    && sed -i "s/abc/\${COMPANY_NAME}/g" /etc/cont-init.d/10-adduser

# Install the mediaserver
# Add missing dependencies (gdb)
# Remove the root tool to prevent it from being used in service mode
RUN apt-get update \
    && apt-get install --no-install-recommends --yes \
        gdb \
        ./vms_server.deb \
    && apt-get clean \
    && apt-get autoremove --purge \
    # Do not delete the root-tool, it is required to get DMI information when not running as root
    # && rm -rf /opt/${COMPANY_NAME}/mediaserver/bin/root-tool-bin \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf ./vms_server.deb

# LSIO : Set ownership permissions
RUN chown --verbose ${COMPANY_NAME}:${COMPANY_NAME} /opt/${COMPANY_NAME}/mediaserver/bin \
    && chown --verbose ${COMPANY_NAME}:${COMPANY_NAME} /opt/${COMPANY_NAME}/mediaserver/bin/external.dat

# LSIO: Copy etc init and services files
# The scripts are using the ${COMPANY_NAME} global environment variable
# https://github.com/just-containers/s6-overlay#container-environment
COPY root/etc /etc

# Expose port 7001
EXPOSE 7001

# Create mount points
# Links will be created at runtime in the etc/cont-init.d/50-relocate-files script
# /opt/digitalwatchdog/mediaserver/etc -> /config/etc
# /opt/digitalwatchdog/mediaserver/var -> /config/var
# /opt/digitalwatchdog/mediaserver/var/data -> /media
# /config is for configuration
# /media is for media recording
# /archive is for media backups
VOLUME /config /media /archive

