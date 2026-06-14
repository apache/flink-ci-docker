FROM ubuntu:22.04

# install packages
RUN set -eux; \
	apt-get update; apt-get upgrade -y ; \
	apt-get install -y \
		gosu \
		sudo \
		locales \
		make \
		less \
		build-essential \
		`# default JDK for Apache Flink` \
		openjdk-8-jdk \
		`# FLINK-18356` \
		libjemalloc-dev \
		curl \
		libapr1 \
		unzip \
		uuid-runtime \
		jq \
		bsdmainutils \
		wget \
		`# for pyflink test suite` \
		python3 \
		docker.io \
		psmisc \
		software-properties-common \
		apt-transport-https; \
	rm -rf /var/lib/apt/lists/*;\
	# enables legacy python command for PyFlink tests
	ln -s /usr/bin/python3 /usr/bin/python

# Install latest git
RUN set -eux; \
  add-apt-repository -y ppa:git-core/ppa; \
  apt-get update; \
  apt-get install -y git

# Install Temurin JDK11
RUN wget -q https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.19%2B7/OpenJDK11U-jdk_x64_linux_hotspot_11.0.19_7.tar.gz
RUN tar -C /usr/lib/jvm -xzf OpenJDK11U-jdk_x64_linux_hotspot_11.0.19_7.tar.gz

# Install Temurin JDK17
RUN wget -q https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.7%2B7/OpenJDK17U-jdk_x64_linux_hotspot_17.0.7_7.tar.gz
RUN tar -C /usr/lib/jvm -xzf OpenJDK17U-jdk_x64_linux_hotspot_17.0.7_7.tar.gz

# Install Temurin JDK21
RUN wget -q https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.1%2B12/OpenJDK21U-jdk_x64_linux_hotspot_21.0.1_12.tar.gz
RUN tar -C /usr/lib/jvm -xzf OpenJDK21U-jdk_x64_linux_hotspot_21.0.1_12.tar.gz

# Install Temurin JDK25
RUN wget -q https://github.com/adoptium/temurin25-binaries/releases/download/jdk-25.0.3%2B9/OpenJDK25U-jdk_x64_linux_hotspot_25.0.3_9.tar.gz
RUN tar -C /usr/lib/jvm -xzf OpenJDK25U-jdk_x64_linux_hotspot_25.0.3_9.tar.gz

# Add java to PATH for all users, and user
# use java8 by default, but provide java11 as well
RUN echo "PATH=/usr/lib/jvm/java-8-openjdk-amd64/bin:$PATH" >> /etc/environment
RUN echo "JAVA_HOME_8_X64=/usr/lib/jvm/java-8-openjdk-amd64" >> /etc/environment
RUN echo "JAVA_HOME_11_X64=/usr/lib/jvm/jdk-11.0.19+7" >> /etc/environment
RUN echo "JAVA_HOME_17_X64=/usr/lib/jvm/jdk-17.0.7+7" >> /etc/environment
RUN echo "JAVA_HOME_21_X64=/usr/lib/jvm/jdk-21.0.1+12" >> /etc/environment
RUN echo "JAVA_HOME_25_X64=/usr/lib/jvm/jdk-25.0.3+9" >> /etc/environment
RUN echo "JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> /etc/environment
RUN update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
RUN update-alternatives --set javac /usr/lib/jvm/java-8-openjdk-amd64/bin/javac
RUN update-alternatives --set jps /usr/lib/jvm/java-8-openjdk-amd64/bin/jps
RUN update-alternatives --set jstack /usr/lib/jvm/java-8-openjdk-amd64/bin/jstack
RUN update-alternatives --set javadoc /usr/lib/jvm/java-8-openjdk-amd64/bin/javadoc

ENV JAVA_HOME_8_X64 "/usr/lib/jvm/java-8-openjdk-amd64"
ENV JAVA_HOME_11_X64 "/usr/lib/jvm/jdk-11.0.19+7"
ENV JAVA_HOME_17_X64 "/usr/lib/jvm/jdk-17.0.7+7"
ENV JAVA_HOME_21_X64 "/usr/lib/jvm/jdk-21.0.1+12"
ENV JAVA_HOME_25_X64 "/usr/lib/jvm/jdk-25.0.3+9"
ENV JAVA_HOME "/usr/lib/jvm/java-8-openjdk-amd64"
ENV LD_PRELOAD="/usr/lib/x86_64-linux-gnu/libjemalloc.so"

# Install go
RUN mkdir -p /usr/share/go \
 && curl -fsSL -o /tmp/go.tar.gz https://go.dev/dl/go1.18.1.linux-amd64.tar.gz \
 && tar -xzf /tmp/go.tar.gz -C /usr/share/go --strip-components=1 \
 && rm -f /tmp/go.tar.gz \
 && ln -s /usr/share/go/bin/go /usr/bin/go

# Install hugo
RUN mkdir -p /usr/share/hugo \ 
 && curl -fsSL -o /tmp/hugo.tar.gz https://github.com/gohugoio/hugo/releases/download/v0.98.0/hugo_extended_0.98.0_Linux-64bit.tar.gz \
 && tar -xzf /tmp/hugo.tar.gz -C /usr/share/hugo \
 && rm -f /tmp/hugo.tar.gz \
 && ln -s /usr/share/hugo/hugo /usr/bin/hugo

# Install maven
ARG MAVEN_VERSION=3.8.6
ARG USER_HOME_DIR="/root"
ARG BASE_URL=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/${MAVEN_VERSION}

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven

# Use UTF-8
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen ; locale-gen ; update-locale en_US.UTF-8
ENV LC_ALL "en_US.UTF-8"
ENV LANG "en_US.UTF-8"
ENV LANGUAGE "en_US.UTF-8"

# Install OpenSSL version that's required by netty-tcnative
RUN wget -r --no-parent -nd --accept=libssl1.0.0_*ubuntu5.*_amd64.deb http://security.ubuntu.com/ubuntu/pool/main/o/openssl1.0/
RUN apt install ./libssl1.0.0_*.deb
