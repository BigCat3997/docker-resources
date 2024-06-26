FROM ubuntu:22.04
ENV TARGETARCH="linux-x64"
ARG ACTIVE_USER="agent"

# Install software system
## By package manager
RUN apt update -y && apt upgrade -y
RUN apt install -y \
        apt-transport-https \
        apt-utils \
        gnupg \
        ca-certificates \
        libicu70 \
        lsb-release \
        software-properties-common \
        iputils-ping \
        telnet \
        wget \
        curl \
        git \
        jq \
        libxml2-utils \
        tree \
        nano \
        vim \
        zip

# Repo credential
## Temurin JDK
RUN wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | apt-key add -
RUN echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
## Docker
RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
RUN chmod a+r /etc/apt/keyrings/docker.asc
RUN echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        tee /etc/apt/sources.list.d/docker.list > /dev/null
## MS software
RUN apt remove 'dotnet*' 'aspnet*' 'netstandard*' -y && \
        touch /etc/apt/preferences && \
        printf 'Package: dotnet* aspnet* netstandard* powershell\nPin: origin "packages.microsoft.com"\nPin-Priority: -10\n' >> /etc/apt/preferences
## Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash -

RUN apt-get update
RUN apt-get install -y \
        docker-ce \
        docker-ce-cli \
        containerd.io \
        docker-buildx-plugin \
        docker-compose-plugin \
        maven \
        temurin-17-jdk \
        temurin-11-jdk \
        nodejs \
        dotnet-sdk-6.0

# Install manually software
# Download the PowerShell package file
RUN wget https://github.com/PowerShell/PowerShell/releases/download/v7.4.2/powershell_7.4.2-1.deb_amd64.deb && \
        dpkg -i powershell_7.4.2-1.deb_amd64.deb && \
        rm powershell_7.4.2-1.deb_amd64.deb

WORKDIR /azp/

COPY ./start.sh ./
RUN chmod +x ./start.sh

RUN mkdir -p /opt /data
RUN useradd -m -d /home/$ACTIVE_USER $ACTIVE_USER
RUN usermod -a -G docker $ACTIVE_USER
RUN chown -R $ACTIVE_USER:$ACTIVE_USER /azp /home/$ACTIVE_USER /opt /data

USER $ACTIVE_USER

# Install software user
## Miniconda
RUN mkdir -p /opt/miniconda3 && \
        wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /opt/miniconda3/miniconda.sh
RUN bash /opt/miniconda3/miniconda.sh -b -u -p /opt/miniconda3
RUN rm -rf /opt/miniconda3/miniconda.sh
RUN /opt/miniconda3/bin/conda init && /opt/miniconda3/bin/conda config --set auto_activate_base false
## Azure agent
RUN curl -LsS https://vstsagentpackage.azureedge.net/agent/3.238.0/vsts-agent-linux-x64-3.238.0.tar.gz | tar -xz & wait $!

# Expose environment variables
## Change the location contains libraries of platforms
ENV CONDA_ENVS_DIRS=/data/.conda/envs
ENV NUGET_PACKAGES=/data/.nuget
ENV JAVA_JDK17_HOME=/usr/lib/jvm/temurin-17-jdk-amd64
ENV JAVA_JDK11_HOME=/usr/lib/jvm/temurin-11-jdk-amd64
ENV MAVEN_OPTS="-Dmaven.repo.local=/data/.m2"
RUN npm config set prefix /data/.npm

ENTRYPOINT ./start.sh
