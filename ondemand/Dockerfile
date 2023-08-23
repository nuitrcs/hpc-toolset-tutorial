ARG HPCTS_VERSION=latest

FROM --platform=linux/amd64 ubccr/hpcts:slurm-${HPCTS_VERSION} as stage-amd64
RUN dnf install -y https://yum.osc.edu/ondemand/3.0/ondemand-release-web-3.0-1.noarch.rpm
RUN dnf install -y netcat ondemand ondemand-dex

FROM --platform=linux/arm64 ubccr/hpcts:slurm-${HPCTS_VERSION} as stage-arm64
RUN dnf install -y file lsof sudo gcc gcc-c++ git \
        patch lua-posix rsync ruby ruby-devel python2 python3 \
        nodejs sqlite sqlite-devel nmap-ncat httpd httpd-devel mod_ssl \
        libcurl-devel autoconf openssl-devel jansson-devel libxml2-devel \
        libxslt-devel gd-devel libaio-devel
RUN gem install rake dotenv bcrypt
COPY . /build
RUN /build/install-dex-arm64.sh
RUN /build/install-passenger-arm64.sh
RUN /build/install-ood-arm64.sh
RUN rm -rf /build

ARG TARGETARCH

FROM stage-${TARGETARCH} as final

COPY . /build
RUN /build/install.sh && rm -rf /build
COPY cluster-config.yml /etc/ood/config/clusters.d/quests.yml
RUN git clone https://github.com/nuitrcs/quest_ood_jupyter.git /var/www/ood/apps/sys/quest_ood_jupyter
COPY Quest-OpenOnDemand-Configs/config/apps/dashboard/motd /etc/ood/config/apps/dashboard/motd
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
COPY Quest-OpenOnDemand-Configs/config/apps/dashboard/initializers /etc/ood/config/apps/dashboard/initializers
COPY themes/hpc-coop /usr/share/ondemand-dex/web/themes/hpc-coop
COPY Quest-OpenOnDemand-Configs/config/apps/dashboard/env /etc/ood/config/apps/dashboard/env
COPY Quest-OpenOnDemand-Configs/config/ondemand.d/ondemand.yml /etc/ood/config/ondemand.d/ondemand.yml
COPY Quest-OpenOnDemand-Configs/config/apps/dashboard/views/widgets /etc/ood/config/apps/dashboard/views/widgets

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
