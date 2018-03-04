FROM fedora:26
MAINTAINER "Ari LiVigni" <ari@redhat.com>

ENV container docker
ENV HOME=/home

WORKDIR $HOME

# Beaker-client required
COPY beaker-client.repo /etc/yum.repos.d/

# Install all package requirements
RUN for i in {1..5} ; do dnf -y update && dnf clean all && break || sleep 10 ; done
RUN for i in {1..5} ; do dnf -y install ansible \
        python2-junit_xml.noarch \
        python3-junit_xml.noarch \
        beaker-client \
        && dnf clean all \
        && break || sleep 10 ; done

VOLUME [ "/sys/fs/cgroup"]
CMD ["/usr/sbin/init"]