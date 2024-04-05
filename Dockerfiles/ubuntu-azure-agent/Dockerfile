ARG PLATFORM

FROM ubuntu:20.04 AS base
RUN DEBIAN_FRONTEND=noninteractive apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    git \
    iputils-ping \
    jq \
    lsb-release \
    software-properties-common \
    wget \
    tree \
    telnet \
    nano \
    nodejs \
    npm \
    maven \
    powershell

RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
RUN chmod a+r /etc/apt/keyrings/docker.asc
RUN echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
            tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update
RUN apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
RUN \
    echo "ulimits: $(ulimit -Sn):$(ulimit -Hn)"; \
    sed -i 's/ulimit -Hn/# ulimit -Hn/g' /etc/init.d/docker; \
    rm -rf /var/cache/apt;

RUN mkdir -p /opt/miniconda3
RUN mkdir -p /opt/dotnet6
RUN mkdir -p /opt/java17
RUN mkdir -p /opt/java11
RUN mkdir -p /opt/flyway

FROM base AS amd-installer
USER root
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /opt/miniconda3/miniconda.sh
RUN wget -v https://download.visualstudio.microsoft.com/download/pr/530c0041-ad39-4918-b658-9e8d9b0e3982/41efca744e6ae51fbffd51a8f546bb9c/dotnet-sdk-6.0.128-linux-x64.tar.gz -O /opt/dotnet6/dotnet-sdk-6.0.128-linux-x64.tar.gz
RUN wget -v https://download.java.net/java/GA/jdk17.0.2/dfd4a8d0985749f896bed50d7138ee7f/8/GPL/openjdk-17.0.2_linux-x64_bin.tar.gz -O /opt/java17/openjdk-17.0.2_linux-x64_bin.tar.gz
RUN wget -v https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz -O /opt/java11/openjdk-11.0.2_linux-x64_bin.tar.gz
RUN wget -v https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/10.9.1/flyway-commandline-10.9.1-linux-x64.tar.gz -O /opt/flyway/flyway-commandline-10.9.1-linux-x64.tar.gz

# Install software
## Dotnet6
RUN tar -xvf /opt/dotnet6/dotnet-sdk-6.0.128-linux-x64.tar.gz -C /opt/dotnet6
RUN rm -rf /opt/dotnet6/dotnet-sdk-6.0.128-linux-x64.tar.gz
## Java17
RUN tar -xvf /opt/java17/openjdk-17.0.2_linux-x64_bin.tar.gz -C /opt/java17
RUN cp -r /opt/java17/jdk-17.0.2/. /opt/java17/.
RUN rm -rf /opt/java17/openjdk-17.0.2_linux-x64_bin.tar.gz /opt/java17/jdk-17.0.2
## Java11
RUN tar -xvf /opt/java11/openjdk-11.0.2_linux-x64_bin.tar.gz -C /opt/java11
RUN cp -r /opt/java11/jdk-11.0.2/. /opt/java11/.
RUN rm -rf /opt/java11/openjdk-11.0.2_linux-x64_bin.tar.gz /opt/java11/jdk-11.0.2
## Flyway
RUN tar -xvf /opt/flyway/flyway-commandline-10.9.1-linux-x64.tar.gz -C /opt/flyway
RUN cp -r /opt/flyway/flyway-10.9.1/. /opt/flyway/.
RUN rm -rf /opt/flyway/flyway-commandline-10.9.1-linux-x64.tar.gz /opt/flyway/flyway-10.9.1

ENV TARGETARCH=linux-x64

FROM base AS arm-installer
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh -O /opt/miniconda3/miniconda.sh
RUN wget -v https://download.visualstudio.microsoft.com/download/pr/0041b4a7-e890-4fb5-8bf3-8d31cefed5ac/505c3738dbcd0a94edfa0af32afdc95e/dotnet-sdk-6.0.128-linux-arm64.tar.gz -O /opt/dotnet6/dotnet-sdk-6.0.128-linux-arm64.tar.gz
RUN wget -v https://download.java.net/java/GA/jdk17.0.2/dfd4a8d0985749f896bed50d7138ee7f/8/GPL/openjdk-17.0.2_linux-aarch64_bin.tar.gz -O /opt/java17/openjdk-17.0.2_linux-aarch64_bin.tar.gz

# Install software
## Dotnet6
RUN tar -xvf /opt/dotnet6/dotnet-sdk-6.0.128-linux-arm64.tar.gz -C /opt/dotnet6
RUN rm -rf /opt/dotnet6/dotnet-sdk-6.0.128-linux-arm64.tar.gz
## Java17
RUN tar -xvf /opt/java17/openjdk-17.0.2_linux-aarch64_bin.tar.gz -C /opt/java17
RUN cp -r /opt/java17/jdk-17.0.2/. /opt/java17/.
RUN rm -rf /opt/java17/openjdk-17.0.2_linux-aarch64_bin.tar.gz /opt/java17/jdk-17.0.2

ENV TARGETARCH=linux-arm64

FROM ${PLATFORM}-installer AS final
# Install software
# Miniconda3
RUN bash /opt/miniconda3/miniconda.sh -b -u -p /opt/miniconda3
RUN rm -rf /opt/miniconda3/miniconda.sh
RUN /opt/miniconda3/bin/conda init
RUN /opt/miniconda3/bin/conda config --set auto_activate_base false
# Override the PATH
ENV PATH="${PATH}:/opt/dotnet6"
ENV JAVA_HOME="/opt/java17"
ENV PATH="${PATH}:${JAVA_HOME}/bin"
ENV CONDA_ENVS_DIRS=/data/.conda/envs
ENV NUGET_PACKAGES=/data/.nuget
ENV MAVEN_OPTS="-Dmaven.repo.local=/data/.m2"
ENV PATH="${PATH}:/opt/flyway"
RUN npm config set prefix /data/.npm

WORKDIR /azp

COPY ./start.sh .
RUN chmod +x start.sh

ENTRYPOINT service docker start && ./start.sh
