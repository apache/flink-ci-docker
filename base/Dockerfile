# check=skip=FromPlatformFlagConstDisallowed
FROM --platform=linux/amd64 ubuntu:26.04

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

ENV JAVA_HOME_8_X64="/usr/lib/jvm/java-8-openjdk-amd64"
ENV JAVA_HOME_11_X64="/usr/lib/jvm/jdk-11.0.19+7"
ENV JAVA_HOME_17_X64="/usr/lib/jvm/jdk-17.0.7+7"
ENV JAVA_HOME_21_X64="/usr/lib/jvm/jdk-21.0.1+12"
ENV JAVA_HOME_25_X64="/usr/lib/jvm/jdk-25.0.3+9"
ENV JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
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
ARG BASE_URL=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/${MAVEN_VERSION}

RUN mkdir -p /usr/share/maven \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME=/usr/share/maven

# Pre-warm the Maven distribution cache for both the jar-based and the only-script
# Maven Wrapper (they use different cache layouts), so CI's ./mvnw needs no network to bootstrap.
# Entries are <version>:<sha256 of apache-maven-<version>-bin.zip>; distributionUrl must match Flink's byte for byte.
# The jar-based wrapper only checks that the zip exists once unpacked, so it is replaced with an empty placeholder.
# Each wrapper is then re-run through a dead proxy to fail the build if it would still go to the network.
# MAVEN_USER_HOME is honored by both wrappers and lives outside /root, so the cache is found regardless of $HOME or uid;
# it is world-writable so versions not pre-warmed here can still be downloaded by any user.
ENV MAVEN_USER_HOME=/opt/maven-user-home
ARG MAVEN_WARM="3.8.6:ccf20a80e75a17ffc34d47c5c95c98c39d426ca17d670f09cd91e877072a9309 3.9.16:5af3b743dd8b876b5c45da33b676251e5f1687712644abb4ee519ca56e1d89ce"
ARG MAVEN_WRAPPER_VERSION=3.3.4
RUN set -eux; \
    work="$(mktemp -d)"; cd "$work"; \
    base="https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper-distribution/${MAVEN_WRAPPER_VERSION}/maven-wrapper-distribution-${MAVEN_WRAPPER_VERSION}"; \
    for type in bin only-script; do \
      mkdir -p "$type/.mvn/wrapper"; \
      curl -fsSL -o "$type.zip" "${base}-${type}.zip"; \
      unzip -q "$type.zip" -d "$type"; chmod +x "$type/mvnw"; \
    done; \
    for entry in $MAVEN_WARM; do \
      v="${entry%%:*}"; sha="${entry#*:}"; \
      for type in bin only-script; do \
        printf 'distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/%s/apache-maven-%s-bin.zip\ndistributionSha256Sum=%s\n' "$v" "$v" "$sha" > "$type/.mvn/wrapper/maven-wrapper.properties"; \
        (cd "$type" && ./mvnw --version); \
      done; \
      for zip in "$MAVEN_USER_HOME"/wrapper/dists/apache-maven-"$v"-bin/*/apache-maven-"$v"-bin.zip; do : > "$zip"; done; \
      for type in bin only-script; do \
        (cd "$type" && https_proxy=http://127.0.0.1:9 MAVEN_OPTS="-Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=9" ./mvnw --version); \
      done; \
    done; \
    chmod -R a+rwX "$MAVEN_USER_HOME"; \
    cd /; rm -rf "$work"

# Stage the maven-wrapper jars used by Flink branches; Flink's CI copies the matching one into .mvn/wrapper
# instead of downloading it, and ./mvnw still validates it. Entries are <version>:<sha256 of maven-wrapper-<version>.jar>.
ARG MAVEN_WRAPPER_JARS="3.2.0:e63a53cfb9c4d291ebe3c2b0edacb7622bbc480326beaa5a0456e412f52f066a 3.3.2:3d8f20ce6103913be8b52aef6d994e0c54705fb527324ceb9b835b338739c7a8 3.3.4:4e2fbf6554bc8a4702cdfdd3bef464f423393d784ddbb037216320ce55d5e4e1"
RUN set -eux; \
    mkdir -p /opt/maven-wrapper; \
    for entry in $MAVEN_WRAPPER_JARS; do \
      v="${entry%%:*}"; sha="${entry#*:}"; jar="/opt/maven-wrapper/maven-wrapper-$v.jar"; \
      curl -fsSL -o "$jar" "https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper/$v/maven-wrapper-$v.jar"; \
      echo "$sha  $jar" | sha256sum -c -; \
    done; \
    chmod -R a+rX /opt/maven-wrapper

# Use UTF-8
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen ; locale-gen ; update-locale en_US.UTF-8
ENV LC_ALL="en_US.UTF-8"
ENV LANG="en_US.UTF-8"
ENV LANGUAGE="en_US.UTF-8"

# OpenSSL 1.0.0, required by netty-tcnative on older Flink branches (e.g. 1.20)
RUN wget -r --no-parent -nd --accept=libssl1.0.0_*ubuntu5.*_amd64.deb http://security.ubuntu.com/ubuntu/pool/main/o/openssl1.0/
RUN apt install ./libssl1.0.0_*.deb
