ARG NV_VERSION=v5.4.2-beta.1
ARG BCI_VERSION=15.6
ARG SRCREPO=neuvector
FROM ${SRCREPO}/controller:${NV_VERSION} AS controller
FROM ${SRCREPO}/enforcer:${NV_VERSION} AS enforcer
FROM ${SRCREPO}/manager:${NV_VERSION} AS manager
FROM registry.suse.com/bci/bci-micro:${BCI_VERSION} AS builder

COPY --from=controller /licenses /chroot/licenses/
COPY --from=controller /usr/local/bin/ /chroot/usr/local/bin/
COPY --from=controller /etc/sysctl.conf /stage/etc/sysctl.conf

COPY --from=enforcer /licenses /chroot/licenses/
COPY --from=enforcer /usr/local/bin/ /chroot/usr/local/bin/
COPY --from=enforcer /etc/sysctl.conf /stage/etc/sysctl.conf

COPY --from=manager /licenses /chroot/licenses/
COPY --from=manager /usr/local/bin/ /chroot/usr/local/bin/
COPY --from=manager /usr/lib/jvm/java-17-openjdk/lib/security/java.security /chroot/usr/lib/jvm/java-17-openjdk/lib/security/java.security
COPY --from=manager /usr/lib/jvm/java-17-openjdk/lib/security/java.security /usr/lib64/jvm/java-17-openjdk-17/conf/security/java.security

#FROM registry.suse.com/bci/bci-micro:15.6
FROM neuvector/all_base:latest
ARG NV_VERSION

COPY --from=builder /chroot /
COPY supervisord.all.conf /etc/supervisor/conf.d/supervisord.conf

LABEL name="allinone" \
      vendor="SUSE Security" \
      version=${NV_VERSION} \
      release=${NV_VERSION} \
      neuvector.image="neuvector/allinone" \
      neuvector.role="controller+enforcer+manager"

ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
