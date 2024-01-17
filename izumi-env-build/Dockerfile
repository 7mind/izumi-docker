ARG V_UBUNTU=22.04
FROM ubuntu:${V_UBUNTU}
ARG URELEASE=jammy

###
ARG V_GRAAL=21.0.2
ARG V_SCALA=2.13.12
ARG V_SBT=1.9.8

ARG V_TYPESCRIPT=4.0.2
ARG V_GOLANG=1.21
ARG V_DOTNETSDK=7.0
ARG V_MVN=3.6.3-5
ARG V_PROTOBUF=25.1
ARG V_JEKYLL=3.2.1
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

# sources
RUN curl -o packages-microsoft-prod.deb https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb && \
  dpkg -i packages-microsoft-prod.deb && \
  rm packages-microsoft-prod.deb

RUN add-apt-repository ppa:longsleep/golang-backports

RUN curl -sS https://download.mono-project.com/repo/xamarin.gpg | apt-key add - && \
  echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" | tee /etc/apt/sources.list.d/mono-official-stable.list

RUN  curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

RUN  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
  echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu ${URELEASE} stable" | tee /etc/apt/sources.list.d/docker.list

RUN curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add - && \
  echo "deb [arch=amd64] https://apt.releases.hashicorp.com ${URELEASE} main"| tee /etc/apt/sources.list.d/terraform.list

RUN  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823 && \
  curl -sL https://deb.nodesource.com/setup_20.x | bash -

RUN curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | apt-key add && \
  echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | tee /etc/apt/sources.list.d/sbt.list && \
  echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | tee /etc/apt/sources.list.d/sbt_old.list

RUN apt-get update && apt-get upgrade -y
# end: sources


# we don't need it but it fixes broken apt
#RUN apt-get install -y openjdk-21-jre-headless

RUN apt-get install -y --no-install-recommends \
  mono-devel nuget nunit-console dotnet-sdk-${V_DOTNETSDK} msbuild

RUN apt-get install -y --no-install-recommends \
  nodejs yarn

RUN apt-get install -y --no-install-recommends \
  python-pip python-setuptools

RUN apt-get install -y --no-install-recommends \
  golang-${V_GOLANG}

RUN apt-get install -y --no-install-recommends \
  docker-ce \
  terraform \
  zsh nano upx

RUN apt-get install -y --no-install-recommends \
  sbt=${V_SBT} zlib1g-dev

RUN apt-get install -y --no-install-recommends \
  maven=${V_MVN}


RUN apt-get install -y --no-install-recommends \
  ruby ruby-dev

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
  update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 99999
# end: java

RUN wget -q www.scala-lang.org/files/archive/scala-${V_SCALA}.deb && dpkg -i --ignore-depends=java8-runtime-headless scala-${V_SCALA}.deb  && rm -rf *.deb && \
  curl -Lo /usr/local/bin/coursier https://git.io/coursier-cli && chmod +x /usr/local/bin/coursier

RUN curl -Lo /usr/local/bin/cs https://git.io/coursier-cli-linux && chmod +x /usr/local/bin/cs

RUN npm i -g typescript@${V_TYPESCRIPT}

RUN apt-get install -y --no-install-recommends python3-pip && \
  pip3 install awscli

RUN gem install sass && \
  gem install jekyll -v ${V_JEKYLL}

#  /usr/lib/go-1.10/bin needs to be added into PATH manually: https://github.com/golang/go/wiki/Ubuntu
ENV GOPATH $HOME/gopath
ENV PATH $HOME/gopath/bin:/usr/lib/go-${V_GOLANG}/bin:$PATH

RUN echo "java: $(java -version)" && \
  echo "java: $(native-image --version)" && \
  echo "scalac: $(scalac -version)" && \
  echo "csc: $(csc -version)" && \
  echo "tsc: $(tsc -version)" && \
  echo "go: $(go version)" && \
  echo "dotnet: $(dotnet --version)" && \
  echo "npm: $(npm -v)" && \
  echo "node: $(node -v)" && \
  echo "yarn: $(yarn -v)" && \
  echo "jekyll: $(jekyll -v)" && \
  sbt about

ENV PB_REL="https://github.com/protocolbuffers/protobuf/releases"
RUN curl -LO $PB_REL/download/v${V_PROTOBUF}/protoc-${V_PROTOBUF}-linux-x86_64.zip && unzip protoc-${V_PROTOBUF}-linux-x86_64.zip -d /usr/proto
RUN mv /usr/proto/bin/protoc /usr/bin/protoc && mv /usr/proto/include /usr/bin/include && chmod +x /usr/bin/protoc

RUN apt-get clean -y
