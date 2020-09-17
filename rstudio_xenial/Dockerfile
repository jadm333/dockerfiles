FROM rstudio/r-base:4.0-xenial

ENV RSTUDIO_VERSION="rstudio-server-1.3.1073-amd64.deb"
ENV RSTUDIO_URL="https://download2.rstudio.org/server/xenial/amd64/${RSTUDIO_VERSION}"
ARG S6_VERSION
ARG PANDOC_TEMPLATES_VERSION
ENV S6_VERSION=${S6_VERSION:-v1.21.7.0}
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2
ENV PATH=/usr/lib/rstudio-server/bin:$PATH
ENV PANDOC_TEMPLATES_VERSION=${PANDOC_TEMPLATES_VERSION:-2.9}

RUN apt-get update -qq
RUN apt-get install -y --no-install-recommends \
    file \
    git \
    libapparmor1 \
    libclang-dev \
    libcurl4-openssl-dev \
    libedit2 \
    libssl-dev \
    lsb-release \
    multiarch-support \
    psmisc \
    procps \
    python-setuptools \
    sudo \
    wget \
    gdebi-core \
    libv8-dev

# Install Rstudio server
RUN wget -q $RSTUDIO_URL
RUN gdebi -n $RSTUDIO_VERSION
RUN rm rstudio-server-*-amd64.deb
# PANDOC
# Symlink pandoc & standard pandoc templates for use system-wide
RUN git clone --recursive --branch ${PANDOC_TEMPLATES_VERSION} https://github.com/jgm/pandoc-templates \
    && mkdir -p /opt/pandoc/templates \
    && cp -r pandoc-templates*/* /opt/pandoc/templates && rm -rf pandoc-templates* \
    && mkdir /root/.pandoc && ln -s /opt/pandoc/templates /root/.pandoc/templates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/

RUN mkdir -p /etc/R \
    && echo '\n\
    \n# Configure httr to perform out-of-band authentication if HTTR_LOCALHOST \
    \n# is not set since a redirect to localhost may not work depending upon \
    \n# where this Docker container is running. \
    \noptions(repos=structure(c(CRAN="https://cran.rstudio.com"))) \
    \nif(is.na(Sys.getenv("HTTR_LOCALHOST", unset=NA))) { \
    \n  options(httr_oob_default = TRUE) \
    \n}' >> /usr/lib/R/etc/Rprofile.site \
    && echo "PATH=${PATH}" >> /usr/lib/R/etc/Renviron \
    ## Need to configure non-root user for RStudio
    && useradd rstudio \
    && echo "rstudio:rstudio" | chpasswd \
    	&& mkdir /home/rstudio \
    	&& chown rstudio:rstudio /home/rstudio \
    	&& addgroup rstudio staff \
    ## Prevent rstudio from deciding to use /usr/bin/R if a user apt-get installs a package
    &&  echo 'rsession-which-r=/usr/bin/R' >> /etc/rstudio/rserver.conf \
    ## use more robust file locking to avoid errors when using shared volumes:
    && echo 'lock-type=advisory' >> /etc/rstudio/file-locks \
    ## configure git not to request password each time
    && git config --system credential.helper 'cache --timeout=3600' \
    && git config --system push.default simple 

# Install R packages
## Use littler installation scripts
RUN Rscript -e "install.packages(c('littler', 'docopt'), repo = 'https://cran.rstudio.com')" \
  && ln -s /usr/lib/R/library/littler/examples/install2.r /usr/local/bin/install2.r \
  && ln -s /usr/lib/R/library/littler/examples/installGithub.r /usr/local/bin/installGithub.r \
  && ln -s /usr/lib/R/library/littler/bin/r /usr/local/bin/r

RUN apt-get update -qq && apt-get -y --no-install-recommends install \
  libxml2-dev \
  libcairo2-dev \
  libsqlite-dev \
  libpq-dev \
  libssh2-1-dev \
  unixodbc-dev \
  libsasl2-dev \
  && install2.r --error \
    --deps TRUE \
    -l /usr/lib/R/library \
    tidyverse \
    devtools

RUN install2.r --error \
    --deps TRUE \
    rstan \
    rstantools

## Set up S6 init system
RUN wget -P /tmp/ https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-amd64.tar.gz \
  && tar xzf /tmp/s6-overlay-amd64.tar.gz -C / \
  && mkdir -p /etc/services.d/rstudio \
  && echo '#!/usr/bin/with-contenv bash \
          \n## load /etc/environment vars first: \
  		  \n for line in $( cat /etc/environment ) ; do export $line ; done \
          \n exec /usr/lib/rstudio-server/bin/rserver --server-daemonize 0' \
          > /etc/services.d/rstudio/run \
  && echo '#!/bin/bash \
          \n rstudio-server stop' \
          > /etc/services.d/rstudio/finish \
  && mkdir -p /home/rstudio/.rstudio/monitored/user-settings \
  && echo 'alwaysSaveHistory="0" \
          \nloadRData="0" \
          \nsaveAction="0"' \
          > /home/rstudio/.rstudio/monitored/user-settings/user-settings \
  && chown -R rstudio:rstudio /home/rstudio/.rstudio

COPY userconf.sh /etc/cont-init.d/userconf
COPY disable_auth_rserver.conf /etc/rstudio/disable_auth_rserver.conf
COPY pam-helper.sh /usr/lib/rstudio-server/bin/pam-helper



EXPOSE 8787

CMD ["/init"]