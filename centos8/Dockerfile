FROM udienz/buildpack:centos8
LABEL maintainer="Mahyuddin Susanto"

ARG user=jenkins
ARG group=jenkins
ARG uid=1001
ARG gid=1001
ARG JENKINS_AGENT_HOME=/home/jenkins

WORKDIR /lib/systemd/system/sysinit.target.wants
# Install systemd -- See https://hub.docker.com/_/centos/
RUN yum -y update; yum clean all; \
(for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;
WORKDIR /
RUN yum update -y \
    && yum makecache timer \
    && yum -y install epel-release \
    && yum -y update \
    && yum -y install sudo which python3-pip libselinux-python3 git git-lfs  \
    && yum clean all \
    && pip3 install --upgrade --no-cache-dir pip

# Disable requiretty.
RUN sed -i -e "s/^\(Defaults\s*requiretty\)/#--- \1/"  /etc/sudoers \
    && groupadd -g ${gid} ${group} \
    && useradd -l -d "${JENKINS_AGENT_HOME}" -u "${uid}" -g "${gid}" -m \
    -s /bin/bash "${user}" \
    && echo "Defaults:${user} !requiretty" >> /etc/sudoers \
    && echo "${user} ALL=NOPASSWD: ALL" >> /etc/sudoers.d/${user}

USER jenkins
RUN export LC_ALL=en_US.UTF-8 \
    && echo "LC_ALL=en_US.UTF-8" >> $HOME/.bashrc \
    && export PATH="$HOME/bin:$HOME/.local/bin:$PATH" \
    && echo 'export PATH=$HOME/bin:$HOME/.local/bin:$PATH' >> $HOME/.bashrc \
    && python3 -m pip install --no-cache-dir --upgrade pip \
   && pip3 install --no-cache-dir ansible ansible-lint molecule-docker

USER root
# Install Ansible inventory file.
RUN mkdir -p /etc/ansible \
    && echo -e '[local]\nlocalhost ansible_connection=local' > /etc/ansible/hosts


VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/lib/systemd/systemd"]
