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
COPY --from=manager /usr/lib64/jvm/java-17-openjdk-17/conf/security/java.security.fips /chroot/usr/lib64/jvm/java-17-openjdk-17/conf/security/java.security.fips
COPY --from=manager /entrypoint.sh /chroot/entrypoint.sh

#
# Base image
#
FROM registry.suse.com/bci/bci-micro:${BCI_VERSION} AS micro
FROM registry.suse.com/bci/bci-base:${BCI_VERSION} AS base

ENV JAVA_VERSION=17.0.12_p8-r0 \
    JAVA_HOME=/usr/lib64/jvm/jre-17-openjdk/bin \
    PATH=$PATH:/usr/lib64/jvm/jre-17-openjdk/bin \
    LANG=C.UTF-8 \
    PYTHONUNBUFFERED=1

COPY --from=micro / /chroot/
RUN zypper --installroot /chroot -n --gpg-auto-import-keys in --no-recommends \
    java-17-openjdk python312 python312-pip \
    ca-certificates iproute2 ethtool lsof procps curl jq iptables grep tar awk tcpdump sed kmod wget unzip \
    libnetfilter_queue-devel liburcu-devel libpcap-devel pcre2-devel libjansson-devel libmnl-devel jemalloc-devel

RUN zypper addrepo https://download.opensuse.org/repositories/isv:SUSE:neuvector/15.6/isv:SUSE:neuvector.repo && \
    zypper --installroot /chroot -n --gpg-auto-import-keys refresh && \
    zypper --installroot /chroot install -y yq vectorscan-devel

RUN cp /etc/resolv.conf /chroot/etc/resolv.conf && \
    chroot /chroot /usr/bin/python3.12 -m pip install --upgrade pip setuptools && \
    rm /chroot/usr/lib/python3.12/site-packages/distutils-precedence.pth && \
    zypper --installroot /chroot clean -a && \
    rm -rf /chroot/var/log/

RUN cd /chroot/usr/bin/ && rm -rf basename chcon chgrp chmod chown chroot cksum dd df dircolors dirname du install install-info join locale localedef mkdir mkfifo mknod mktemp paste pathchk readlink realpath sync smidiff smidump smilink smiquery smistrip smixlate tee tiemout tload top truncate unlink watch

RUN touch /chroot/usr/local/bin/.nvcontainer

FROM micro
WORKDIR /
ARG NV_VERSION

COPY requirements.txt /
COPY --from=base /chroot /
COPY --from=base /usr/sbin/useradd /usr/sbin
COPY --from=builder /chroot /

COPY supervisord.all.conf /etc/supervisor/conf.d/supervisord.conf

RUN ln -s /usr/bin/python3.12 /usr/bin/python && \
    ln -s /usr/bin/python3.12 /usr/bin/python3  && \
    pip3 install --no-cache-dir "supervisor==4.2.5" && \
    pip3 install --no-cache-dir -r /requirements.txt && \
    ln -s /usr/local/bin/supervisord /usr/bin/supervisord && \
    rm -r /root/.cache /requirements.txt

RUN ln -s /usr/lib64/libpcap.so /usr/lib64/libpcap.so.0.8 && \
    ln -s /usr/lib64/libpcre.so.1 /usr/lib64/libpcre.so.3

LABEL name="allinone" \
      vendor="SUSE Security" \
      version=${NV_VERSION} \
      release=${NV_VERSION} \
      neuvector.image="neuvector/allinone" \
      neuvector.role="controller+enforcer+manager"

ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
