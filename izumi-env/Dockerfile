ARG V_UBUNTU=18.04
FROM ubuntu:${V_UBUNTU}

ARG V_SCALA=2.12.8
ARG V_TYPESCRIPT=3.2.2
ARG V_GOLANG=1.10
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
  apt-get update && apt-get install -y --no-install-recommends tzdata gnupg ca-certificates wget curl unzip apt-utils && \
  ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  apt-get clean -y

RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF && \
  echo "deb https://download.mono-project.com/repo/ubuntu stable-${URELEASE} main" | tee /etc/apt/sources.list.d/mono-official-stable.list && \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823 && \
  echo "deb https://dl.bintray.com/sbt/debian /" | tee /etc/apt/sources.list.d/sbt.list && \
  curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu ${URELEASE} stable" | tee /etc/apt/sources.list.d/docker.list

RUN apt-get update && \
  apt-get install -y --no-install-recommends openjdk-${V_OPENJDK}-jdk git mono-devel nuget nunit-console npm yarn docker-ce golang-${V_GOLANG}

RUN wget -q www.scala-lang.org/files/archive/scala-${V_SCALA}.deb && dpkg -i scala-${V_SCALA}.deb && rm -rf *.deb && \
  npm i -g typescript@${V_TYPESCRIPT} && \
  wget -q -O /usr/local/bin/coursier https://git.io/vgvpD && chmod +x /usr/local/bin/coursier

RUN apt-get update && \
  apt-get install -y --no-install-recommends sbt && \
  apt-get install -y --no-install-recommends ruby ruby-dev && \
  apt-get clean -y

RUN rvm use 2.2.8 --install --fuzzy && \
  gem update --system && \
  gem install sass && \
  gem install jekyll -v 3.2.1

#  /usr/lib/go-1.10/bin needs to be added into PATH manually: https://github.com/golang/go/wiki/Ubuntu
ENV GOPATH $HOME/gopath
ENV PATH $HOME/gopath/bin:/usr/lib/go-${V_GOLANG}/bin:$PATH

RUN java -version && \
  scalac -version && \
  csc -version && \
  tsc -version && \
  go version && \
  sbt about && \
  npm -v && \
  yarn -v
