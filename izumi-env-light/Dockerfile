ARG V_UBUNTU=22.04
#ARG URELEASE=jammy
FROM ubuntu:${V_UBUNTU}

ARG V_GRAAL=22.3.0
ARG V_JAVA=java17

ARG V_DOTNETSDK=6.0

RUN mkdir -p /work
RUN mkdir -p /root
VOLUME /work
VOLUME /root
WORKDIR /work

ENV DEBIAN_FRONTEND noninteractive
ENV JAVA_HOME ${JAVA_BASE}/graalvm
ENV PATH "$PATH:$JAVA_HOME/bin:/app:/.dotnet/tools"

RUN apt-get update
RUN apt-get -y install locales
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && \
  locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN echo "APT::Acquire::Queue-Mode "access"; APT::Acquire::Retries 3; " > /etc/apt/apt.conf.d/99parallel && \
  export DEBIAN_FRONTEND=noninteractive && \
  apt-get update && apt-get install -y --no-install-recommends tzdata gnupg ca-certificates wget curl zip unzip apt-utils jq openssh-client software-properties-common && \
  ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  apt-get upgrade -y && apt-get clean -y

# java
ARG JAVA_BASE=/usr/lib/jvm

RUN cd /tmp && \
  wget -q https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-${V_GRAAL}/graalvm-ce-${V_JAVA}-linux-amd64-${V_GRAAL}.tar.gz -O dist.tar.gz && \
  tar -xvzf dist.tar.gz && \
  rm -f dist.tar.gz && \
  mkdir -p ${JAVA_BASE} && \
  chmod -R 755 ${JAVA_BASE} && \
  mv graalvm-*/ ${JAVA_HOME} && \
  update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 1 && \
  gu install nodejs && \
  gu install js

# dotnet
ENV DOTNET_CLI_TELEMETRY_OPTOUT=1
RUN printf 'Package: *\nPin: origin "packages.microsoft.com"\nPin-Priority: 1001' > /etc/apt/preferences.d/99microsoft-dotnet.pref

RUN curl -o packages-microsoft-prod.deb https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb && \
  dpkg -i packages-microsoft-prod.deb && \
  rm packages-microsoft-prod.deb

RUN apt-get update && apt-get install -y apt-transport-https dotnet-sdk-${V_DOTNETSDK} aspnetcore-runtime-${V_DOTNETSDK} && apt-get upgrade -y && apt-get clean -y

RUN dotnet --version && \
  dotnet tool install Amazon.Lambda.Tools --tool-path /.dotnet/tools && \
  dotnet new -i Amazon.Lambda.Templates

