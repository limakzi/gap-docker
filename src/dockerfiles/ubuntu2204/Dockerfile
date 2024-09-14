ARG PLATFORM="linux/arm64"

## {{{
ARG GAP_BUILD="slim"
ARG GAP_VERSION="4.13.1"
## }}}

## {{{
ARG GAP_USER_USERNAME="gap"
ARG GAP_USER_GROUPNAME="gap"
ARG GAP_USER_UID=1000
ARG GAP_USER_GID=1000
ARG GAP_USER_SHELL="/bin/bash"
ARG GAP_USER_HOMEDIR="/opt/gap"
## }}}

## {{{
ARG GAP_DOWNLOAD_TARBALL_NAME="gap-${GAP_VERSION}.tar.gz"
ARG GAP_DOWNLOAD_URL="https://github.com/gap-system/gap/releases/download/v${GAP_VERSION}/${GAP_DOWNLOAD_TARBALL_NAME}"
## }}}




FROM --platform=${PLATFORM} ubuntu:22.04 AS base
ARG GAP_USER_GID
ARG GAP_USER_GROUPNAME
ARG GAP_USER_UID
ARG GAP_USER_USERNAME
ARG GAP_USER_HOMEDIR

RUN cat > /etc/apt/apt.conf.d/99norecommend <<EOF
APT::Install-Recommends "0";
APT::Install-Suggests "0";
EOF

RUN groupadd --system --gid ${GAP_USER_GID} ${GAP_USER_GROUPNAME} && \
     useradd --system --gid ${GAP_USER_GID} --home-dir ${GAP_USER_HOMEDIR} --create-home ${GAP_USER_USERNAME}


FROM base AS build
ARG GAP_BUILD
ARG GAP_VERSION
ARG GAP_DOWNLOAD_URL
ARG GAP_DOWNLOAD_TARBALL_NAME
ARG GAP_USER_HOMEDIR
ARG GAP_USER_USERNAME
ARG GAP_USER_GROUPNAME

RUN apt-get update --yes &&                               \
    apt-get install --no-install-recommends --quiet --yes \
        4ti2                                              \
        pari-gp                                           \
        autoconf                                          \
        build-essential                                   \
        libcdd-dev                                        \
        libcurl4-openssl-dev                              \
        libfplll-dev                                      \
        libgmp-dev                                        \
        libmpc-dev                                        \
        libmpfi-dev                                       \
        libmpfr-dev                                       \
        libncurses-dev                                    \
        libreadline-dev                                   \
        libzmq3-dev                                       \
        singular                                          \
        zlib1g-dev

USER ${GAP_USER_USERNAME}
WORKDIR ${GAP_USER_HOMEDIR}
ADD --chown=${GAP_USER_USERNAME}:${GAP_USER_GROUPNAME} ${GAP_DOWNLOAD_URL} ./
RUN tar -xzf ${GAP_DOWNLOAD_TARBALL_NAME}

WORKDIR "./gap-${GAP_VERSION}"
RUN ./configure && \
    MAKEFLAGS=-j8 make

RUN [[ ${GAP_BUILD} == "slim" ]] || cd ./pkg/ && ../bin/BuildPackages.sh

FROM base AS result
ARG GAP_VERSION
ARG GAP_USER_HOMEDIR
ARG GAP_USER_USERNAME
ARG GAP_USER_GROUPNAME

ENV PATH="${GAP_USER_HOMEDIR}/gap-${GAP_VERSION}/:${PATH}"
ENV LD_LIBRARY_PATH="${GAP_USER_HOMEDIR}/gap-${GAP_VERSION}/lib"

RUN apt-get update --yes &&                               \
    apt-get install --no-install-recommends --quiet --yes \
        4ti2                                              \
        libcdd0d                                          \
        libcurl4                                          \
        libgmp10                                          \
        libmpc3                                           \
        libmpfi0                                          \
        libmpfr6                                          \
        libncurses5                                       \
        libreadline8                                      \
        libzmq5                                           \
        pari-gp                                           \
        singular                                          \
        zlib1g

USER ${GAP_USER_USERNAME}
COPY --chown=${GAP_USER_USERNAME}:${GAP_USER_GROUPNAME} --from=build "${GAP_USER_HOMEDIR}/gap-${GAP_VERSION}/" "${GAP_USER_HOMEDIR}/gap-${GAP_VERSION}//"

CMD [ "bash" ]