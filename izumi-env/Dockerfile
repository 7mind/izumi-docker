ARG V_UBUNTU=18.04
FROM ubuntu:${V_UBUNTU}

ARG V_SCALA=2.12.8
ARG V_TYPESCRIPT=3.2.2
ARG V_GOLANG=1.10
ARG V_DOTNETSDK=2.2
ARG V_OPENJDK=11
ARG URELEASE=bionic

RUN mkdir -p /work
RUN mkdir -p /root
VOLUME /work
VOLUME /root
WORKDIR /work

ENV DEBIAN_FRONTEND noninteractive

RUN echo "APT::Acquire::Queue-Mode "access"; APT::Acquire::Retries 3; " > /etc/apt/apt.conf.d/99parallel && \
  export DEBIAN_FRONTEND=noninteractive && \
  apt-get update && apt-get install -y --no-install-recommends tzdata gnupg ca-certificates wget curl unzip apt-utils build-essential git openssh-client && \
  ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  apt-get clean -y

RUN curl -o packages-microsoft-prod.deb https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb && \
  dpkg -i packages-microsoft-prod.deb && \
  rm packages-microsoft-prod.deb

RUN curl -sS https://download.mono-project.com/repo/xamarin.gpg | apt-key add - && \
  echo "deb https://download.mono-project.com/repo/ubuntu stable-${URELEASE} main" | tee /etc/apt/sources.list.d/mono-official-stable.list && \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823 && \
  echo "deb https://dl.bintray.com/sbt/debian /" | tee /etc/apt/sources.list.d/sbt.list && \
  curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
  echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu ${URELEASE} stable" | tee /etc/apt/sources.list.d/docker.list && \
  apt-get update

RUN apt-get install -y --no-install-recommends openjdk-${V_OPENJDK}-jdk \
  mono-devel nuget nunit-console dotnet-sdk-${V_DOTNETSDK} \
  npm yarn \
  python-pip python-setuptools \
  golang-${V_GOLANG} \
  docker-ce

RUN wget -q www.scala-lang.org/files/archive/scala-${V_SCALA}.deb && dpkg -i scala-${V_SCALA}.deb && rm -rf *.deb && \
  npm i -g typescript@${V_TYPESCRIPT} && \
  wget -q -O /usr/local/bin/coursier https://git.io/vgvpD && chmod +x /usr/local/bin/coursier && \
  pip install awscli

RUN apt-get install -y --no-install-recommends sbt && \
  apt-get install --no-install-recommends -y ruby ruby-dev && \
  apt-get clean -y

RUN gem update --system && \
  gem install sass && \
  gem install jekyll -v 3.2.1

#  /usr/lib/go-1.10/bin needs to be added into PATH manually: https://github.com/golang/go/wiki/Ubuntu
ENV GOPATH $HOME/gopath
ENV PATH $HOME/gopath/bin:/usr/lib/go-${V_GOLANG}/bin:$PATH

RUN echo "java: $(java -version)" && \
  echo "scalac: $(scalac -version)" && \
  echo "csc: $(csc -version)" && \
  echo "tsc: $(tsc -version)" && \
  echo "go: $(go version)" && \
  echo "dotnet: $(dotnet --version)" && \
  echo "npm: $(npm -v)" && \
  echo "yarn: $(yarn -v)" && \
  echo "jekyll: $(jekyll -v)" && \
  sbt about
