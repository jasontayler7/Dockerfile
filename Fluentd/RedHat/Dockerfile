FROM centos:latest

MAINTAINER maintainer "Abhishek <abhishek.dubey@opstree.com>"

# default syslog listener port
EXPOSE 10514
# monitor agent port
EXPOSE 24220
# default forwarder port
EXPOSE 24224
# default debug port
EXPOSE 24230
#default port for prometheus to pull metrics
EXPOSE 24231

USER root

ENV HOME=/opt/app-root/src \
    PATH=/opt/app-root/src/bin:/opt/app-root/bin:$PATH \
    RUBY_VERSION=2.0 \
    FLUENTD_VERSION=0.12.42 \
    GEM_HOME=/opt/app-root/src \
    SYSLOG_LISTEN_PORT=10514 \
    RUBYLIB=/opt/app-root/src/amqp_qpid/lib \
    RUBYVERREPOPKGS="centos-release-scl" \
    RUBYVERPKGS="rh-ruby22 scl-utils" \
    FLUENTD_OPT=""

RUN rpmkeys --import file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

RUN yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm ${RUBYVERREPOPKGS} && \
    yum update -y --setopt=tsflags=nodocs \
    && \
    mkdir -p ${HOME}/amqp_qpid \
    && \
    yum install -y --setopt=tsflags=nodocs \
        ruby rubygem-qpid_proton ${RUBYVERPKGS} \
    && \
    yum install -y --setopt=tsflags=nodocs --setopt=history_record=yes \
        gcc-c++ ruby-devel libcurl-devel make cmake swig autoconf automake libtool m4 \
    && \
    gem install -N --conservative --minimal-deps \
        'activesupport:<5' \
        'public_suffix:<3' \
        'elasticsearch-transport:<5' \
        'elasticsearch-api:<5' \
        'elasticsearch:<5' \
        fluentd:${FLUENTD_VERSION} \
        'fluent-plugin-elasticsearch:~>1.0' \
        'fluent-plugin-kubernetes_metadata_filter:1.0.1' \
        'fluent-plugin-prometheus:0.4.0' \
        'fluent-plugin-rewrite-tag-filter:<1.6.0' \
        fluent-plugin-parser \
        'fluent-plugin-grok-parser:<0.14' \
        fluent-plugin-secure-forward \
        'fluent-plugin-systemd:<0.1.0' \
        fluent-plugin-viaq_data_model \
        fluent-plugin-add \
        fluent-plugin-collectd-nest \
    && \
    yum -y history undo last \
    && \
    yum -y autoremove \
    && \
    yum -y install hostname \
    && \
    yum clean all

VOLUME /data

RUN  mkdir -p /etc/fluent/configs.d ${HOME}/forwarder-example && \
     mkdir -p /fluentd/plugins
     
COPY fluent.conf /etc/fluent/

WORKDIR ${HOME}
ADD run.sh ${HOME}/
RUN chmod +x ${HOME}/run.sh

CMD ${HOME}/run.sh