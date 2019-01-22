FROM centos:7.4.1708
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

ENV MOFED_VERSION 4.4-2.0.7.0

WORKDIR /

# Pick up some MOFED dependencies
RUN yum install -y install \
			--setopt=tsflags=nodocs \
			--disableplugin=fastestmirror \ \
	        wget \
	&& yum clean all \
	&& rm -rf /etc/ld.so.cache \
	&& rm -rf /sbin/sln \
	&& rm -rf /usr/{{lib,share}/locale,share/{man,doc,info,cracklib,i18n},{lib,lib64}/gconv,bin/localedef,sbin/build-locale-archive} \
	&& rm -rf /{root,tmp,var/cache/{ldconfig,yum}}/* \
	&& > /etc/sysconfig/i18n

# Download and install Mellanox OFED $MOFED_VERSION
RUN cd / && wget http://content.mellanox.com/ofed/MLNX_OFED-$MOFED_VERSION/MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64.tgz \
	&& tar -xzvf MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64.tgz \
	&& cd /MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64/RPMS \
	&& yum install -y install \
                        --setopt=tsflags=nodocs \
                        --disableplugin=fastestmirror \ \
                opensm-5*.x86_64.rpm opensm-libs-*.x86_64.rpm libibverbs-4*.x86_64.rpm librdmacm-4*.x86_64.rpm ucx-1*.x86_64.rpm libibcm-4*.x86_64.rpm mxm-*.x86_64.rpm hcoll-*.x86_64.rpm sharp-*.x86_64.rpm libmlx5-4*.x86_64.rpm \
        && yum clean all \
        && rm -rf /etc/ld.so.cache \
        && rm -rf /sbin/sln \
        && rm -rf /usr/{{lib,share}/locale,share/{man,doc,info,cracklib,i18n},{lib,lib64}/gconv,bin/localedef,sbin/build-locale-archive} \
        && rm -rf /{root,tmp,var/cache/{ldconfig,yum}}/* \
        && > /etc/sysconfig/i18n \
	&& cd / \
        && rm -rf MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64 \
        && rm -rf *.tgz \
	&& rm -rf /tmp/*

RUN echo -e \
		"routing_engine updn\\nsharp_enabled 2\\nvirt_enabled 2" \
		 > /etc/opensm/opensm.conf
