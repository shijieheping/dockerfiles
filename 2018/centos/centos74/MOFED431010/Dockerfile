FROM centos:7.4.1708
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

ENV MOFED_VERSION 4.3-1.0.1.0

WORKDIR /

# Pick up some MOFED dependencies
RUN yum install -y install \
			--setopt=tsflags=nodocs \
			--disableplugin=fastestmirror \ \
	        wget \
	        perl \
        	lsb-release \
		numactl-libs gtk2 atk cairo gcc-gfortran tcsh libnl3 tcl tk \
	&& yum clean all \
	&& rm -rf /etc/ld.so.cache \
	&& rm -rf /sbin/sln \
	&& rm -rf /usr/{{lib,share}/locale,share/{man,doc,info,cracklib,i18n},{lib,lib64}/gconv,bin/localedef,sbin/build-locale-archive} \
	&& rm -rf /{root,tmp,var/cache/{ldconfig,yum}}/* \
	&& > /etc/sysconfig/i18n

# Download and install Mellanox OFED $MOFED_VERSION
RUN wget http://content.mellanox.com/ofed/MLNX_OFED-$MOFED_VERSION/MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64.tgz \
	&& tar -xzvf MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64.tgz \
	&& yes y | head -1 | MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64/mlnxofedinstall --user-space-only --without-fw-update --all -vvv \
	&& cd .. \
        && rm -rf MLNX_OFED_LINUX-$MOFED_VERSION-rhel7.4-x86_64 \
        && rm -rf *.tgz \
	&& rm -rf /tmp/*

RUN echo -e \
		"routing_engine updn\\nsharp_enabled 2\\nvirt_enabled 2" \
		 > /etc/opensm/opensm.conf
