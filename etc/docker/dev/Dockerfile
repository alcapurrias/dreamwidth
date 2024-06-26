#
# This creates a Dreamwidth worker container.
#

FROM  ghcr.io/dreamwidth/base:latest
LABEL org.opencontainers.image.authors="Mark Smith <mark@dreamwidth.org>"

# Configuration can go here.
ARG COMMIT=main

# Things that commands need, but shouldn't change.
ENV HOME /dw
ENV LJHOME /dw
ENV PERL5LIB /dw/extlib/lib/perl5

# Install some packages we need in workers only.
RUN echo $COMMIT && \
    apt-get update && \
    apt-get install -y mysql-client && \
    curl https://raw.githubusercontent.com/dreamwidth/dreamwidth/main/doc/dependencies-system | \
        xargs apt-get -y install && \
    curl https://raw.githubusercontent.com/dreamwidth/dreamwidth/main/doc/dependencies-dev | \
        xargs apt-get -y install && \
    curl https://raw.githubusercontent.com/dreamwidth/dreamwidth/main/doc/dependencies-cpanm | \
        cpm install -v --show-build-log-on-failure --no-color -L /dw/extlib/ - && \
    rm -rf /root/.cpanm && \
    rm -rf /root/.perl-cpm && \
    rm -rf /var/lib/apt/lists/*

# Actually check out the source code now.
RUN git -C $LJHOME fetch && \
    git -C $LJHOME checkout $COMMIT && \
    git -C $LJHOME pull --ff-only origin $COMMIT

# Copy in support scripts/configurations that are useful.
ADD scripts/ /opt/

# Setup script that runs to make sure we get configs in the right place and do any
# last minute configuration.
RUN bash /opt/setup.sh

# Kick off the startup script, which does some healthcheck and then starts
# Apache if things look good.
CMD bash /opt/startup.sh
