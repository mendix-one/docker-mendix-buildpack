# Dockerfile to package Mendix project (build from mpr file to mda file)
ARG PACKAGE_BASE_IMAGE=ubuntu:noble
FROM ${PACKAGE_BASE_IMAGE} AS package

# Update package lists and install software-properties-common
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    apt-get install -y wget curl unzip tar vim && \
    rm -rf /var/lib/apt/lists/*

ARG JAVA_JDK_SOURCE_PATH=.
ARG JAVA_JDK_SOURCE_NAME=jdk-21_linux-x64_bin.deb
ARG MXDOCKER_SOURCE_PATH=..
ARG MXBUILD_SOURCE_HOST=https://cdn.mendix.com/runtime
ARG MXBUILD_SOURCE_FILE=mxbuild-9.24.40.80973.tar.gz

ARG MXBUILD_PATH=/opt/mxbuild
ARG MXDOCKER_PATH=/opt/mxdocker
ARG JAVA_HOME_PATH=/usr/lib/jvm/jdk-21.0.9-oracle-x64/
ARG PROJECT_PATH=/project
ARG PROJECT_FILENAME=SampleApp.mpr
ARG USER_UID=1001

# Check variable
RUN echo "PACKAGE_BASE_IMAGE: ${PACKAGE_BASE_IMAGE}" && \
    echo "JAVA_JDK_SOURCE_PATH: ${JAVA_JDK_SOURCE_PATH}" && \
    echo "JAVA_JDK_SOURCE_NAME: ${JAVA_JDK_SOURCE_NAME}" && \
    echo "MXDOCKER_SOURCE_PATH: ${MXDOCKER_SOURCE_PATH}" && \
    echo "MXBUILD_SOURCE_URL: ${MXBUILD_SOURCE_HOST}/${MXBUILD_SOURCE_FILE}" && \
    echo "MXBUILD_PATH: ${MXBUILD_PATH}" && \
    echo "MXDOCKER_PATH: ${MXDOCKER_PATH}" && \
    echo "JAVA_HOME_PATH: ${JAVA_HOME_PATH}" && \
    echo "PROJECT_PATH: ${PROJECT_PATH}" && \
    echo "PROJECT_NAME: ${PROJECT_NAME}" && \
    echo "USER_UID: ${USER_UID}"

# Install JDK
WORKDIR /tmp
COPY ${JAVA_JDK_SOURCE_PATH}/${JAVA_JDK_SOURCE_NAME} ./
RUN dpkg -i ./${JAVA_JDK_SOURCE_NAME}
RUN ${JAVA_HOME_PATH}/bin/java --version
RUN rm -rf ./${JAVA_JDK_SOURCE_NAME}

# Download mxbuild kit
RUN mkdir -p ${MXBUILD_PATH} && \
    cd ${MXBUILD_PATH} && \
    curl -LJO ${MXBUILD_SOURCE_HOST}/${MXBUILD_SOURCE_FILE} && \
    tar -zxvf ${MXBUILD_SOURCE_FILE} &&\
    rm -rf ${MXBUILD_SOURCE_FILE}

# Make MX Docker Patch
RUN mkdir -p ${MXDOCKER_PATH}
WORKDIR ${MXDOCKER_PATH}
COPY ${MXDOCKER_SOURCE_PATH}/* ./
RUN ls -la

RUN echo "${MXBUILD_PATH}/modeler/mxbuild --java-home=\"${JAVA_HOME_PATH}\" --java-exe-path=\"${JAVA_HOME_PATH}/bin/java\" ${PROJECT_PATH}/${PROJECT_NAME} && mkdir -p ${MXDOCKER_PATH}/build && copy -r ${PROJECT_PATH}/deployment/* ${MXDOCKER_PATH}/build/" > ${MXBUILD_PATH}/start

RUN chmod +x ${MXBUILD_PATH}/start
RUN chown -R ${USER_UID}:0 ${MXBUILD_PATH} && chmod -R g=u ${MXBUILD_PATH}
RUN chown -R ${USER_UID}:0 ${MXDOCKER_PATH} && chmod -R g=u ${MXDOCKER_PATH}

# Dockerfile
RUN mkdir -p ${PROJECT_PATH} && chown -R ${USER_UID}:0 ${PROJECT_PATH} && chmod -R g=u ${PROJECT_PATH}
WORKDIR ${MXDOCKER_PATH}

USER ${USER_UID}

VOLUME ${PROJECT_PATH}
ENV PROJECT_NAME="${PROJECT_FILENAME}"

ENTRYPOINT ["${MXBUILD_PATH}/start"]
