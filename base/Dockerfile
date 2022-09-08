
## Dockerfile for constructing the base Open Education Effort (OPE)
## container.  This container contains everything required for authoring OPE courses
## Well is should anyway ;-)

ARG FROM_REG
ARG FROM_IMAGE
ARG FROM_TAG
FROM ${FROM_REG}${FROM_IMAGE}${FROM_TAG} AS stage-one

ARG ADDITIONAL_DISTRO_PACKAGES
ARG BUILD_SRC
ARG MOUNT_DIR
ARG JUPYTER_ENABLE_EXTENSIONS

LABEL name="s2i-odh-ope-base" \
      version="latest" \
      summary="Open Education Effort Customized Jupyter Notebook Source-to-Image for Python 3.9 applications." \
      description="Notebook image based on Source-to-Image.These images can be used in OpenDatahub JupterHub." \
      io.k8s.description="Notebook image based on Source-to-Image.These images can be used in OpenDatahub JupterHub." \
      io.k8s.display-name="Custom Notebook Python 3.8 S2I" \
      io.openshift.expose-services="8888:http" \
      io.openshift.tags="python,python38" \
      io.openshift.s2i.build.commit.ref="container" \
      io.openshift.s2i.build.source-location="https://github.com/OPEFFORT/ope"

ENV XDG_CACHE_HOME="/opt/app-root/src/.cache" \
    THAMOS_RUNTIME_ENVIRONMENT="" \
    UPGRADE_PIP_TO_LATEST="1" \
    WEB_CONCURRENCY="1" \
    THOTH_ADVISE="0" \
    THOTH_ERROR_FALLBACK="1" \
    THOTH_DRY_RUN="1" \
    THAMOS_DEBUG="0" \
    THAMOS_VERBOSE="1" \
    THOTH_PROVENANCE_CHECK="0"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Add a "USER root" statement followed by RUN statements to install system packages using apt-get,
# change file permissions, etc.

# install linux packages that we require for systems classes
USER root

# Insert your changes here.
RUN yum -y install ${ADDITIONAL_DISTRO_PACKAGES}

# build-essential
RUN yum update --assumeyes --nobest --setopt=tsflags=nodocs && \
    yum --assumeyes clean all && \
    rm -rf /var/cache/dnf

# creating a new user for student



# Installing some system packages from source
COPY builder /tmp

RUN cd /tmp && chmod +x builder && ./builder ${BUILD_SRC}

#  adding moria for old time sake
#  sed '36d' CMakeLists.txt -> Ref: https://github.com/dungeons-of-moria/umoria/issues/44
RUN cd /tmp && wget https://github.com/dungeons-of-moria/umoria/archive/refs/tags/v5.7.15.tar.gz \
    && tar -zxf v5.7.15.tar.gz && cd umoria-5.7.15 && sed -i '36d' CMakeLists.txt \
    && cmake . && make \
    && mv umoria /opt/umoria \
    && ln -s /opt/umoria/umoria /opt/umoria/moria \
    && cd /tmp && rm -rf v5.7.15 && rm v5.7.15.tar.gz

ENV PATH=$PATH:/opt/umoria

# adding our custom OPE version of RISE for lab -- remove once main RISE is fixed
RUN curl -sL https://dl.yarnpkg.com/rpm/yarn.repo -o /etc/yum.repos.d/yarn.repo && \
    dnf -y install yarn


RUN git clone https://github.com/OPEFFORT/RISE.git /home/RISE  && \
    cd /home/RISE && git checkout dev && \
    yarn install && yarn run build && \
    pip install -e .


ARG FROM_REG
ARG FROM_IMAGE
ARG FROM_TAG

FROM ${FROM_REG}${FROM_IMAGE}${FROM_TAG} AS stage-two

COPY --from=stage-one /usr/local/bin /usr/local/bin
COPY --from=stage-one /opt/umoria /opt/umoria
COPY --from=stage-one /home/RISE /home/RISE
COPY --from=stage-one /opt/app-root/lib/python3.8/site-packages/ /opt/app-root/lib/python3.8/site-packages/

LABEL name="s2i-odh-ope-base" \
      version="latest" \
      summary="Open Education Effort Customized Jupyter Notebook Source-to-Image for Python 3.9 applications." \
      description="Notebook image based on Source-to-Image.These images can be used in OpenDatahub JupterHub." \
      io.k8s.description="Notebook image based on Source-to-Image.These images can be used in OpenDatahub JupterHub." \
      io.k8s.display-name="Custom Notebook Python 3.8 S2I" \
      io.openshift.expose-services="8888:http" \
      io.openshift.tags="python,python38" \
      io.openshift.s2i.build.commit.ref="container" \
      io.openshift.s2i.build.source-location="https://github.com/OPEFFORT/ope"

ENV XDG_CACHE_HOME="/opt/app-root/src/.cache" \
    THAMOS_RUNTIME_ENVIRONMENT="" \
    UPGRADE_PIP_TO_LATEST="1" \
    WEB_CONCURRENCY="1" \
    THOTH_ADVISE="0" \
    THOTH_ERROR_FALLBACK="1" \
    THOTH_DRY_RUN="1" \
    THAMOS_DEBUG="0" \
    THAMOS_VERBOSE="1" \
    THOTH_PROVENANCE_CHECK="0"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]


ENV PATH=$PATH:/opt/umoria


ARG MOUNT_DIR
ARG JUPYTER_ENABLE_EXTENSIONS

USER root

# FIXME: This probably should go back to being a seperate image but to ease bootstrap
# have merged this into the base image
# Un-minimize the system -- aka add documentation and man pages back to the container
# To ensure a more complete UNIX user experience
# from http://docs.projectatomic.io/container-best-practices/
RUN [ -e /etc/dnf/dnf.conf ] && sed -i '/tsflags=nodocs/d' /etc/dnf/dnf.conf || true
RUN dnf -y reinstall "*"

# Change default user to student
#RUN sed -i 's|^default.*|default:x:1001:0:Default Student User:/home/default:/bin/bash|' /etc/passwd

RUN useradd -g root -m -d /home/student -s /bin/bash student
RUN sed -i 's|^\([[:blank:]]*echo.*\)|    echo "student:x:${USER_ID}:0:Default Student User:/home/student:/bin/bash" >> $NSS_WRAPPER_PASSWD|' ${MOUNT_DIR}/../etc/generate_container_user

# copy overrides.json
COPY settings ${MOUNT_DIR}/../share/jupyter/lab/settings
RUN fix-permissions ${MOUNT_DIR}/../share/jupyter/lab/settings

# clean up cache
RUN rm -rf /var/cache/dnf

# Copying in override assemble/run scripts
COPY .s2i/bin /tmp/scripts
# Copying in source code
COPY . /tmp/src
COPY .thoth.yaml /opt/app-root/src/
COPY .thoth.yaml /opt/app-root
COPY Pipfile Pipfile.lock /opt/app-root/src/

# Change file ownership to the assemble user. Builder image must support chown command.
RUN chown -R student:0 /tmp/scripts /tmp/src /opt/app-root
USER root

RUN /tmp/scripts/assemble
CMD /tmp/scripts/run
