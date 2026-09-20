FROM --platform=linux/amd64 ubuntu:24.04

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
		openjdk-8-jdk-headless \
		`# FLINK-18356` \
		libjemalloc-dev \
		curl \
		`# libapr1 was renamed to libapr1t64 in the time_t transition` \
		libapr1t64 \
		unzip \
		uuid-runtime \
		jq \
		bsdmainutils \
    	moreutils \
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
  apt-get install -y git; \
  rm -rf /var/lib/apt/lists/*

# Install Temurin JDK11
RUN wget -qO- https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.19%2B7/OpenJDK11U-jdk_x64_linux_hotspot_11.0.19_7.tar.gz \
  | tar -C /usr/lib/jvm -xz

# Install Temurin JDK17
RUN wget -qO- https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.7%2B7/OpenJDK17U-jdk_x64_linux_hotspot_17.0.7_7.tar.gz \
  | tar -C /usr/lib/jvm -xz

# Install Temurin JDK21
RUN wget -qO- https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.1%2B12/OpenJDK21U-jdk_x64_linux_hotspot_21.0.1_12.tar.gz \
  | tar -C /usr/lib/jvm -xz

# Install Temurin JDK25
RUN wget -qO- https://github.com/adoptium/temurin25-binaries/releases/download/jdk-25.0.3%2B9/OpenJDK25U-jdk_x64_linux_hotspot_25.0.3_9.tar.gz \
  | tar -C /usr/lib/jvm -xz

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
 && wget -qO- https://go.dev/dl/go1.27.1.linux-amd64.tar.gz \
  | tar -xz -C /usr/share/go --strip-components=1 \
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

RUN mkdir -p /usr/share/maven \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven

# Download the Maven Wrapper once, then use it to stage its jar and pre-warm the
# Maven distribution cache, so CI's ./mvnw needs no network to bootstrap.
ARG MAVEN_WARM_VERSIONS="3.8.6 3.9.16"
ARG MAVEN_WRAPPER_VERSION=3.3.4
RUN set -eux; \
    work="$(mktemp -d)"; cd "$work"; \
    curl -fsSL -o d.zip "https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper-distribution/${MAVEN_WRAPPER_VERSION}/maven-wrapper-distribution-${MAVEN_WRAPPER_VERSION}-bin.zip"; \
    unzip -q d.zip; chmod +x mvnw; \
    mkdir -p /opt/maven-wrapper; \
    cp .mvn/wrapper/maven-wrapper.jar "/opt/maven-wrapper/maven-wrapper-${MAVEN_WRAPPER_VERSION}.jar"; \
    for v in $MAVEN_WARM_VERSIONS; do \
      printf 'distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/%s/apache-maven-%s-bin.zip\n' "$v" "$v" > .mvn/wrapper/maven-wrapper.properties; \
      ./mvnw --version; \
    done; \
    cd /; rm -rf "$work"

# Use UTF-8
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen ; locale-gen ; update-locale en_US.UTF-8
ENV LC_ALL "en_US.UTF-8"
ENV LANG "en_US.UTF-8"
ENV LANGUAGE "en_US.UTF-8"

# OpenSSL 1.0.0, required by netty-tcnative on older Flink branches (e.g. 1.20)
RUN wget -r --no-parent -nd --accept=libssl1.0.0_*ubuntu5.*_amd64.deb http://security.ubuntu.com/ubuntu/pool/main/o/openssl1.0/
RUN apt install ./libssl1.0.0_*.deb

# OpenSSL 3.4.x (OPENSSL_3.2.0 symbols) required by flink-shaded-netty-tcnative-dynamic 2.0.81+,
# which Ubuntu 24.04 (OpenSSL 3.0.13) lacks. Put it in /usr/local/lib so ldconfig prefers it.
RUN set -eux; \
	tmp="$(mktemp -d)"; cd "$tmp"; \
	wget -q http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl3t64_3.4.1-1ubuntu3_amd64.deb; \
	dpkg-deb -x libssl3t64_3.4.1-1ubuntu3_amd64.deb extracted; \
	cp -a extracted/usr/lib/x86_64-linux-gnu/libssl.so.3 extracted/usr/lib/x86_64-linux-gnu/libcrypto.so.3 /usr/local/lib/; \
	ldconfig; \
	cd /; rm -rf "$tmp"
