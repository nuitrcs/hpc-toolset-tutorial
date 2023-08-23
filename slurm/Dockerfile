ARG HPCTS_VERSION=latest
FROM --platform=linux/amd64 ubccr/hpcts:base-${HPCTS_VERSION} as stage-amd64
FROM --platform=linux/arm64 ubccr/hpcts:base-${HPCTS_VERSION} as stage-arm64

ARG TARGETARCH

FROM stage-${TARGETARCH} as final

ARG SLURM_VERSION

ENV SLURM_VERSION=${SLURM_VERSION:-23.02.3-1}
ENV WEBSOCKIFY_VERSION=${WEBSOCKIFY_VERSION:-0.11.0}
ENV ARCHTYPE=`uname -m`

RUN curl -o /etc/yum.repos.d/turbovnc.repo https://turbovnc.org/pmwiki/uploads/Downloads/TurboVNC.repo
RUN dnf -y install dnf-plugins-core
RUN dnf -y config-manager --set-enabled powertools
RUN dnf -y module enable ruby:3.0 nodejs:14
RUN dnf install -y \
    @Development \
    munge \
    munge-devel \
    libcgroup \
    curl \
    bzip2 \
    readline-devel \
    numactl-devel \
    pam-devel \
    glib2-devel \
    hwloc-devel \
    openssl-devel \
    curl-devel \
    mariadb \
    turbovnc \
    mariadb-devel \
    python39 \
    python39-devel \
    python2-numpy \
    kitty-terminfo \
    stress

RUN alternatives --set python3 /usr/bin/python3.9

RUN wget -O /tmp/websockify-${WEBSOCKIFY_VERSION}.tar.gz https://github.com/novnc/websockify/archive/refs/tags/v${WEBSOCKIFY_VERSION}.tar.gz
WORKDIR /tmp
RUN tar xzf websockify-${WEBSOCKIFY_VERSION}.tar.gz
WORKDIR websockify-${WEBSOCKIFY_VERSION}
RUN python3 setup.py install
WORKDIR /
RUN rm -rf /tmp/websockify*

RUN curl -o /tmp/slurm-${SLURM_VERSION}.tar.bz2 https://download.schedmd.com/slurm/slurm-${SLURM_VERSION}.tar.bz2
WORKDIR /tmp
RUN tar xf slurm-${SLURM_VERSION}.tar.bz2
WORKDIR slurm-${SLURM_VERSION}
RUN ./configure --prefix=/usr --sysconfdir=/etc/slurm 
RUN make -j4
RUN make install
RUN install -D -m644 etc/cgroup.conf.example /etc/slurm/cgroup.conf.example
RUN install -D -m644 etc/slurm.conf.example /etc/slurm/slurm.conf.example
RUN install -D -m644 etc/slurmdbd.conf.example /etc/slurm/slurmdbd.conf.example
RUN install -D -m644 contribs/slurm_completion_help/slurm_completion.sh /etc/profile.d/slurm_completion.sh
WORKDIR /
rm -rf /tmp/slurm*

RUN groupadd -r --gid=1000 slurm
RUN useradd -r -g slurm --uid=1000 slurm

RUN mkdir /etc/sysconfig/slurm \
    /var/spool/slurmd \
    /var/run/slurmd \
    /var/run/slurmdbd \
    /var/lib/slurmd \
    /var/log/slurm \
    /data

RUN touch /var/lib/slurmd/node_state \
    /var/lib/slurmd/front_end_state \
    /var/lib/slurmd/job_state \
    /var/lib/slurmd/resv_state \
    /var/lib/slurmd/trigger_state \
    /var/lib/slurmd/assoc_mgr_state \
    /var/lib/slurmd/assoc_usage \
    /var/lib/slurmd/qos_usage \
    /var/lib/slurmd/fed_mgr_state

RUN chown -R slurm:slurm /var/*/slurm*

RUN /sbin/create-munge-key

RUN dnf install -y pcp

RUN mkdir -p /run/pcp
RUN ln -s /usr/lib/systemd/system/pmlogger.service /etc/systemd/system/multi-user.target.wants/pmlogger.service

RUN echo -e "# Disable Avahi (since it does not run inside the containers)\n-A" >> /etc/pcp/pmcd/pmcd.options

RUN sed -i 's#^LOCALHOSTNAME.*$#LOCALHOSTNAME   y   n   "/home/pcp/$(date +%Y)/$(date +%m)/LOCALHOSTNAME/$(date +%Y)-$(date +%m)-$(date +%d)"   -r -c /etc/pcp/pmlogger/pmlogger-supremm.config#' /etc/pcp/pmlogger/control.d/local

RUN dnf clean all
RUN rm -rf /var/cache/dnf

COPY slurm.conf /etc/slurm/slurm.conf
COPY --chown=slurm:slurm slurmdbd.conf /etc/slurm/slurmdbd.conf
RUN chmod 600 /etc/slurm/slurmdbd.conf
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
COPY pmlogger-supremm.config /etc/pcp/pmlogger/pmlogger-supremm.config
COPY slurm-prolog.sh /usr/local/bin/slurm-prolog.sh
COPY slurm-epilog.sh /usr/local/bin/slurm-epilog.sh
COPY submit_jobs.sh /usr/local/bin/submit_jobs.sh
COPY example_job.sbatch /usr/local/bin/example_job.sbatch
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
