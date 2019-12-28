ARG TAG=latest
FROM ubuntu:${TAG}

LABEL maintainer "Alan Young <harleypig@harleypig.com>"

SHELL ["/bin/bash", "-c"]

#-----------------------------------------------------------------------------
ARG TESTUSER
RUN : "${TESTUSER:?TESTUSER not set}"

#-----------------------------------------------------------------------------
RUN apt-get install ssh
RUN systemctl ssh start
RUN systemctl ssh enable
RUN systemctl ssh status

#-----------------------------------------------------------------------------
RUN useradd --create-home --home-dir /home/${TESTUSER} --shell /bin/bash ${TESTUSER}

USER ${TESTUSER}
WORKDIR /home/${TESTUSER}

COPY --chown=${TESTUSER} testfiles/ ./
