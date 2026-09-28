FROM ubuntu:16.04 
ENV ZK_USER=zookeeper \
ZK_DATA_DIR=/var/lib/zookeeper/data \
ZK_DATA_LOG_DIR=/var/lib/zookeeper/log \
ZK_LOG_DIR=/var/log/zookeeper \
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

ARG ZK_VERSION=3.7.1
# ZooKeeper 3.5+ publishes apache-zookeeper-<version>-bin.tar.gz inside a
# zookeeper-<version>/ directory, unlike the 3.4.x zookeeper-<version>.tar.gz
# this file was written for. The two names differ, hence both args.
ARG ZK_DIST=apache-zookeeper-${ZK_VERSION}-bin
# Apache moves releases out of /dist once they are superseded, so the original
# www.apache.org/dist URL now 404s. Archived releases live here permanently.
ARG ZK_MIRROR=https://archive.apache.org/dist/zookeeper
# The 3.7.1 -bin tarball ships only bin/ conf/ docs/ lib/ plus licences and readmes,
# far less than the 3.4.x source tarball, so the cleanup below is much shorter.
# LICENSE.txt and NOTICE.txt are kept.
RUN set -x \
    && apt-get update \
    && apt-get install -y openjdk-8-jre-headless wget netcat-openbsd ca-certificates gnupg \
	&& wget -q "$ZK_MIRROR/zookeeper-$ZK_VERSION/$ZK_DIST.tar.gz" \
    && wget -q "$ZK_MIRROR/zookeeper-$ZK_VERSION/$ZK_DIST.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && wget -qO- "https://archive.apache.org/dist/zookeeper/KEYS" | gpg --batch --import - \
    && gpg --batch --verify "$ZK_DIST.tar.gz.asc" "$ZK_DIST.tar.gz" \
    && tar -xzf "$ZK_DIST.tar.gz" -C /opt \
    && rm -r "$GNUPGHOME" "$ZK_DIST.tar.gz" "$ZK_DIST.tar.gz.asc" \
    && ln -s /opt/$ZK_DIST /opt/zookeeper \
    && rm -rf /opt/zookeeper/docs \
    /opt/zookeeper/README.md \
    /opt/zookeeper/README_packaging.md \
	&& apt-get autoremove -y wget \
	&& rm -rf /var/lib/apt/lists/*

#Copy configuration generator script to bin
COPY scripts /opt/zookeeper/bin/

# Create a user for the zookeeper process and configure file system ownership
# for nessecary directories and symlink the distribution as a user executable.
# 3.5+ has no top-level $ZK_DIST.jar to symlink; every jar lives under lib/.
RUN set -x \
	&& useradd $ZK_USER \
    && [ `id -u $ZK_USER` -eq 1000 ] \
    && [ `id -g $ZK_USER` -eq 1000 ] \
    && mkdir -p $ZK_DATA_DIR $ZK_DATA_LOG_DIR $ZK_LOG_DIR /usr/share/zookeeper /tmp/zookeeper /usr/etc/ \
	&& chown -R "$ZK_USER:$ZK_USER" /opt/$ZK_DIST $ZK_DATA_DIR $ZK_LOG_DIR $ZK_DATA_LOG_DIR /tmp/zookeeper \
	&& ln -s /opt/zookeeper/conf/ /usr/etc/zookeeper \
	&& ln -s /opt/zookeeper/bin/* /usr/bin \
	&& ln -s /opt/zookeeper/lib/* /usr/share/zookeeper
