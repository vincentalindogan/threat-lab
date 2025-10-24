FROM amazonlinux:2023 AS builder

ARG WAZUH_VERSION=4.14.0
ARG WAZUH_TAG_REVISION=1
ARG WAZUH_PACKAGE_VERSION=4.x
ARG NODE_NAME=index-1

ENV NODE_NAME ${NODE_NAME}

COPY wazuh-cs.tar /

RUN rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH && \
    echo -e "[wazuh]\ngpgcheck=1\ngpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH\nenabled=1\nname=EL-\$releasever - Wazuh\nbaseurl=https://packages.wazuh.com/${WAZUH_PACKAGE_VERSION}/yum/\npriority=1" | tee /etc/yum.repos.d/wazuh.repo

RUN yum install -y curl-minimal openssl xz tar findutils \
    shadow-utils procps-ng util-linux grep sed \
    wazuh-indexer-${WAZUH_VERSION}-${WAZUH_TAG_REVISION}

COPY config/opensearch.yml /etc/wazuh-indexer/opensearch.yml

RUN mkdir /etc/wazuh-indexer/certs && \
    tar -xf ./wazuh-cs.tar -C /etc/wazuh-indexer/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./admin.pem ./admin-key.pem ./root-ca.pem && \
    mv -n /etc/wazuh-indexer/certs/$NODE_NAME.pem /etc/wazuh-indexer/certs/indexer.pem && \
    mv -n /etc/wazuh-indexer/certs/$NODE_NAME-key.pem /etc/wazuh-indexer/certs/indexer-key.pem && \
    chmod 500 /etc/wazuh-indexer/certs && \
    chmod 400 /etc/wazuh-indexer/certs/* && \
    chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs

#CMD ["tail", "-f", "/dev/null"]

#COPY config/check_repository.sh /

#RUN chmod 775 /check_repository.sh && \
#    source /check_repository.sh

#RUN yum install wazuh-indexer-${WAZUH_VERSION}-${WAZUH_TAG_REVISION} -y && \
#    yum clean all

#COPY config/opensearch.yml /

#COPY config/config.sh .

#COPY config/config.yml /

#COPY config/action_groups.yml /

#COPY config/internal_users.yml /

#COPY config/roles_mapping.yml /

#COPY config/roles.yml /

#RUN bash config.sh

################################################################################
# Build stage 1 (the actual Wazuh indexer image):
#
# Copy wazuh-indexer from stage 0
# Add entrypoint

################################################################################
FROM amazonlinux:2023

ENV USER="wazuh-indexer" \
    GROUP="wazuh-indexer" \
    NAME="wazuh-indexer" \
    INSTALL_DIR="/usr/share/wazuh-indexer"

RUN yum install curl-minimal shadow-utils findutils hostname -y

RUN getent group $GROUP || groupadd -r -g 1000 $GROUP

RUN useradd --system \
            --uid 1000 \
            --no-create-home \
            --home-dir $INSTALL_DIR \
            --gid $GROUP \
            --shell /sbin/nologin \
            --comment "$USER user" \
            $USER

WORKDIR $INSTALL_DIR

#COPY config/entrypoint.sh /

#COPY config/securityadmin.sh /

#RUN chmod 700 /entrypoint.sh && chmod 700 /securityadmin.sh && \
#    mkdir -p /usr/share/wazuh-indexer && \
#    chown 1000:1000 /usr/share/wazuh-indexer && \
#    chown 1000:1000 /*.sh

COPY --from=builder --chown=1000:1000 /usr/share/wazuh-indexer /usr/share/wazuh-indexer
COPY --from=builder --chown=1000:1000 /etc/wazuh-indexer /usr/share/wazuh-indexer/config
COPY --from=builder --chown=1000:1000 /etc/wazuh-indexer /etc/wazuh-indexer
COPY --from=builder --chown=1000:1000 /etc/sysconfig/wazuh-indexer /etc/sysconfig/wazuh-indexer
#COPY --from=builder --chown=1000:1000 /debian/wazuh-indexer/usr/share/wazuh-indexer /usr/share/wazuh-indexer
#COPY --from=builder --chown=0:0 /debian/wazuh-indexer/usr/lib/systemd /usr/lib/systemd
#COPY --from=builder --chown=0:0 /debian/wazuh-indexer/usr/lib/sysctl.d /usr/lib/sysctl.d
#COPY --from=builder --chown=0:0 /debian/wazuh-indexer/usr/lib/tmpfiles.d /usr/lib/tmpfiles.d

RUN mkdir -p /var/lib/wazuh-indexer && chown 1000:1000 /var/lib/wazuh-indexer && \
    mkdir -p /usr/share/wazuh-indexer/logs && chown 1000:1000 /usr/share/wazuh-indexer/logs && \
    mkdir -p /run/wazuh-indexer && chown 1000:1000 /run/wazuh-indexer && \
    mkdir -p /var/log/wazuh-indexer && chown 1000:1000 /var/log/wazuh-indexer && \
    chmod 755 /usr/share/wazuh-indexer && \
    chmod 755 /usr/share/wazuh-indexer/config && \
    chmod 655 /usr/share/wazuh-indexer/config/jvm.options && \
    chmod 655 /usr/share/wazuh-indexer/config/opensearch.yml

USER wazuh-indexer

# Services ports
EXPOSE 9200

CMD ["tail", "-f", "/dev/null"]
#ENTRYPOINT ["/entrypoint.sh"]
# Dummy overridable parameter parsed by entrypoint
#CMD ["opensearchwrapper"]