ARG V_UBUNTU=22.04
FROM ubuntu:${V_UBUNTU}
ARG URELEASE=jammy

###
ARG V_GRAAL=21.0.2
ARG V_DOTNETSDK=7.0
###

# basic configuration
RUN mkdir -p /work
RUN mkdir -p /root
VOLUME /work
VOLUME /root
WORKDIR /work

ENV DEBIAN_FRONTEND noninteractive
RUN echo "APT::Acquire::Queue-Mode "access"; APT::Acquire::Retries 3; " > /etc/apt/apt.conf.d/99parallel

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y --no-install-recommends tzdata gnupg ca-certificates wget curl zip unzip apt-utils jq openssh-client software-properties-common

RUN apt-get -y --no-install-recommends install locales
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && \
  locale-gen

RUN ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
  dpkg-reconfigure --frontend noninteractive tzdata
# end: basic configuration

# java
ARG JAVA_BASE=/usr/lib/jvm
ENV JAVA_HOME ${JAVA_BASE}/graalvm
ENV PATH "$PATH:$JAVA_HOME/bin"

RUN cd /tmp && \
  wget -q https://github.com/graalvm/graalvm-ce-builds/releases/download/jdk-${V_GRAAL}/graalvm-community-jdk-${V_GRAAL}_linux-x64_bin.tar.gz -O dist.tar.gz && \
  tar -xvzf dist.tar.gz && \
  rm -f dist.tar.gz && \
  mkdir -p ${JAVA_BASE} && \
  chmod -R 755 ${JAVA_BASE} && \
  mv graalvm-*/ ${JAVA_HOME} && \
  update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 99999 && \
  java --version
# end: java

# dotnet
ENV PATH "$PATH:/.dotnet/tools"

ENV DOTNET_CLI_TELEMETRY_OPTOUT=1
RUN printf 'Package: *\nPin: origin "packages.microsoft.com"\nPin-Priority: 1001' > /etc/apt/preferences.d/99microsoft-dotnet.pref

RUN curl -o packages-microsoft-prod.deb https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb && \
  dpkg -i packages-microsoft-prod.deb && \
  rm packages-microsoft-prod.deb

RUN apt-get update && apt-get install -y apt-transport-https dotnet-sdk-${V_DOTNETSDK} aspnetcore-runtime-${V_DOTNETSDK} && apt-get upgrade -y

RUN dotnet --version && \
  dotnet tool install Amazon.Lambda.Tools --tool-path /.dotnet/tools && \
  dotnet new -i Amazon.Lambda.Templates
# end: dotnet

ENV PATH "$PATH:/app"
RUN apt-get clean -y
